---
title: "resolve 関数と reject 関数の使い方"
cssclass: zenn
date: 2022-05-06
modified: 2024-08-14
AutoNoteMover: disable
tags: type/zenn/book, JavaScript/async
aliases: Promise本『resolve 関数と reject 関数の使い方』
---

## このチャプターについて

『[Promise の基本概念](a-epasync-promise-basic-concept)』のチャプターでも少し触れましたが、`resolve()` メソッドは `reject()` メソッドよりも複雑なので使い方に気をつける必要があります。

このチャプターでは、`new Promise(executor)` で使用する Executor 関数の引数である `resolve()` 関数と `reject()` 関数の注意点について解説していきます。

## resolve や reject ではコールバック関数の実行は止まらない

次のコードの様に `Promise()` コンストラクタに渡すコールバックである executor 関数内において、`resolve()` や `reject()` を呼び出しただけでは関数の実行は終わらないことに注意してください。

```js
// unstoppablePromiseConstructor.js
const promise = new Promise((resolve, reject) => {
  if (Math.random() < 0.5) {
    resolve("😁 true なので履行する");
  } else {
    reject("😭 false なので拒否する");
  }
  console.log("👻 関数は止まらない");
});

promise
  .then((value) => console.log("🍓 履行値:", value))
  .catch((reason) => console.error("🥦 拒否理由:", reason))
  .finally(() => console.log("👍 チェーン最後に実行"));
```

それぞれ 50% の確率で履行、拒否されます。出力結果は以下のようになり、`resolve()` と `reject()` のどちらを呼び出したとしてもコールバック関数の実行が終わっていないことが分かります。

```sh
❯ deno run unstoppablePromiseConstructor.js
👻 関数は止まらない
🍓 履行値: 😁 true なので履行する
👍 チェーン最後に実行
❯ deno run unstoppablePromiseConstructor.js
👻 関数は止まらない
🥦 拒否理由: 😭 false なので拒否する
👍 チェーン最後に実行
```

`resolve()` や `reject()` 関数でコールバックの実行が止まらないということは次のようなことができてしまいます。

```js
// whichState.js
const promise = new Promise((resolve, reject) => {
  resolve("😁 履行する [1]");
  console.log("👻 関数は止まらない");
  reject("😭 拒否する [1]");
  console.log("👻 関数は止まらない");
  resolve("😁 履行する [2] ");
  console.log("👻 関数は止まらない");
  reject("😭 拒否する [2]");
});

promise
  .then((value) => console.log("🍓 履行値:", value))
  .catch((reason) => console.error("🥦 拒否理由:", reason))
  .finally(() => console.log("👍 チェーン最後に実行"));
```

さて、このコードの出力はどうなるでしょうか？

:::details 答え
出力結果は以下のものとなります。

```sh
❯ deno run whichState.js
👻 関数は止まらない
👻 関数は止まらない
👻 関数は止まらない
🍓 履行値: 😁 履行する [1]
👍 チェーン最後に実行
```
:::

`resolve()` や `reject()` でコールバック関数内の処理は止まりません。ですが、『[Promise の基本概念](a-epasync-promise-basic-concept)』のチャプターで言ったとおり、Promise インスタンスが一度 Settled (Fulfilled または Rejected 状態) になったらもう二度とそのインスタンスの状態は変わりません。従って、一度状態が変化したら、いくら `resolve()` や `reject()` を呼び出しても何も効果を得ることはできません。解決値が変わったり、例外が発生することもありません。

`resolve()` や `reject()` は Promise インスタンスの状態を変えるのを試みるだけです。

ということで、`resolve()` と `reject()` 関数の先に呼び出されたもののみがその Promise インスタンスの状態へ影響を与えます。ということで、上記コードで最初に呼び出された `resolve("😁 履行する [1]");` のみが効果があります。それゆえ、`then()` メソッドのコールバックで出力される履行値が、`😁 履行する [1]` となっています。

`resolve()` や `reject()` で確実に止めたいなら、`return` するか `new Error()` で例外を発生させます。

