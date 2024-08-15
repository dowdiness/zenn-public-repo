---
title: "第２部 - Promise インスタンスと連鎖"
cssclass: zenn
date: 2022-06-16
modified: 2024-08-14
AutoNoteMover: disable
tags: type/zenn/book, JavaScript/async
aliases: EPAsync 第２部
---

## この部について

第２部では Promise インスタンスと Promise chain について見ていきます。Promise chain は非同期処理の処理予測を行うためのメンタルモデルを構築する上で基礎となる非常に重要な概念です。Promise chain の具体的なコード実行を通じて非同期処理の実行順序を予測できるようにこの部で訓練していきます。

この部の内容をしっかり学ぶことで、[第３部](part-03-epasync)の async/await を理解できるようになります。

:::message
この本の執筆経緯的にも Promise chain についての解説である第２部をはじめに執筆しており難易度としては簡単なので、[第１部](part-01-epasync)の内容が難しい場合には第２部から読み始めるのも一つの手です。
:::

## 間違いについて

この部にかつてあった間違った解説については別個で新しいチャプター『[Promise.prototype.then の仕様挙動](m-epasync-promise-prototype-then)』を用意して解説しています。

影響を受けていたチャプターについても修正を施したため、上記のチャプターは番外編というポジションに位置づけました(ECMAScript 仕様そのもの解説を含むため)。

## チャプター

- [Promise の基本概念](a-epasync-promise-basic-concept)
- [Promise コンストラクタと Executor 関数](3-epasync-promise-constructor-executor-func)
- [コールバック関数の同期実行と非同期実行](4-epasync-callback-is-sync-or-async)
- [resolve 関数と reject 関数の使い方](g-epasync-resolve-reject)
- [複数の Promise を走らせる](5-epasync-multiple-promises)
- [then メソッドは常に新しい Promise を返す](6-epasync-then-always-return-new-promise)
- [Promise chain で値を繋ぐ](7-epasync-pass-value-to-the-next-chain)
- [then メソッドのコールバックで Promise インスタンスを返す](8-epasync-return-promise-in-then-callback)
- [Promise chain はネストさせない](9-epasync-dont-nest-promise-chain)
- [コールバックで副作用となる非同期処理](10-epasync-dont-use-side-effect)
- [アロー関数で return を省略する](11-epasync-omit-return-by-arrow-shortcut)
- [Promise.prototype.then の仕様挙動](m-epasync-promise-prototype-then)
- [catch メソッドと finally メソッド](h-epasync-catch-finally)
- [古い非同期 API を Promise でラップする](12-epasync-wrapping-macrotask)
- [イベントループは内部にネストしたループがある](13-epasync-loop-is-nested)
