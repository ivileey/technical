# MongoDB

>MongoDB是为快速开发互联网Web应用而设计的数据库系统。
>
>其数据模型和持久化策略就是为了构建高读/写吞吐量和高自动灾备伸缩性的系统。
>
>一种非关系型数据库，在文档里存储数据

## 使用

```
> use tutorial
> db.users.insert({username:"smith"}) //insert
> db.users.find()
> db.users.count()
> db.users.update({username:"smith"},{$set:{country:"Canda"}})
```