`reject()` 関数では引数に `new Error()` を取ることで例外を発生させると確実に止めることができます。

```js
// unstoppablePromiseConstructor.js
const promise = new Promise((resolve, reject) => {
  if (Math.random() < 0.5) {
    return resolve("😁 true なので履行する");
    console.log("👍 関数は止まるよ");
  } else {
    reject(new Error("😭 拒否する"));
    console.log("👍 関数は止まるよ");
  }
});

promise
  .then((value) => console.log("🍓 履行値:", value))
  .catch((reason) => console.log("🥦 拒否理由:", reason))
  .finally(() => console.log("👍 チェーン最後に実行"));
```

また、Promise chain において、`then()` メソッドに登録したコールバック関数も従っている Promise インスタンスの状態が変化した場合にたった一度だけ呼び出されます。それゆえ、一番最初に呼び出される `resolve()` や `reject()` 以外はまったく何も起きませんし、効果もありません。

## Promise コンストラクタにおける例外発生

ただし、`throw new Error()` をコールバックの中で行った場合にも、それ以降のコードが実行されないことに注意してください。

そして、`Promise()` コンストラクタでの処理で例外が発生した場合は、自動的に Promise が reject されて例外がキャッチされます。従って、例外が発生したその Promise インスタンスは `reject()` 関数を呼び出したのと同じように拒否状態となります。

```js
// withoutReject.js
const promise = new Promise((_, reject) => {
  throw new Error("😭 例外発生する!");
  reject("👻 効果無し");
  console.log("😅 これは実行されない");
});

promise
  .then(() => console.log("😅 これは実行されない"))
  .catch((error) => console.log(error.message))
  .finally(() => console.log("👿 最後に実行される"));
```

これを実行すると次の結果を得ます。

```sh
❯ deno run withoutReject.js
😭 例外発生する!
👿 最後に実行される
```

## Promise コンストラクタ関数内の返り値は無視される

重要なこととして、`new Promise(executor)` のコールバック executor 関数では、`return` による**返り値そのものは無視されます**。従って、次のような Promise を返す関数から返ってくるのはあくまで `resolve()` された値を持つ Promise インスタンスであり、`return` された値は持っていません。

```js
// ignoredReturnedValue
const returnPromise = () => {
  return new Promise((resolve) => {
    resolve("😁 こっちが履行値");
    console.log("👻 resolveじゃ関数は止まらない");
    return "😭 これは履行値にならずに無視される";
    console.log("👿 関数は止まるから実行されない");
  });
};

returnPromise()
  .then((value) => console.log("🍓 履行値:", value))
  .catch((reason) => console.log("🥦 拒否理由:", reason))
  .finally(() => console.log("👍 チェーン最後に実行"));
```

これを実行すると以下のようになります。

```sh
❯ deno run ignoredReturnedValue.js
👻 resolveじゃ関数は止まらない
🍓 履行値: 😁 こっちが履行値
👍 チェーン最後に実行
```

`return resolve("履行値")` でも返り値は特に意味がなく、`resolve()` している値が重要となります。

`return` での返り値は何の意味もないので、次のようにした場合は、`returnPromise()` 関数から返ってくる Promise インスタンスの状態が永遠に Pending 状態となり Promise chain の処理は行うことができません。

```js
// forEverPending.js
const returnPromise = () => {
  return new Promise(() => {
    console.log("👻 これは永遠に Pending 状態の Promise インスタンス");
    return "😭 これは履行値にならずに無視される";
  });
};

returnPromise()
  .then((value) => console.log("🍓 履行されない", value))
  .catch((reason) => console.log("🥦 拒否されない:", reason))
  .finally(() => console.log("👍 チェーン最後に実行するはずができない"));
```

`returnPromise()` では resolve と reject のどちらも行っていないので、この関数から返ってくる Promise インスタンスの運命(Fate)は Unresolved であり、状態(State)も永遠に Pending 状態のままです。

