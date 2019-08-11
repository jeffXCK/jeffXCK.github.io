---
title: >-
  Mysql编码引起的 Illegal mix of collations (utf8_unicode_ci,IMPLICIT) and
  (utf8_general_ci,IMPLICIT)错误
date: 2019-07-18 12:28:47
categories: MySQL
tags: MySQL
---

##### 1.【错误经过：】

* 在 mysql 数据库执行多表连接查询时：

```mysql
select * from A LEFT JOIN B ON A.user_id = b.user_id
```

* 出现错误：

```
 Illegal mix of collations (utf8_unicode_ci,IMPLICIT) and (utf8_general_ci,IMPLICIT)
```

​	意思大概就是说 A 表的编码格式和 B 表的编码方式不一致，不能进行比较。

##### 2.【解决办法：】

  * 将 A表 和 B表 的 （ `collations` 或者 `校对规则`）的编码的方式统一为 utf8_general_ci

  * 然后执行如下语句：

    ```mysql
    alter table 表名 convert to character set utf8 collate utf8_general_ci
    ```
    
    


*注意 ！！！*

*一般做了上面这一步还是不能解决问题，这是因为表里面的数据是之前插入进去的，编码方式自然也是之前的 `utf8_unicode_ci` 了，所以做数据比较的时候依然报错！！！要解决问题：还需要继续执行第二步，把表数据的编码规则也统一纠正过来*





*(另，注：utf8_general_ci和utf8_unicode_ci的区别，前者校对速度快，但准确度稍差；后者准确度高，但校对速度稍慢。）*