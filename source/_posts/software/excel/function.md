---
    title:
    categories:
    tags:
    creator: cjq
    create_time: 2021/01/27
---

#### timestamp to date
1. second: ```=(A1+8*3600)/86400+70*365+19```
2. millsecond: ```=(A1+8*3600)/8640000+70*365+19```

#### date to timestamp
1. second: ```=(B1-70*365-19)*86400-8*3600```
