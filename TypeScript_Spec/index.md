## Introduction

### 1.3 Object Types

TypeScript 对象类型可以对 JavaScript 对象所能展示的行为多样性建模。例如 jQuery 库定义了一个对象 `$`，其中有 `get` 等方法。但是，jQuery 也可以直接使用例如 `$('#id')` 来将 `$` 作为函数调用。

``` ts
interface JQuery {
  text(content: string);
}

interface JQueryStatic {
  get(url: string, callback: (data: string) => any);
  (query: string): JQuery;
}

declare var $: JQueryStatic;

$.get('http://mysite.org/divContent', (data: string) => {
  $('div').text(data);
})
```

### 1.6 Classes

``` ts
class BankAccount {
  constructor(public balance: number) {}
  deposit(credit: number) {
    this.balance += credit
    return this.balance
  }
}
```

这里 BankAccount 的构造函数中有一个 balance 参数，它前面的 `public` 关键字表示此构造函数参数将作为字段保留。
> 指定 `private` 或 `protected` 有同样的效果。

