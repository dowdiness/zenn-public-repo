---
title: "プルリクエストによる翻訳作業のやり方"
published: true
cssClasses: zenn
emoji: "👾"
type: "tech"
topics: [ "git", "github", "英語", "翻訳", "Obsidian" ]
date: 2021-03-28
modified: 2022-10-20
url: "https://zenn.dev/estra/articles/translate-with-gitandgithub"
tags: [" #type/zenn #git/GitHub #Zenn #翻訳 #obsidian "]
aliases:
  - git翻訳のまとめ記事
  - 記事_プルリクエストによる翻訳作業のやり方
---

# はじめに

[Obsidian](https://obsidian.md)というソフトウェアの UI とヘルプドキュメントの翻訳と保守を有志で行っています。翻訳作業を通してプルリクエストのやり方を学び、いい機会だと思ったので Git と GitHub の使い方からプルリクエストによって共同で翻訳作業をしていく方法を紹介していこうと思います。初学者の観点から git の概念やつまずいたところやコマンドの使い方、プルリクエストまでの流れなどをまとめています。次のような記事もあるので、翻訳作業を通して git の使い方をマスタするのは結構おすすめだと思います。

[翻訳から始めるオープンソースコントリビューション](https://qiita.com/yhay81/items/3773ab2e001a9e39ccc8)

次のような OSS(オープンソースコミュニティ)へのプルリクエストに関する記事なども参考になりますが、翻訳での継続的なプルリクエスト(2 回目、3 回目~)についての全体の流れがわかる記事はあまり見たことがありませんでしたので主にその点について書いてみました。

[【超図解】OSSにPull Requestを出す時の備忘録 - Qiita](https://qiita.com/y-vectorfield/items/b955617712f3b66359f2)

ドキュメントの翻訳作業をしてみたいが、やり方がよく分からないという方、Git 初心者に役立てばよいなと思います。

また、UI ではなくヘルプドキュメントの翻訳作業を念頭において書いています。というのもヘルプドキュメントは大量のファイルを扱うので、Github 上ではなくローカル環境でしっかりと作業する必要があります。ローカルでの翻訳作業からプルリクエスト作成を通して Git や Github の基本的な使い方がマスタできます。翻訳作業そのものに関しては素人なので注意してください。

実際に翻訳されたヘルプドキュメントが Obsidian の公式サイトで公開されています。↓
https://publish.obsidian.md/help-ja/

上のドキュメントで表現の向上(こうしたほうがよい)や誤訳、タイポなどを見つけた場合には公式リポジトリや Fork したリポジトリでもいいので issue かプルリクエストを作成してください。↓
[obsidianmd/obsidian-docs: Help documentation for Obsidian](https://github.com/obsidianmd/obsidian-docs/)

## 最低限必要な知識や事前に必要なもの

作業を始めるのに次の最低限必要な知識やアプリケーションなどは準備するようにしてください。

- CLI の使い方の基本
- Git のインストール
- github のアカウント作成
- vscode などのテキストエディタ

## 主なソース

- [エンジニアのためのGitの教科書](https://amzn.to/3dcc9h5) 
	- 薄くてわかりやすいのでおすすめです。わからなくなったら読むといいです。
- [Github で Fork してから Pull Request をするまでの流れ](http://kik.xii.jp/archives/179)
	- 最初のプルリクエストの流れが分かりやい記事ですが、2 回目以降については言及がなかったので補足します。また `rebase` や `push -f` など初心者が使わない方がよいです。
- [初めてのPull Request(プルリクエスト)](https://kaworu.jpn.org/kaworu/2017-10-19-1.php#2017-10-19-1-7a386224b42e28c840ef6ce67c51ca62)
	- OSS でのプルリクエストについての記事で、図はないですが細かく、分かりやすいです。

# つまずきポイントメモ

簡易的な箇条書きですが、初心者がつまずきやすいポイントややらかさないようにする注意点などを前もって書いておきます。

- リポジトリ(箱)とブランチ(枝)とコミット(点)の概念の理解
- 概念を理解していないと予期しない操作をしてしまう
- upstream と origin と local がごっちゃになるのでリポジトリが 3 つあることを理解する
- プルリクエスト承認後のながれ(2 回目以降のプルリクエスト)がわからない
- clone と fork の違い
- プルリクエストは Github 上で作成する(ターミナルからはやらない)
- pull は使わないで fetch して merge する
- checkout する前にブランチ状態を確認する
- add はファイルごとフォルダごとに行う
- コミットのハッシュ値は最初の数行のみ扱えば良い
- master ブランチは触らずに、なにかするときは作業ブランチを作成してそこで行う
- 強制プッシュは行わない
- rebase はしない(rebase してほしいと言われたときのみ行う)
- merge には種類があることを理解(ファストフォワードとリカーシブを理解)して使い分ける
- merge や fetch する際にはどこのブランチにいるか確認してから行う
- Github でプルリクエストのボタンが消えたときの対処
- 別のブランチに push しないように気をつける
- アカウントプライバシーを設定してから作業する

やらかしたときはネットで調べまくりましょう。
[Gitでやらかした時に使える19個の奥義 - Qiita](https://qiita.com/muran001/items/dea2bbbaea1260098051#%E3%82%84%E3%82%89%E3%81%8B%E3%81%97%EF%BC%91%EF%BC%96%E3%83%AA%E3%83%A2%E3%83%BC%E3%83%88%E3%81%8B%E3%82%89pull%E3%81%97%E3%81%9F%E3%82%89%E3%82%B3%E3%83%B3%E3%83%95%E3%83%AA%E3%82%AF%E3%83%88%E3%81%97%E3%81%BE%E3%81%8F%E3%81%A3%E3%81%9F%E3%81%AE%E3%81%A7%E4%B8%80%E6%97%A6%E5%85%83%E3%81%AB%E6%88%BB%E3%81%97%E3%81%9F%E3%81%84%E6%99%82)

#  周辺概念

最初に概念の説明をしておこうと思います。長くなってしまったので概念が理解できている方はこのセクションはとばしてください。

リポジトリとは雑にいうとフォルダです。Git を使うとそのリポジトリ内部(フォルダ)でさまざまな変更を行い特定の時点の状態そのものをある点(commit)として保存管理できます。プルリクエストとはそのように自身のリポジトリ内にて行った変更を、Github 上(ウェブ上)にある他のリポジトリに対して統合(merge)するように申請する仕組みです。

プルリクエストを作成してマージ(変更の統合)を行うまでの流れには、**計3つのリポジトリを使用します**。この 3 つというのがなぜ必要なのかが最初分かりづらかったので注意してください。

必要なリポジトリとして、1 つ目は公式のリポジトリ(オリジナルのリポジトリ)で、これを upstream という名前で便宜的に識別します。2 つ目は Github の自分のアカウント上に upstream の複製リポジトリを作成します。この行為を fork すると呼びます。この fork によって作成されたリポジトリを origin として識別します。origin や upstream はリポジトリの名前として考えてください。この origin と upstream はユーザーにとってはオンライン上に存在するので「**リモートリポジトリ**」と呼びます。

3 つ目は、自分の PC 上での作業のために origin リポジトリから更にローカル環境に複製リポジトリを作成(clone)します。clone による複製と fork による複製の違いに注意してください。基本的には、この複製されたリポジトリで編集作業を行っていきます。ローカル環境上にあるリポジトリなので先程のリモートリポジトリに対して「**ローカルリポジトリ**」と呼びます。このリポジトリを origin や upstream と同様に local という名前で識別します。この 3 つのリポジトリを使ってプルリクエストによる共同編集を行います。

upstream は自分が管理するリポジトリではなく、他者が管理するオリジナルのリポジトリです。origin は upstream から複製された自分が管理するリポジトリなので自由に編集できます。しかし、実際に触るのは origin ではなく「**ローカル上のリポジトリであるlocal**」です。このあたりが紛らわしいのですが、オンライン上で作業するよりもローカルで作業するほうが軽く、常にネット接続している必要もありません。ゆえに*リポジトリlocalで編集作業を行い、その変更をもとのリポジトリupstreamに統合することによって共同で翻訳作業を行っていくわけです*。ですが local からいきなり upstream に対して変更を申請することはできません。local から自分の管理するリモートリポジトリ origin に一度変更を反映させる必要があります。その後、origin から upstream に対して変更の統合を申請するというのがプルリクエストの流れになります。

このように、リポジトリの複製の流れや内容変更の反映の流れが複雑になるのでつまづきポイントが結構あります。下でそれぞれの流れを図示しているので参考にしてください。

まとめると

| リポジトリの識別名 | リポジトリの内容                                         | 備考                         |
| ------------------ | -------------------------------------------------------- | ---------------------------- |
| upstream           | オリジナルのリモートリポジトリ                           | 他人が管理                   |
| origin             | forkして複製した自分のアカウントにあるリモートリポジトリ | 自分が管理                   |
| local              | 自分のローカルマシンにcloneしたローカルリポジトリ        | 自分が管理、ここで編集を行う |

ここにブランチという概念がさらに追加されます。リポジトリでは特定の時点の情報の状態をコミットという形で保存しています。このコミットからさらに変更を加えて意味ある形で再びコミットを作成していくと、コミットが時系列によって連結されたブランチ(枝)ができあがります。枝のある点から全く別の点を派生させることで枝を分岐させることができます。

イメージでまとめると、箱(リポジトリ)の中に点(コミット)が時系列に連結した枝(ブランチ)が存在しているような感じです。分岐して存在する複数の枝の先端の点は枝そのものの名前として扱われます。master ブランチやそこから派生させてつくった作業ブランチなどです。イメージで言うと、プルリクエストというのはリポジトリ upstream の主要な枝(master ブランチ)の先頭に自分が編集した変更情報を点として(コミットとして)連結する操作です。

ブランチは各リポジトリごとに存在します。つまり、**upstreamのmasterブランチ、originのmasterブランチ、localのmasterブランチというような感じで重複した名前のブランチが存在**しますが、それぞれは別のものなので注意してください。

結局のところ、翻訳作業で実際に触るのはリポジトリ local ですが、local の master ブランチは基本的には触りません、**masterブランチから派生させた作業ブランチ**(自分で名前をつける)上で編集します。

結局、言葉で説明しても限界があるので検索して図などで理解するのがよいと思います。また実際に Git を使って**ブランチ操作**をやってみるとわかってくると思います。

参考: [マンガでわかるGit 12話「本家リポジトリに追従する方法」](https://next.rikunabi.com/journal/20180322_t12_iq/)

---

では、実際にプルリクエスト作成までの流れを説明していきます。最初に流れを時系列順に並べます。その後に具体的な操作について説明していきます(ブラウザ上での操作とターミナル上での操作があることに気をつけてください)。

# (1) 最初のプルリクエストまでの流れ

0. Github のアカウントプライバシーを設定し、Git で日本語ファイルが文字化けをしないように設定する
1. 公式リポジトリから自分のアカウントのリポジトリに fork する
2. ローカル環境に fork したリポジトリを clone する
3. コーディングスタイルを確認
4. 作業ブランチを作成してチェックアウト
5. 作業ブランチにて翻訳開始
6. ある程度の作業まとまりで add/commit する
7. orgin に push する
8. Github にてプルリクエストを作成し、承認を待つ
9. プルリクエストに修正があれば修正後 origin に再び push する

![1-プルリクの流れ1](https://storage.googleapis.com/zenn-user-upload/aq1b7im42nlm4376fudurn36k92x)

## 実際のやり方

上の 0〜9 までの流れを説明していきます。

共同で編集するということで、まず Github のアカウントプライバシーの設定を行います。デフォルトの状態だと PC のユーザー名と Github で登録してアカウントが commit log に表示されてしまいます。このままプルリクエストを出して承認されてしまうと面倒です(表示されてもよいという方はやらなくて結構です)。

参考: [GitHubにホストされているGitからの個人情報流出を防ぐために - ツレヅレナルママニ](https://tkhs0604.hatenablog.com/entry/github-email-exposure)

次に Git で日本語ファイルの文字化けを回避するためにコマンドを打ちます。

```shell
git config --global core.quotepath false
```

それでは最低限準備したので、本格的に作業を開始します。まず upstream を fork して origin を作成します。Github のリポジトリページから右上の fork ボタンをクリックすると自分のアカウントにリポジトリが複製されます。

![2-Githubでフォーク](https://storage.googleapis.com/zenn-user-upload/hisqw3tezev16w87t3mver3ax1kk)

![3-frokで作成された自分のリポジトリ](https://storage.googleapis.com/zenn-user-upload/5ldouqa8nd0ex0y1jed65uqpy6qp)

![3-クリップボードにコピー](https://storage.googleapis.com/zenn-user-upload/jwq8xcxsra470ka8nh8r8kcv3y85)

ここまで Github でできたら、ターミナルを開き、作業ディレクトリを作成しフォークした自分のアカウントにあるリポジトリをクローンします。

```shell
git clone リポジトリのURL
git clone https://github.com/yo-goto/obsidian-docs.git
# ローカルにリポジトリをclone
git branch translation
# 作業ブランチtranslationの作成
git checkout translation
# 作業ブランチにチェックアウトする
git branch -a 
# ブランチの確認
```

作業ブランチにチェックアウトしたら Obsidian や VSCode でリポジトリを開き実際に翻訳作業を開始します。編集するフォルダは日本語用のフォルダのみとして他の言語フォルダを編集しないよう注意してください。

翻訳作業が一通り終わったら `git add` でステージングして、`git commit` して保存します。日本語のフォルダのみ変更を加えるため add するのは日本語フォルダ(Obsidian の場合だと `ja`)以下となります。つまりフォルダごとに add する場合には `git add ja/FolderName` と打つようになります。`git add -A` などのコマンドを利用してしまうと他の言語フォルダを間違って編集してしまった場合にそれらの間違った変更も追加してしまうことになるので `git add -A` などは使わずにフォルダ単位、ファイル単位で add を行うように気をつけてください。

```shell
git add ja/folderName_1
git add ja/folderName_2
# ファイルごと、フォルダごとにaddする
git committ -m "ja: update"
# コミットメッセージを添えてコミット
git push origin translation
# originの作業ブランチtranslationにpush
```

origin(リモートリポジトリ)の作業ブランチに push したら Github にてプルリクエストを作成し送信します。master ブランチではなく作業ブランチ translation からプルリクエストを作成する点に注意してください。「Compare & pull request」ボタンを押して、内容に関するコメントを適度に書き承認を待ちます。

![プルリク説明0-1](https://storage.googleapis.com/zenn-user-upload/zgbrrqjax0ys4xx31d60e9w59ggp)

master ではなく push した作業ブランチになっていることを確認してください。作業ブランチになっていれば「Compare & pull request」ボタンをクリックしてください。更新内容について簡単に説明コメントを記載して「Create pull request」ボタンを作成すればプルリクエストの作成完了となります。

![プルリク説明7 コメント](https://storage.googleapis.com/zenn-user-upload/9zbqm2srwaz6ryogpdqrym7smlwn)

プルリクエストが承認されるまでは次のようなページが表示され「open」状態になります。リポジトリの管理者や他のコントリビューターからプルリクエストについてのコメントがあれば下に表示されます。

![プルリク説明6](https://storage.googleapis.com/zenn-user-upload/mhb49o175ooz4p6aeymv44pgnv69)

あとは承認される(マージされる)のを待ちつつ作業するか、作業を中断します。

`git push` した後、時間がたってしまうと「Compare & pull request」が非表示になる場合があります。これであわててしまうことがあるので注意してください。非表示になっている場合には「Pull requests」のセクションに進み、右側にある「New pull request」ボタンをクリックしてください。

![プルリク説明1 非表示](https://storage.googleapis.com/zenn-user-upload/xctt21ypb90rv6ga0kja2022a4p1)

フォークを利用したプルリクエストを作成する場合にはデフォルトの状態ではできないので、「compaer across forks」のリンクをクリックして、比較するブランチを upstream ではなく origin の作業ブランチにします。

![プルリク説明2 非表示](https://storage.googleapis.com/zenn-user-upload/r95hpiq5oo78qlgnmcijy7g01i9t)
![プルリク説明3 非表示](https://storage.googleapis.com/zenn-user-upload/od1ug234k6na04r0q4otevv3xy0g)
![プルリク説明4 非表示](https://storage.googleapis.com/zenn-user-upload/fxaybjrca4mtr8qvv6sdt6r1tubm)

ここまでやって「Create pull request」ボタンをクリックすることでプルリクエストが作成できるようになります。

![プルリク説明5 非表示](https://storage.googleapis.com/zenn-user-upload/w0z2ybbc45apm1q4fghmz0ecsu77)

プルリクエスト作成後に内容を修正する場合には、そのままの状態でローカルで修正を加えた後で同様に add/commit して origin の作業ブランチ translation に再び push します

```shell
git add ja/修正を加えたフォルダやファイル名
git commit -m "コミットメッセージ"
git push origin translation
# originの作業ブランチに再度pushする
```

これによってプルリクエストの内容が自動修正されます。

![プルリク説明9 修正後に再push](https://storage.googleapis.com/zenn-user-upload/ad2d2dk4rww86hezqlhy9kmuw0lm)

プルリクエストが承認されたら(マージされたら)、公式のオリジナルのリポジトリ(upstream)の master ブランチが自分の作成したプルリクエストの内容がマージされた状態となります。

![プルリク説明10 プルリク承認](https://storage.googleapis.com/zenn-user-upload/3iwcvfo1o1obz0vbhxrqhaqkwtuz)

基本的にはこの流れで作業終了となりますが、ヘルプドキュメントでは更新は定期的にあるため、時間が経過したら再び翻訳内容が追加され、再び翻訳作業をしプルリクエストを作成することになります。2 回目以降はいままで流れと若干異なり、ローカルのブランチを更新する作業がでてくるので注意してください。

# (2) 2回目以降のプルリクエストの流れ

9. upstream の追加と確認
10. ローカルの master ブランチを upsteream の最新に追いつかせる
11. origin の master に push
12. (a)master の最新内容を前の作業ブランチに merge して反映してからチェックアウト、または (b)要らない作業ブランチを削除し作業ブランチを作り直しチェックアウト
13. 更新された内容との差分を確認しながら翻訳再開する
14. add/commmit した後に origin に push する
15. Github にてプルリクエストを作成し、承認を待つ
16. プルリクエストに修正があれば修正後 origin に再び push する

![プルリクエストの流れ3](https://storage.googleapis.com/zenn-user-upload/wwv11qft5vaoe7ff5rig8xkfz6i5)

## 実際のやり方

上の 9〜16 までの流れを説明していきます。

### 翻訳作業再開するまで

作業を再開するために、ローカルリポジトリ(local)の master ブランチの内容をこのオリジナルのリモートリポジトリ(upstream)の master ブランチの状態に追いつかせる必要があります。

まず remote の設定を確認し、upstream として追加します。次のコマンドで upstream という名前でオリジナルのリモートリポジトリを追加します。*この作業は3回目以降のプルリクエストからは必要ありません*。

```shell
git remote -v
# fork元の公式オリジナルのリモートリポジトリがremoteのupstreamとして設定しているか確認する。
git remote add upstream fork元のURL
# upstreamとして設定する
git remote add upstream https://github.com/obsidianmd/obsidian-docs.git
```

upstream から最新を fetch して local の master ブランチにマージしてあげます。local の master ブランチが最新になったら自分のリモートブランチ(origin)の master に push して反映させます。上で提示した 9〜16 までの流れの 12(a)になります。

```shell
git checkout master
# masterブランチにチェックアウトする
git fetch upstream
# upstreamから更新情報をとりよせる
# 更新情報は upstream/master に
git merge --ff-only upstream/master
# ファストフォワードマージを行い、リポジトリlocalのmasterブランチを最新にする
git push origin master
# 自分のリモートリポジトリ(origin)のmasterブランチにその最新内容を反映させる
```

`git mrege` コマンドでは、オプション `--ff-only` をつけることでファストフォワードマージでマージを実行します。

### 翻訳作業再開

```shell
git checkout translation
# 作業ブランチにチェックアウトする
git status
# 作業ブランチに変更がないか調べる(もしなにかあればコミットしたり、stashしたりして退避させる)
git log --all decorate --graph --oneline
# グラフを出してコミットのハッシュ値を確認できる。作業ブランチtranslationのハッシュ値をしらべる。冒頭の数桁のみでよい
git show HEAD
# これでも最新のコミットのハッシュがわかる(あとでハッシュ値を使う)
git merge --no-ff master
# 最新内容であるlocalのmasterを作業ブランチにリカーシブマージさせる(これを行うとマージメッセージの入力画面がvimモードででるので適当なメッセージを書いて終了)
```

マージを `--no-ff` オプションで行うとリカーシブマージになります。これを行うと Vim が立ち上がりマージコミットのメッセージを書くことになります。Vim の使い方(挿入モードや保存終了の方法)を簡単に調べておくとよいです。

前回から更新された内容を確認するため差分を表示させます。上で調べた前回コミットのハッシュ値と master ブランチとの差分を確認します。

```shell
git diff ハッシュ値 master
# これでも差分を確認できる。ハッシュ値は冒頭の数桁の数字をいれればよい
git diff 78d63a7 master en/
# masterブランチとコミット78d63a7間でenフォルダについての差分を確認できる
git diff 78d63a7 master en/Pulgins/
# Pluginフォルダの内容だけ差分を見る
git diff 78d63a7 master en/Pulgins/test.md
# test.mdのファイルの差分を見る
git diff 作業ブランチ master en/
# ブランチで比較
```

更新内容が複数のファイルや追加内容が多い場合にはターミナルで diff を使うよりも[Githubのcompare機能](https://docs.github.com/ja/github/committing-changes-to-your-project/comparing-commits)や VSCode などの差分表示機能(特に[GItLens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens)などの拡張機能)を使った方がよいですが、なれるまでは `git diff` を使ってひとつずつやる方が後学のためになると思います。

![プルリク説明11 差分表示](https://storage.googleapis.com/zenn-user-upload/mn6hy4y4ghg95u1ll11grgm1vh6f)

差分を確認しながら作業ブランチにて追加内容分の翻訳作業をすすめていきます。ターミナル上で `git diff` を使って差分表示させると修正前と修正後が赤と緑で表示されるので、その内容をみながら日本語版での内容を追加修正していきます。Obsidian の場合、英文(原文)のドキュメントフォルダ(en/)と日本語などの他言語用のフォルダ(ja/など)は別れています。なので diff を使うときには英語のドキュメントフォルダを対象に差分を見ます。その差分から追加された部分を見て、翻訳を開始します。

#### なるべくマージしないで作業する方法

作業ブランチに master をマージするのではなく、作業ブランチを毎回新しく作りなおす方法を行ったほうがわかりやすいことがあります。個人的に慣れるまではこの方法を行っていたので、マージの意味がよくわかっていない場合には、こちらの方法をとるのがよいです。上で提示した 9〜16 までの流れの 12(b)です。(a)を行わない代わりに(b)をやるという感じですね。

プルリクエスト承認後、`git merge --no-ff master` をやらずに次の操作を行います。

```shell
git checkout master
# masterにチェックアウト
git branch -d translation
# プルリクエストが承認されたので作業ブランチを削除してしまう
git branch -a
# ブランチが削除されたことを確認
git branch translation
# 作業ブランチを最新のmasterから作り直す(分岐させる)
git checkout translation
# これで最新内容になった作業ブランチで作業再開できる
```

マージの意味が理解できたら、いちいち作業ブランチを破棄して作り直す方法をとらずに、マージによって作業ブランチの内容を最新にすればよいです。

### 作業終了後にプルリクエスト作成

初回のプルリクエストまでの流れと同じく add/commit 後に自分のリモートリポジトリ(origin)の作業ブランチに push します。

```shell
git add ja/folderName_1
git add ja/folderName_2
# ファイルごと、フォルダごとにaddする
git committ -m "ja: update"
# コミットメッセージを添えてコミット
git push origin translation
# originの作業ブランチtranslationにpush
```

push を終えたら同じように Github で作業ブランチ translation からプルリクエストを作成し承認を待ちます。

## 流れのまとめ

シーケンス図でのまとめです。

![シーケンス図](https://storage.googleapis.com/zenn-user-upload/0smd4210ozbpr3a8osgszx14g4h9)

# (3) 慣れたらやってみること

## プルリクエストの分割

別々の意図でプルリクエストを分割できる。ローカルブランチを複数きって別々に `git push` を行い、それぞれでプルリクエストを作成することでプルリクエストの分割を行えます。内容追加と修正でプルリクエストを分割するなどができるようになります。

## Git clientの利用

Working Copy などの Git client を利用してモバイルで作業すると暇なときに翻訳作業を行うことができます。こちらでもメールアドレスを Github で設定した同一のもにする必要があります。

## vscodeの拡張機能で作業効率アップ

GitLens や GitGraph などの拡張機能を入れることで作業効率を上げることができます。

![差分を見ながら追記](https://storage.googleapis.com/zenn-user-upload/xz9q1mkpehaeehd8iqhvvg9ogdsi)

GitLens を利用すればブランチ間(コミット間)の差分を簡単に確認しながら翻訳作業を行えます。更新された部分のみを見ながら翻訳を追加できます。

# 他によく使うコマンドの概説

```shell
git checkout -b branchName
# ブランチを新規作成した上でチェックアウトする

git merge --no-ff branchName
# non fast-forward(recursiveマージ)
git merge --ff-only branchName
# fast-forwardマージできるならマージする、できない場合にはマージしない

git rm fileName
# git監視下でファイル削除
git mv 旧ファイル名 新ファイル名
# ファイルの名称変更
git mv fileName folderpath
# ファイルの移動

git config --global list 
# globalのconfig確認(ユーザー情報などの確認)
git log --graph --oneline
# ログをグラフとして簡易表示

```

リモートブランチ関連
```shell
git push --delete origin branchName
# リモートブランチの削除
git checkout -b branchName origin/branchName
# リモートブランチをローカルにcheckout
```

# 翻訳そのものについて

## 自由にやると結構楽しい

翻訳の厳密性はプロジェクトごとに色々あると思いますが有志の翻訳であれば、ある程度のゆるさがあっても大丈夫だと思います。あまり気にしすぎずに挑戦してみるとよいと思います。分からない表現や専門用語はどんどんネットで検索しましょう。自分の場合には、不自然な日本語になるくらないなら解釈から相当する自然な言い回しを優先しています。文も切ったりつなげたり、結構自由にやっています。参加者の少ない日本語翻訳のプロジェクトの場合、このように自分の好きなようにできるのも利点だと思います。一方、人数が少ないと校正や、表現性の向上、誤訳やタイポなどの修正があまりできないのが難点です(textlint を使うのがいいかもしれません)。

Obsidian の翻訳プロジェクトでは、翻訳に問題や修正、追加内容があれば誰でもプルリクエストや issue を Github に作成できるスタイルでやっているので翻訳作業自体は結構はやいスピードで進みます。もちろんニッチなソフトウェアのベータ版という開発途中のもの故にこのような自由な事ができるわけで、他のメジャーなソフトウェアやサービスではなかなか難しいのではないでしょうか。

他の翻訳プロジェクトってどうやってやっているのかと思って他のサービスなどちょっと調べてみました(スケールによって結構違いますねやはり)。

- [Notion Community Translation Guide](https://www.notion.so/Community-Translation-Guide-d1a37aa2e22f4c9daf2a1efe9ae13bc2)
- [Hugo Docs Japanese | Hugo Documentation の和訳・日本語翻訳プロジェクト](https://hugojapan.github.io/)
- [yohamta/typescript-book-jp: TypeScript Bookの日本語訳です。](https://github.com/yohamta/typescript-book-jp)

## 日本語表現が難しい

英文の解釈よりも日本語の表現が難しいです。英語で理解できても日本語自体の表現や語彙のストックがないので(笑)、どうしても直訳っぽくなってしまい、他のアプリケーションやサービスのドキュメントの日本語訳版などを参考にすることもありました。また、他の言語翻訳者がどのように翻訳しているのか Google 翻訳などを使って見ていました(例えばイタリア語やロシア語など)。

実際のところ、まだ稚拙な表現が多いかもしれませんが、現在は「とりあえず翻訳を完成すること」を念頭にプルリクエストで修正点や校正などをはさみつつクオリティを上げていくようにしています。

## UIとヘルプドキュメントの一貫性

また、自分の場合はヘルプドキュメントから翻訳をはじめましたが途中から UI との整合性を図るためにヘルプドキュメントでは UI をベースとした説明文になるように注力しました。UI の翻訳では json ファイルの文字列を翻訳していく形でしたが、英文の文字列を見てもどういった操作なのかわからないことが多いので、実際に Obsidian の画面を出してどの部分の文字列なのか確認しながらやらないといけませんでした。

Obsidian は Electron ベースで開発されているので developer tool が使えます。翻訳であやしい部分があれば UI にあっているか、developer tool で実際に変えてみて合うかどうかを確認しています。

他に、特段注意すべきことは、句読点やスペースや半角全角などの表記ルールや語彙対応表などを早いうちから作成しておくと良いです。あとから整合性を整えるのは面倒なので最初から表現や単語のルールを統一しておきましょう。UI との整合性もを考える上で重要です。といってもいきなりルールを想定することはできません。その場合は翻訳しているうちに必要なルールがどんどんでてきますので、その都度はやめに追加していくのがよいです。表記のスタイルについては[Wrodpressの翻訳スタイルガイド](https://wordpress.com/ja/support/translation-style-guide/)や[JTF(日本翻訳連盟)](https://www.jtf.jp/tips/styleguide)の翻訳スタイルガイドなどが参考になります。ルールを新たに追加する際には既存のドキュメントに対してルールを追加するためテキストエディタの検索と置換機能を駆使して全体に適応します(これも textlint でできそうです。プリセットルールで JTF-style なるものが存在します)。

# 他参考サイト

[Gitをインストールしたら真っ先にやっておくべき初期設定 - Qiita](https://qiita.com/wnoguchi/items/f7358a227dfe2640cce3)
