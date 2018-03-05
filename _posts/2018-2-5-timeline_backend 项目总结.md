timeline_backend 项目总结
## 1
client?d62f:147 [WDS] Warnings while compiling.
warnings @ client?d62f:147
onmessage @ socket.js:41
EventTarget.dispatchEvent @ sockjs.js:170
(anonymous) @ sockjs.js:883
SockJS._transportMessage @ sockjs.js:881
EventEmitter.emit @ sockjs.js:86
WebSocketTransport.ws.onmessage @ sockjs.js:2957
client?d62f:153 ./~/_vue-loader@11.3.4@vue-loader/lib/template-compiler?{"id":"data-v-8e114958"}!./~/_vue-loader@11.3.4@vue-loader/lib/selector.js?type=template&index=0!./~/_iview-loader@1.0.0@iview-loader?{"prefix":false}!./src/views/Layout/index.vue
(Emitted value instead of an instance of Error) <BreadcrumbItem v-for="item in breadcrumb">: component lists rendered with v-for should have explicit keys. See https://vuejs.org/guide/list.html#key for more info.
 @ ./src/views/Layout/index.vue 9:2-296
 @ ./src/router.js
 @ ./src/main.js
 @ multi ./~/_webpack-dev-server@2.11.2@webpack-dev-server/client?http://localhost:8081 webpack/hot/dev-server ./src/main

## 2

vue.esm.js:578 [Vue warn]: Invalid prop: type check failed for prop "pageSize". Expected Number, got String.

found in

---> <Page>
       <Index> at D:\vue\timeline_backend\src\views\timeline\index.vue
         <Card>
           <Content>
             <Layout>... (1 recursive calls)
               <Index> at D:\vue\timeline_backend\src\views\Layout\index.vue
                 <App> at D:\vue\timeline_backend\src\app.vue
                   <Root>

## 3
vue-router.esm.js:16 [vue-router] Duplicate named routes definition: { name: "detail", path: "/index/detail" }

## 4
刷新页面，menu菜单的activity跳到了第一个，并且breadcrumb不显示了

## 5
路由/index/timeline/detail 显示的是/index/timeline页面而不是它自己的页面

## /index?id=1

当使用this.$router.query 时，打印结果为undefined
当使用this.$route.query时，打印结果为{"id":1}

### NOTICE 
this.$router和this.$route的区别

1.
```
const router = new VueRouter({
  routes // （缩写）相当于 routes: routes
}) 	//创建 router 实例，然后传 `routes` 配置
```

该文档通篇都常使用 router 实例。留意一下 this.$router 和 router 使用起来完全一样。我们使用 this.$router 的原因是我们并不想在每个独立需要封装路由的组件中都导入路由。

2.

一个 route object（路由信息对象） 表示当前激活的路由的状态信息，包含了当前 URL 解析得到的信息，还有 URL 匹配到的 route records（路由记录）。

route object 是 immutable（不可变） 的，每次成功的导航后都会产生一个新的对象。

route object 出现在多个地方:

在组件内，即 this.$route

在 $route 观察者回调内

router.match(location) 的返回值

