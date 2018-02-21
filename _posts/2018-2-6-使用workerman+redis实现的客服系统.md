## redis
主要的几个状态：
1. 会话中
2. 等待中
3. 排队中

### db0 客户消息

###  会话
客户和客服建立连接，相互之间发送消息，回复消息，就叫会话。

会话建立的步骤：
1. 用户通过微信给客服系统发送消息，微信将消息通过post推送给客服公众号后台系统（公众号的回调地址）
2. 开发者对微信服务器推送的事件进行响应处理，即根据用户发送消息类型（文本，图像，声音，位置，链接等）进行相应的处理
2. 客服系统收到消息，首先根据用户openid查找是否之前发过客服消息即是否存在客服会话（MySQL的message_session表）
3. 如果之前没有发送过客服消息，建立一个会话并保存至MySQL的tb_message_session表中
4. 会话建立成功之后，将该**客服消息**保存至message_data表中（Common::addMessagge方法），并添加至redis的db1中
5. 根据关键词向message_rule表中查找会话规则，自动回复客户
4. 然后该会话进入（排队中）状态，等待客服人员接入；已被客服人员接入的会话，会进入（等待中）状态，会话列表中的第一个会话进入会话中状态；
5. 如果之前存在会话，使用之前建立的这个会话；同时，一个会话若超过一段时间（eg.10天）没有回复，则系统会回收这个回话的客资。即下次该客户再次询问客服时会进入排队系统，等待系统给他分配新的uid（客服人员）；
6. 会话建立后，客服人员通过调用we_chat/WxOperation控制器的sendMessage方法给客户发送消息。

具体实现：
1. 创建客服会话 会话类型other（不指定客服） user（指定的客服） group（指定的客服分组）,先判断公众号是否绑定了第三方平台
2. 匹配是否存在正在会话中数据： 会话状态 -4管理员关闭会话 -3超时无人接待 -2接待超时关闭 -1会话关闭 （0等待接入会话 1会话中 3排队中）：正在会话中状态
3. 判断是否存在专属客服： wx_user 专属客服 `customer_service_uid` bigint(12) NOT NULL DEFAULT '-1' COMMENT '专属客服uid'；如果存在专属客服则查找此客服信息
4. 

 
### redis中几个主要的表
#### db0  等待中（已分配了客服） db1（客户发送的消息） db2会话排队中（未分配客服 state=3,存储在message_session表）
 1. 客户通过微信发送信息
 2. 触发微信事件 logic/we_chat/ line 307
````
{
  "text": "\u4f60\u597d\uff0c\u6628\u5929\u6211\u4eec\u7684\u7f51\u9c7c\u670d\u52a1\u8425\u9500\u7cfb\u7edf\u4e86\u89e3\u7684\u5982\u4f55\uff0c\u6709\u65f6\u95f4\u5728\u6765\u770b\u770b\u52d2\uff0c\u6211\u7ed9\u4f60\u8be6\u7ec6\u5256\u6790\u4e0b\u6211\u4eec\u5ba2\u8d44\u7ba1\u7406\u6a21\u5757",
  "message_id": "4e59468e2c63ba9c8e52f8dbb7eacef8",
  "appid": "wx4a14a2375e93cb7b", 
  "company_id": "51454009d703c86c91353f61011ecf2f",
  "customer_service_id": 66,
  "customer_wx_openid": "oyeC-wH2XKLJo94TOoBpnCeEAMeQ",
  "add_time": "2018-01-18 10:55:50",
  "opercode": 3,
  "message_type": 1, //`message_type` int(1) NOT NULL DEFAULT '0' COMMENT '消息类型 0其他 1文本 2图片 3语音 4视频 5坐标 6链接',
  "session_id": "48fbc56d108a03d39585f274066bffde", //回话id，每个用户仅保存一个回话
  "uid": 8082  //客服uid
}

{
  "media_id": "OekkzkyvXu14Yq107AqIXY63HU0nC1_nOtfKQ7O6oSzXl7Jy43ofIDNwTh43xUcF",
  "resources_id": "",
  "file_url": "",
  "message_id": "7f95af68c436edf317c717e1b70958e5",
  "appid": "wx4a14a2375e93cb7b",
  "company_id": "51454009d703c86c91353f61011ecf2f",
  "customer_service_id": 8082,
  "customer_wx_openid": "oyeC-wH2XKLJo94TOoBpnCeEAMeQ",
  "add_time": "2018-01-18 10:56:08",
  "opercode": 2,
  "message_type": 3,  // `message_type` int(1) NOT NULL DEFAULT '0' COMMENT '消息类型 0其他 1文本 2图片 3语音 4视频 5坐标 6链接',
  "session_id": "48fbc56d108a03d39585f274066bffde",
  "uid": 8082
}
```
db2 会话消息

db4 缓存 token 有效期，权限验证

````

### 网鱼客户端打开调试界面‘ ctrl+alt+d+b ’

### GatewayWorker 主要使用方法介绍
#### 业务开发只需要关注 Applications/YourApp/Events.php这个文件即可。所以，下述主要的方法也都在这个文件里使用
1. 当客户端连接时触发
```
/**
        * 当客户端连接时触发
        * 如果业务不需此回调可以删除onConnect
        * @param int $client_id 连接id
        */
       public static function onConnect($client_id){
            // 向当前client_id发送数据
                    Gateway::sendToClient($client_id, "Hello $client_id");
                    // 向所有人发送
                    Gateway::sendToAll("$client_id login");
       }
```
2. 当客户端发来消息时触发
````
 /**
    * 当客户端发来消息时触发
    * @param int $client_id 连接id
    * @param string $message 具体消息
    */
   public static function onMessage($client_id, $message)
   {
        // 向所有人发送
        Gateway::sendToAll("$client_id said $message");
   }
````
3. 当客户端断开连接时触发
````
 /**
    * 当用户断开连接时触发
    * @param int $client_id 连接id
    */
   public static function onClose($client_id)
   {
       // 向所有人发送
       GateWay::sendToAll("$client_id logout");
   }
````

#### Events.php中定义5个事件回调方法，

1. onWorkerStart businessWorker进程启动事件（一般用不到）
2. onConnect 连接事件(比较少用到)
3. onMessage 消息事件(必用) **重要**
4. onClose 连接断开事件(比较常用到) **重要**
5. onWorkerStop businessWorker进程退出事件（几乎用不到）

#### start_gateway.php
start_gateway.php为gateway进程启动脚本，主要定义了客户端连接的端口号、协议等信息，具体参见 Gateway类的使用一节。

客户端连接的就是start_gateway.php中初始化的Gateway端口。

#### start_businessworker.php
start_businessworker.php为businessWorker进程启动脚本，也即是调用Events.php的业务处理进程，具体参见 BusinessWorker类的使用一节。

#### start_register.php
start_register.php为注册服务启动脚本，用于协调GatewayWorker集群内部Gateway与Worker的通信，参见Register类使用一节。

注意：客户端不要连接Register服务端口，客户端应该连接Gateway端口



### redis
1. zRange(key, start, end,withscores)：
返回名称为key的zset（元素已按score从小到大排序）中的index从start到end的所有元素

2. sMembers, sGetMembers
返回名称为key的set的所有元素

3. zAdd
zAdd(key, score, member)：向名称为key的zset中添加元素member，score用于排序。如果该元素已经存在，则根据score更新该元素的顺序。

3. sAdd
向名称为key的set中添加元素value,如果value存在，不写入，return false










