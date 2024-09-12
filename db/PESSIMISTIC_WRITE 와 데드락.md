---
title: PESSIMISTIC_WRITE 와 데드락
date: 2024-09-04
draft: true
tags:
banner:
cssclasses:
description:
permalink:
aliases:
completed:
---
## 📝 요약
> [!summary]
> **REPEATABLE READ** 격리 수준에서는 비유니크 인덱스를 조건으로 **PESSIMISTIC_WRITE**를 사용하면 InnoDB에서 **레코드 락(Record Lock)**뿐만 아니라 **갭 락(Gap Lock)**도 수행될 수 있으며, 이러한 갭 락은 의도치 않은 데드락을 유발할 수 있다.
> 데드락을 피하기 위해서는 상황에 따라 아래 방법 등을 고민해볼 수 있다.
> - 비유니크 인덱스 조건을 **`WHERE PK IN (A, B)`**와 같이 기본 키(PK)를 이용한 조건으로 변경
> - 격리 수준을 **READ COMMITTED** 이하로 변경
> - 비관적 락이 아닌 낙관적 락으로 로직 변경 
> - 트랜젝션 오류 시 재시도 로직 추가

## ⚙️ 환경
- mariadb 10.8.3

## 💬 이슈
사내 솔루션의 비관적락(PESSIMISTIC_WRITE) 을 사용하는 로직에 트랜젝션 경합이 발생할 경우 간헐적으로 deadlock 이 발생하는 이슈가 확인되었다.  

로그를 분석하다 보니 의문점이 생겼다.  
- 일반적인 상황에서는 lock 획득을 하기 위해서 대기를 하는 것이 정상이지만, 문제 상황에서는 lock 획득 시도 후 1초 이내에 오류가 발생했으며, 이는 `innodb_lock_wait_timeout` 에 설정된 50초 보다 터무니 없이 짧은 수준
- 발생 오류 또한 optimisticLockException 으로 lock 획득 불가시 발생하는 pessimisticLockException 과 stackTrace 가 다름

내부 로직이 복잡하기 때문에 


## 🧗 해결
### 원인 파악
우선 `SHOW ENGINE INNODB STATUS` 를 통해 deadlock 이 발생한 직후 `LATEST DETECTED DEADLOCK` 데이터를 확인하려 했지만 연쇄적으로 다른 deadlock 이 발생하는 문제로 로그를 확보하기가 어려웠다.  
```config
[mysqld]
# 모든 데드락 로그를 저장
innodb_print_all_deadlocks = 1
```

그래서 mysql 설정에 다음 항목을 추가하고 재기동해주고나서 mysql_error.log 로그에서 deadlock 을 확인할 수 있었다.

