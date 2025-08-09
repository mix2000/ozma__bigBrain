---
order: 1
title: Отключение проверок целостности
---

```sql
SET FOREIGN_KEY_CHECKS = 0;
TRUNCATE TABLE maru_admin;
SET FOREIGN_KEY_CHECKS = 1;
```


