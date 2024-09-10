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
사내 솔루션 내부에 비관적락(PESSIMISTIC_WRITE) 을 사용하는 로직에 간헐적으로 deadlock 이 발생하는 경우가 생겼다.



## 🧗 해결

비유니크 인덱스에 대한 쿼리 였기 때문에 gap lock 이 발생했다는 것을 깨달았고

## 🚀 참고
- [https://mangkyu.tistory.com/299](https://mangkyu.tistory.com/299)
- [https://medium.com/daangn/mysql-gap-lock-%EB%8B%A4%EC%8B%9C%EB%B3%B4%EA%B8%B0-7f47ea3f68bc](https://medium.com/daangn/mysql-gap-lock-%EB%8B%A4%EC%8B%9C%EB%B3%B4%EA%B8%B0-7f47ea3f68bc)
