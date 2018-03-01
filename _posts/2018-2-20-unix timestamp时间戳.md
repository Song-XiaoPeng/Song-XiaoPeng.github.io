Unix时间戳(Unix timestamp)，或称Unix时间(Unix time)、POSIX时间(POSIX time)，是一种时间表示方式，定义为从格林威治时间1970年01月01日00时00分00秒起至现在的总秒数。Unix时间戳不仅被使用在Unix 系统、类Unix系统中，也在许多其他操作系统中被广告采用。

　　目前相当一部分操作系统使用32位二进制数字表示时间。此类系统的Unix时间戳最多可以使用到格林威治时间2038年01月19日03时14分07秒（二进制：01111111 11111111 11111111 11111111）。其后一秒，二进制数字会变为10000000 00000000 00000000 00000000， 发生溢出错误，造成系统将时间误解为1901年12月13日20时45分52秒。这很可能会引起软件故障，甚至是系统瘫痪。使用64位二进制数字表示时间 的系统（最多可以使用到格林威治时间292,277,026,596年12月04日15时30分08秒）则基本不会遇到这类溢出问题。

首先你要理解什么是时间戳。时间戳是指格林威治时间1970年01月01日00时00分00秒(北京时间1970年01月01日08时00分00秒)起至某一时间点的总秒数。

例如北京时间2015-12-31 17:00:00的时间戳是1451552400，就是指从北京时间1970-01-01 08:00:00到2015-12-31 17:00:00已经过去了1451552400秒。

网上有在线的时间戳转换工具：http://tool.chinaz.com/Tools/unixtime.aspx

下面一一问答你的问题：

1、MySQL数据库的timestamp为什么从1970到2038的某一时间？
MySQL的timestamp类型是4个字节，最大值是2的31次方减1，也就是2147483647，转换成北京时间就是2038-01-19 11:14:07。可以使用上面的在线的时间戳转换工具得出。

2、某一时间是指什么时间？
如上，北京时间是2038-01-19 11:14:07。

3、过了这个时间之后怎么办？
1) 使用datetime；
2) 等过了再说。

最后补充一个彩蛋：北京时间2038-01-19 11:14:07，如果某些系统还在使用MySQL的Timestamp，或者系统使用的编程语言用4字节补码整型（例如Java的int）来表示时间戳的，这些系统都会跪掉，说不定会搞出点什么大新闻（例如飞机航班系统挂掉、银行某系统跪掉），好吧，我乱猜的，等着那一天吧。

### Mysql TIMESTAMP 

TIMESTAMP DEFAULT CURRENT_TIMESTAMP 表示插入的时候自动获取当前时间（格式为Y-m-d H:i:s）
[sql] view plain copy
ALTER TABLE [table_name] MODIFY created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP NOT NULL;  
TIMESTAMP ON UPDATE CURRENT_TIMESTAMP 表示更新的时候自动获取当前时间（格式为Y-m-d H:i:s）
[sql] view plain copy
ALTER TABLE [table_name] MODIFY updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP NOT NULL;  
ps:TIMESTAMP 与 DATETIME的区别
TIMESTAMP:值不能早于1970或晚于2037。。DATETIME:'1000-01-01 00:00:00'到'9999-12-31 23:59:59'