```sh
❯ deno run foreverPending.js
👻 これは永遠に Pending 状態の Promise インスタンス
```

## Promise インスタンスで resolve する

ある Promise インスタンスで、別の Promise インスタンスを resolve してみたいと思います。

Promise インスタンスで resolve を試みると、resolve するのに使用した Promise インスタンスの**履行値や拒否理由が伝達されます**。

```js
// resolveWithPromise.js
const promise1 = new Promise((resolve, reject) => {
  if (Math.random() < 0.5) {
    resolve("🍎 promise1の履行値");
  } else {
    reject("🥦 promise1の拒否理由");
  }
});
const promise2 = Promise.resolve(promise1);
//   promise1 で resolve を試みる ^^^^^^^^

promise2
  .then((value) => console.log("😄 履行値:", value))
  .catch((reason) => console.log("😭 拒否理由:", reason))
  .finally(() => console.log("👍 チェーン最後に実行"));
```

半分ずつの確率で、履行または拒否となるので、実行すると以下のように結果を得ます。

```sh
❯ deno run resolveWithPromise.js
😭 拒否理由: 🥦 promise1の拒否理由
👍 チェーン最後に実行
❯ deno run resolveWithPromise.js
😄 履行値: 🍎 promise1の履行値
👍 チェーン最後に実行
```

`promise2` に対して Promise インスタンスで resolve を試みているため `promise2` 自体の Fate は Resolved です。そして、`promise2` 自体の履行値や拒否理由は `promise1` で `resolve()` や  `reject()` を行った値となります。

この様に resolve の行為は単に引数の値で Promise インスタンスを Fulfilled 状態にするものではありません。**Promise インスタンスで resolve した結果として Rejected 状態になることもあります**。`promise2` は `Promise1` の状態や履行値、拒否理由に対して自身の状態と値すべてを委ねています。

:::message
この挙動は仕様的には [CreateResolvingFunctions](https://tc39.es/ecma262/#sec-createresolvingfunctions) 抽象操作から作成される [Promise Resolve Functions](https://tc39.es/ecma262/#sec-promise-resolve-functions) に定義されています。`resolve` 関数の実体はこの関数です。詳細については『[Promise.prototype.then の仕様挙動](m-epasync-promise-prototype-then)』のチャプターで解説しています。
:::

`resolve(promise1)` でやったような Promise インスタンスの状態と履行値、拒否理由を解析して、自身の状態と値にするような能力を "**Unwrapping**" と言います。この Unwrapping の能力は `Promise.resolve()` や Executor 関数に渡す `resolve()` 関数にはありますが、`Promise.reject()` や Executor 関数に渡す `reject()` 関数にはありません。

```js
// resolveWithPromise.js
const promise1 = new Promise((resolve, reject) => {
  if (Math.random() < 0.5) {
    resolve("🍎 promise1の履行値");
  } else {
    reject("🥦 promise1の拒否理由");
  }
});
const promise2 = Promise.reject(promise1);
// promise2 は必ず拒否状態になる

promise2
  .then((value) => console.log("😄 履行値:", value))
  .catch((reason) => console.log("😭 拒否理由:", reason))
  .finally(() => console.log("👍 チェーン最後に実行"));
```

これを実行すると、`promise1` の状態がどうであろうと、`promise2` は拒否状態となります。

```sh
❯ deno run rejectWithPromise.js
😭 拒否理由: Promise { "🍎 promise1の履行値" }
👍 チェーン最後に実行
❯ deno run rejectWithPromise.js
😭 拒否理由: Promise { <rejected> "🥦 promise1の拒否理由" }
👍 チェーン最後に実行
error: Uncaught (in promise) 🥦 promise1の拒否理由
```

また、`promise1` の履行値や拒否理由を自身の履行値や拒否理由にすることはできずに、Promise インスタンスそのものを値として使ってしまっています。従って、Unwrapping の能力が無いことを確認できます。

参考
https://www.saurabhmisra.dev/promises-in-javascript-resolved-promise-fates
