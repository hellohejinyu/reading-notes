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

## 第五章 合并数据流

### 合并类操作符

#### concat

首尾相连，当第一个 Observable 对象 complete 之后，concat 就会去 subscribe 第二个 Observable 对象获取数据，把数据同样传给下游。
假设 concat 有两个输入，分别称为 source1$ 和 source2$。source1$ 产生的所有数据全都被 concat 直接转给了下游，当 source1$ 完结的时候，concat 会调用 source1$.unsubscribe，然后调用 source2$.subscribe，继续从 source2$ 中抽取数据传给下游。

#### merge

merge 做的事情很简单：依次订阅上游 Observable 对象，把接收到的数据转给下游，等待所有上游对象 Observable 完结。假设有两个同步数据的 Observable 对象，当 merge 订阅 source1$ 之后，还没来得及去订阅 source2$, source1$ 就一口气把自己的数据全吐出来了，实际上产生了 concat 的效果。  
所以，应该避免用merge去合并同步数据流，merge应该用于合并产生异步数据的Observable对象，一个常用场景就是合并DOM事件。

#### zip

```js
import { of, zip } from "rxjs";

const source1$ = of(1, 2, 3);
const source2$ = of("a", "b", "c");
const zipped$ = zip(source1$, source2$);

zipped$.subscribe(v => {
  console.log(v);
  // [1, "a"]
  // [2, "b"]
  // [3, "c"]
});
```

当 zip 执行的时候，它会立刻订阅所有的上游 Observable，然后开始合并数据，在上面的例子中，source1$ 产生的数据序列会和 source2$ 产生的数据序列配对，1 配上 a,2 配上 b,3 配上 c，所以产生 3 个数组传递给下游。  
如果某个上游 source1$ 吐出数据的速度很快，而另一个上游 source2$ 吐出数据的速度很慢，那 zip 就不得不先存储 source1$ 吐出的数据，因为 RxJS 的工作方式是“推”, Observable 把数据推给下游之后自己就没有责任保存数据了。被 source1$ 推送了数据之后，zip 就有责任保存这些数据，等着和 source2$ 未来吐出的数据配对。假如 source2$ 迟迟不吐出数据，那么 zip 就会一直保存 source1$ 没有配对的数据，然而这时候 source1$ 可能会持续地产生数据，最后 zip 积压的数据就会越来越多，占用的内存也就越来越多。

#### combineLatest

combineLatest 合并数据流的方式是当任何一个上游 Observable 产生数据时，从所有输入 Observable 对象中拿最后一次产生的数据（最新数据），然后把这些数据组合起来传给下游。

combineLatest 在某些情况下会有一些问题，例如下面的两个 Observable 对象 s1$ 和 s2$，数据来源均为 origin$，如果使用 combineLatest，第一条数据为正常的 `["0a", "0b"]`，但是接着会输出两行 `["1a", "0b"], ["1a", "1b"]`，这是因为多个上游 Observable “同时”吐出一个数据，当然，并不是真正的“同时”，几个事件之间可能会间隔几纳秒的时间，但是因为它们是由同一个数据源（在上面的例子中就是 origin$）引发的，所以逻辑上算是“同时”。
```js
const origin$ = interval(1000);

const s1$ = origin$.pipe(map(v => v + "a"));
const s2$ = origin$.pipe(map(v => v + "b"));
```
解决这个问题就是使用 withLatestFrom 来替代这种情况。

#### withLatestFrom

- 如果要合并完全独立的 Observable 对象，使用 combineLatest。
- 如果要把一个 Observable 对象“映射”成新的数据流，同时要从其他 Observable 对象获取“最新数据”，就是用 withLatestFrom。

#### race

胜者通吃，剩余的 Observable 对象会被退订。

#### startWith

指定 Observable 被订阅时立即返回的数据。

#### forkJoin

forkJoin 就是 RxJS 界的 Promise.all, Promise.all 等待所有输入的 Promise 对象成功之后把结果合并，forkJoin 等待所有输入的 Observable 对象完结之后把最后一个数据合并。

