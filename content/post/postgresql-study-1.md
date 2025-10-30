+++
date = '2025-10-30T13:32:10+08:00'
draft = false
title = 'PostgreSQL 学习记录'
tags = ['数据库', 'postgresql']
categories = ['分享']

+++

**创建schema**

下面举例创建一个名为 mr 的schema。

**切换指定的schema**

```sql
set search_path = mr;
```



```sql
create schema if not exists mr;
```

**创建table**

```sql
CREATE TABLE mr.users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    create_time TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    enabled BOOLEAN DEFAULT TRUE
);
```

上面的 sql 是在 mr schema 下面创建了一个名为 users 的表，下面是字段类型解释

- id：SERIAL 类型类似 mysql 中的自增，它是 psql 中的自增
- create_time：`timestamp with time zone default now()` 表示存储了一个带时区的时间，这样在全球不同的地方都会显示正确的时区时间。

上面俩个是主要与 mysql 不同的字段类型，其余相同类似的就不做介绍。

> serial 类型拓展：
>
> smallserial：2字节，范围大概到32,767
>
> serial：（默认）4字节，范围 1～2,147,483,647
>
> bigserial：8字节
>
> 它的机制是创建表是如果使用了serial类型，psql 会自动为我们创建一个 **sequence（序列对象）**，然后将该列的默认值设置为从这个序列中 `nextval()` 的结果。它会自动隐含 NOT NULL 约束。

**如何重置 serial 呢**

1. 确认 sequence 名称

   ```sql
   select pg_get_serial_sequence('users', 'id');
   ```

2. 重置 sequence 为1

   ```sql
   alter sequence mr.users_id_seq restart with 1;
   ```

通过上面的操作就可以将 serial 的字段重置为1，下次插入时就会从1开始。
