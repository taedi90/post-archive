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
> 

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



```sql
MariaDB [(none)]> show variables like '%isolation%';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| tx_isolation  | REPEATABLE-READ |
+---------------+-----------------+
```

비유니크 인덱스에 대한 쿼리 였기 때문에 gap lock 이 발생했다는 것을 깨달았고

## 🚀 참고
- [https://mangkyu.tistory.com/299](https://mangkyu.tistory.com/299)
- [https://medium.com/daangn/mysql-gap-lock-%EB%8B%A4%EC%8B%9C%EB%B3%B4%EA%B8%B0-7f47ea3f68bc](https://medium.com/daangn/mysql-gap-lock-%EB%8B%A4%EC%8B%9C%EB%B3%B4%EA%B8%B0-7f47ea3f68bc)
