## 
- 字段类型
    - timestamp
        - 格式： `YYYY-MM-DD HH:MM:SS[.fraction]    `
        - 存储范围： `1970-01-01 00:00:01.000000` - `2038-01-19 03:14:07.999999`
        - 存储方式：把客户端插入的时间从当前时区转化为UTC（世界标准时间）进行存储。查询时，将其又转化为客户端当前时区进行返回。
        - 变体：

            ```
                1，TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
                在创建新记录和修改现有记录的时候都对这个数据列刷新

                2，TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                在创建新记录的时候把这个字段设置为当前时间，但以后修改时，不再刷新它

                3，TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
                在创建新记录的时候把这个字段设置为0，以后修改时刷新它

                4，TIMESTAMP DEFAULT ‘yyyy-mm-dd hh:mm:ss’ ON UPDATE CURRENT_TIMESTAMP 
                在创建新记录的时候把这个字段设置为给定值，以后修改时刷新它
            ```
    - datetime
        - 格式: `YYYY-MM-DD HH:MM:SS[.fraction]`
        - 存储范围: `1000-01-01 00:00:00.000000` - `9999-12-31 23:59:59.999999`
        - 存储方式：不做任何改变，基本上是原样输入和输出。


## demo
```
# 创建时设置未当前时间，更新时不变
alter table users modify created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP;

# 创建和修改时都对该列进行更新
alter table users modify created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP; 
```

- laravel框架Eloquent会默认自动维护这两个字段
 created_at 
 updated_at 

 1. 可在模型内将 $timestamps 属性设置为 false，就不自动更新了
 2. 如果你需要自定义自己的时间戳格式，可在模型内设置 $dateFormat 属性,这个属性决定了日期应如何在数据库中存储，以及当模型被序列化成数组或 JSON 格式.
 3. 自定义时间戳字段名
 
 ```
    const CREATED_AT = 'created';
    const UPDATED_AT = 'updated';
```

 > 参考blog[Mysql中timestamp用法详解](https://www.cnblogs.com/givemelove/p/8251185.html)