```
2024-09-10 12:38:06 2195 [Note] InnoDB: Transactions deadlock detected, dumping detailed information.

2024-09-10 12:38:06 2195 [Note] InnoDB:

*** (1) TRANSACTION:

  

TRANSACTION 7994580, ACTIVE 0 sec fetching rows

mysql tables in use 1, locked 1

LOCK WAIT 5 lock struct(s), heap size 1128, 3 row lock(s), undo log entries 2

MariaDB thread id 2195, OS thread handle 139620720838400, query id 43213 172.18.0.8 root Sending data

select ... from dv_datamart.tb_rm_agent_multistatus agentmulti0_ where agentmulti0_.agent_id=134 for update

2024-09-10 12:38:06 2195 [Note] InnoDB: *** WAITING FOR THIS LOCK TO BE GRANTED:

  

RECORD LOCKS space id 288 page no 3 n bits 112 index PRIMARY of table `dv_datamart`.`tb_rm_agent_multistatus` trx id 7994580 lock_mode X locks rec but not gap waiting

Record lock, heap no 105 PHYSICAL RECORD: n_fields 16; compact format; info bits 0
...

2024-09-10 12:38:06 2195 [Note] InnoDB: *** CONFLICTING WITH:

  

RECORD LOCKS space id 288 page no 3 n bits 112 index PRIMARY of table `dv_datamart`.`tb_rm_agent_multistatus` trx id 7994579 lock_mode X locks rec but not gap

Record lock, heap no 83 PHYSICAL RECORD: n_fields 16; compact format; info bits 0
...

  

Record lock, heap no 105 PHYSICAL RECORD: n_fields 16; compact format; info bits 0
...

2024-09-10 12:38:06 2195 [Note] InnoDB:

*** (2) TRANSACTION:

  

TRANSACTION 7994579, ACTIVE 0 sec fetching rows

mysql tables in use 1, locked 1

LOCK WAIT 5 lock struct(s), heap size 1128, 4 row lock(s), undo log entries 2

MariaDB thread id 2194, OS thread handle 139620691040000, query id 43212 172.18.0.8 root Sending data

select agentmulti0_.multi_id as multi_id1_119_, agentmulti0_.acwclose_time as acwclose2_119_, agentmulti0_.agent_id as agent_id3_119_, agentmulti0_.agent_multistatus as agent_mu4_119_, agentmulti0_.assign_time as assign_t5_119_, agentmulti0_.channel_id as channel_6_119_, agentmulti0_.chatclose_time as chatclos7_119_, agentmulti0_.chatstart_time as chatstar8_119_, agentmulti0_.db_update_time as db_updat9_119_, agentmulti0_.nexthop as nexthop10_119_, agentmulti0_.passive_yn as passive11_119_, agentmulti0_.routing_type as routing12_119_, agentmulti0_.status_time as status_13_119_, agentmulti0_.ucid as ucid14_119_ from dv_datamart.tb_rm_agent_multistatus agentmulti0_ where agentmulti0_.agent_id=74 for update

2024-09-10 12:38:06 2195 [Note] InnoDB: *** WAITING FOR THIS LOCK TO BE GRANTED:

  

RECORD LOCKS space id 288 page no 3 n bits 112 index PRIMARY of table `dv_datamart`.`tb_rm_agent_multistatus` trx id 7994579 lock_mode X locks rec but not gap waiting

Record lock, heap no 106 PHYSICAL RECORD: n_fields 16; compact format; info bits 0
...

  

2024-09-10 12:38:06 2195 [Note] InnoDB: *** CONFLICTING WITH:

  

RECORD LOCKS space id 288 page no 3 n bits 112 index PRIMARY of table `dv_datamart`.`tb_rm_agent_multistatus` trx id 7994580 lock_mode X locks rec but not gap

Record lock, heap no 106 PHYSICAL RECORD: n_fields 16; compact format; info bits 0
...

  

2024-09-10 12:38:06 2195 [Note] InnoDB: *** WE ROLL BACK TRANSACTION (0)

```

lock_mode X locks rec but not gap waiting

```sql
MariaDB [(none)]> show variables like '%isolation%';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| tx_isolation  | REPEATABLE-READ |
+---------------+-----------------+
```

비유니크 인덱스에 대한 쿼리 였기 때문에 gap lock 이 발생했다는 것을 깨달았고

격리수준이나 낙관적락으로 로직을 변경해도 될까?


데드락 발생 시나리오
```sql
[transaction 1]> START TRANSACTION;
[transaction 1]> SELECT * from dv_datamart.tb_rm_agent_multistatus  
         WHERE agent_id = 74 FOR UPDATE ;
[transaction 2]> START TRANSACTION;
[transaction 2]> SELECT * from dv_datamart.tb_rm_agent_multistatus  
         WHERE agent_id = 134 FOR UPDATE ;
[transaction 1]> SELECT * from dv_datamart.tb_rm_agent_multistatus  
         WHERE agent_id = 74 FOR UPDATE ; /* 데드락 발생 */
```
## 🚀 참고
- https://mariadb.com/kb/en/innodb-lock-modes/
- https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html
- [https://mangkyu.tistory.com/299](https://mangkyu.tistory.com/299)
- [https://medium.com/daangn/mysql-gap-lock-%EB%8B%A4%EC%8B%9C%EB%B3%B4%EA%B8%B0-7f47ea3f68bc](https://medium.com/daangn/mysql-gap-lock-%EB%8B%A4%EC%8B%9C%EB%B3%B4%EA%B8%B0-7f47ea3f68bc)
- [https://jaeseongdev.github.io/development/2021/06/16/Lock%EC%9D%98-%EC%A2%85%EB%A5%98-(Shared-Lock,-Exclusive-Lock,-Record-Lock,-Gap-Lock,-Next-key-Lock)/](https://jaeseongdev.github.io/development/2021/06/16/Lock%EC%9D%98-%EC%A2%85%EB%A5%98-(Shared-Lock,-Exclusive-Lock,-Record-Lock,-Gap-Lock,-Next-key-Lock)/)
- [https://medium.com/daangn/mysql-gap-lock-%EB%91%90%EB%B2%88%EC%A7%B8-%EC%9D%B4%EC%95%BC%EA%B8%B0-49727c005084](https://medium.com/daangn/mysql-gap-lock-%EB%91%90%EB%B2%88%EC%A7%B8-%EC%9D%B4%EC%95%BC%EA%B8%B0-49727c005084)