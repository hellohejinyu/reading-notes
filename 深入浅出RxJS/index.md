# 深入浅出 RxJS

![cover](https://rescdn.qqmail.com/weread/cover/890/933890/t6_933890.jpg)

    作者：程墨
    ISBN：9787111596646

***

## 第一章 函数响应式编程

函数式编程对于函数的使用有一些特殊的要求，这些要求包括以下几点：
  - 声明式
  - 纯函数
    > 一个函数之所以不纯，可能做了下面这些事情：
    > - 改变全局变量的值
    > - 改变输入参数引用的对象
    > - 读取用户输入，比如调用了 alert 或者 confirm 函数
    > - 抛出一个异常
    > - 网络输入/输出操作，比如通过 Ajax 调用一个服务器的 API
    > - 操作浏览器的 DOM
  - 数据不可变性

## 第二章 RxJS 入门

### Hot Observable 和 Cold Observable

一个 Observable 是 Hot 还是 Cold，是“热”还是“冷”，都是相对于生产者而言的，如果每次订阅的时候，已经有一个热的“生产者”准备好了，那就是 Hot Observable，相反，如果每次订阅都要产生一个新的生产者，新的生产者就像汽车引擎一样刚启动时肯定是冷的，所以叫 Cold Observable。
> 假设有这样的场景，一个 Observable 对象有两个 Observer 对象来订阅，而且这两个 Observer 对象并不是同时订阅，第一个 Observer 对象订阅 N 秒钟之后，第二个 Observer 对象才订阅同一个 Observable 对象，而且，在这 N 秒钟之内，Observable 对象已经吐出了一些数据。如果后一个 Observer 将会错过订阅前 Observable 发出的消息，那么这就是 Hot Observable

## 第四章 创建数据流

### defer

defer 接受一个函数作为参数，当 defer 产生的 Observable 对象被订阅的时候，defer 的函数参数就会被调用，预期这个函数会返回另一个 Observable 对象，也就是 defer 转嫁所有工作的对象。

``` js
import { defer } from "rxjs";
import { ajax } from "rxjs/ajax";

const observableFactory = () =>
  ajax("https://api.github.com/repos/ReactiveX/rxjs");

const deferSource$ = defer(observableFactory);

setTimeout(() => {
  deferSource$.subscribe(res => {
    console.log(res);
  });
}, 3e3);
```

上面的代码执行完之后产生了 deferSource$ ，但是 AJAX 请求并没有发送出去，只有当 deferSource$ 订阅的时候才发送 AJAX 请求，，这归功于 defer 产生了 deferSource$ 这样一个代理 Observable 对象。