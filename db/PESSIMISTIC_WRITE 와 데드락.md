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

이슈를 파헤치다 보니 몇가지 의문이 발생했다.  
- `innodb_lock_wait_timeout` 에 설정된 50초 보다 터무니 없이 짧은 시간인 1초 내외로 deadlock 예외가 발생한다는 점
- PessimisticLockException 이 아닌 OptimisticLockException 예외가 발생한다는 점

내부 로직이 복잡하기 때문에 


## 🧗 해결

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
