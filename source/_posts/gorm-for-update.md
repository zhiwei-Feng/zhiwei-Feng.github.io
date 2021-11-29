---
title: Gorm加悲观锁的最新用法
date: 2021-11-30 00:06:39
tags:
- gorm
- go
categories:
- Golang
---

因为google了“gorm for update”或者是“gorm 开启排他锁”出来的文章清一色的使用着如下用法来开启表的行/表锁  
```go
tx.Set("gorm:query_option", "FOR UPDATE").First(&employee, id)
```

但是经过测试，我加了没有作用，搜索gorm官方文档，结果用法已经变成如下
```go
db.Clauses(clause.Locking{Strength: "UPDATE"}).Find(&users)
// SELECT * FROM `users` FOR UPDATE

db.Clauses(clause.Locking{
  Strength: "SHARE",
  Table: clause.Table{Name: clause.CurrentTable},
}).Find(&users)
// SELECT * FROM `users` FOR SHARE OF `users`
```

果然有问题先找官方文档。😂
