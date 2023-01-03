---
title: "リポジトリ作成時に自動でIssueを立てるには"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["github", "githubactions", "ci"]
published: false
---

この記事は [KMCアドベントカレンダー2022](https://adventar.org/calendars/8005) の22日目の記事です。ギリ遅刻です。

# はじめに

2023年、あけましておめでとうございます。Zenn をご覧の皆さまであれば普段から GitHub を使っていると思いますが、リポジトリの作成時に自動で Issue を立てたいと思ったことはありませんか？

ありますよね。

今回は自動化オタクの皆さまのためにとっておきの方法をご用意しました。この記事では、GitHub の [テンプレートリポジトリ](https://docs.github.com/ja/repositories/creating-and-managing-repositories/creating-a-template-repository) および GitHub Actions を使用して、表題の機能を実現します。

:::message
この記事は GitHub Actions を多少利用したことがあるものと想定して執筆しているため、GitHub Actions の基本的な説明については省略します。
:::

はじめに、テンプレートリポジトリについて軽く説明しておきたいと思います。

# テンプレートリポジトリとは

テンプレートリポジトリとは、その名の通りリポジトリをテンプレート化する GitHub の機能です。
一般的に GitHub 上でリポジトリを作成した場合は、Initial commit は行われないか、あるいは `README.md` や `LICENSE` などのファイルを含む commit となります。しかし、いつも使う開発環境や設定ファイルは初めから用意されていたほうが便利ですよね。テンプレートリポジトリからリポジトリを作成すると、テンプレートリポジトリの内容を Initial commit としてリポジトリが作成される[^1]ため、必要なライブラリやパッケージなどの開発環境をテンプレートリポジトリとして用意しておくことで、スムーズに開発を進めることができるわけです。
テンプレートリポジトリは、リポジトリの `Settings` タブから簡単に設定可能であり、すぐに使い始めることができます。

![Settingsタブのテンプレートリポジトリ設定が有効化されている状態](/images/create-a-issue-automatically/template-repository-setting.jpg)

# テンプレートリポジトリでは Issue をテンプレート化できない

さて、ここで表題にある通り Issue を自動で立てる方法について考えていきましょう。先程述べた通り、テンプレートリポジトリを用いることでリポジトリ作成時の中身を自由に設定できます。しかし、リポジトリ作成時に自由な内容の Issue を作成する方法は用意されていません[^2]。
そこで、GitHub Actions を用いてこれを実現しましょう。

# 用意するワークフローの概要

GitHub Actions のワークフローは `.github/workflows` ディレクトリで定義されており、テンプレートリポジトリからリポジトリを新規作成した場合は当然 `.github/workflows` ディレクトリの内容も引き継がれます。**リポジトリ作成時に Issue を立てるワークフロー**を定義しておくことで、自動で Issue を作成することができます。

ではこのワークフローを実装するにあたって、

1. リポジトリ作成時に
2. Issue を立てる

の2段階で考えていきましょう。

# 1. リポジトリの作成時にのみジョブを実行する

## 処理の概要

ワークフローは、ワークフロートリガーと呼ばれるワークフローの実行を引き起こすイベントが発生すると実行されます。ワークフロートリガーとして、「commit や tag が push されたとき」・「issue が作成、変更されたとき」・「branch や tag が作成されたとき」などの様々なイベントが用意されています[^3]が、リポジトリの新規作成はワークフロートリガーとして用意されていません（それはそう）。
では、テンプレートリポジトリからのリポジトリの作成を検知するにはどのようにすれば良いでしょうか。ここで、リポジトリの新規作成時は commit が1度しか行われていない（Initial commit のみ）という点に着目します。

そこで、**push をトリガーした上で commit が1度しか行われていないことを確認する**という方法で、テンプレートリポジトリからのリポジトリ新規作成を検知することにしましょう。

## 実装

ここまでを実装すると以下のようになります。

```yaml:.github/workflows/create-issue.yaml
name: create template issue
on:
  - push
jobs:
  create-template-issue:
    runs-on: ubuntu-latest
    steps:
      - name: Check out the repo
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - name: Get parent hash
        run: echo "PARENT_HASH=$(git rev-parse HEAD^@)" >> $GITHUB_ENV
      - name: Create template issue
        if: env.PARENT_HASH == ''
        # 以下、リポジトリ新規作成にのみ実行させる処理を記載
```

まずはいつも通り `actions/checkout` でチェックアウトしますが、ここで `fetch-depth` を明示的に指定している点に注意してください。`actions/checkout` の `fetch-depth` は既定値が1、すなわちデフォルトでは先頭の commit のみ取得します[^4]が、今回は commit 数が1を超えているかどうかを判定したいため、先頭の commit だけでは不十分です。`fetch-depth: 0` とすることで、すべての commit を取得することができます。

続いて、commit が1度しか行われていないことを確認する方法について確認します。

方法はいくつか考えられ、例えば `git log --pretty=oneline` の出力行数を数えることで commit の回数を調べられます。`git log` の出力終端に改行がないことによって `wc -l` で数えた場合にカウントが1少なくなってしまうので、`grep -c ''` で数えたほうが良さそうです。

```bash
$ git log --pretty=oneline | wc -l
# 0
$ git log --pretty=oneline | grep -c ''
# 1
```

他の方法として、`git rev-parse` を用いることができます。

```bash
$ git rev-parse HEAD^@
```

`HEAD^@` は、最も初めの commit を表しており[^5]、このコマンドの出力は1番最初の commit の hash を返します。このコマンドは commit が1つしかない場合のみ `""` を返すため、commit 数が1つであるかを判定するのに使用できます。今回は `git rev-parse` をワークフローに使用しています。

続いて2ステップ目を確認します。ここでは `$GITHUB_ENV` を使用し[^6]、環境変数 `PARENT_HASH` に最初の commit の hash を入れています。

最後に `if` [^7]を用いて、環境変数 `PARENT_HASH` を評価[^8]しています。先ほど述べた通り、リポジトリ新規作成時のみ `PARENT_HASH` が空白になります。

# 2. GitHub Actions を利用して Issue を立てる

[JasonEtco/create-an-issue](https://github.com/marketplace/actions/create-an-issue) を使用します。このアクションを使用すると、予め用意した Markdown ファイルの内容で Issue を作成できます[^9]。

用意する Markdown ファイルの書き方は、[Issue テンプレート](https://docs.github.com/ja/communities/using-templates-to-encourage-useful-issues-and-pull-requests/configuring-issue-templates-for-your-repository) で使用するものと同じです。

## 実装

では、以下のような Markdown ファイルを用意してこの内容で Issue を立ててみましょう。

```md:.github/issues/license.md
---
name: 'license'
title: 'ライセンスの設定'
labels: ''
assignees: ''
---

# 概要
`LICENSE` ファイルを入れる
```

ワークフローの実装は以下のようになります。

```yaml:.github/workflows/create-issue.yaml
name: create template issue
on:
  - push
permissions:
  contents: read
  issues: write 
jobs:
  create-template-issue:
    runs-on: ubuntu-latest
    steps:
      - name: Check out the repo
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - name: Get parent hash
        run: echo "PARENT_HASH=$(git rev-parse HEAD^@)" >> $GITHUB_ENV
      - name: Create template issue
        if: env.PARENT_HASH == ''
        uses: JasonEtco/create-an-issue@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          filename: .github/issues/license.md
```

:::details Diff
```diff yaml:.github/workflows/create-issue.yaml
 name: create template issue
 on:
   - push
+permissions:
+  contents: read
+  issues: write 
 jobs:
   create-template-issue:
     runs-on: ubuntu-latest
     steps:
       - name: Check out the repo
         uses: actions/checkout@v3
         with:
           fetch-depth: 0
       - name: Get parent hash
         run: echo "PARENT_HASH=$(git rev-parse HEAD^@)" >> $GITHUB_ENV
       - name: Create template issue
         if: env.PARENT_HASH == ''
-        # 以下、リポジトリ新規作成にのみ実行させる処理を記載
+        uses: JasonEtco/create-an-issue@v2
+        env:
+          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
+        with:
+          filename: .github/issues/license.md
```
:::

これで、リポジトリの新規作成時に自由な内容で Issue を作成できるようになりました。

![リポジトリの新規作成時に自動で Issue が作成された様子](/images/create-a-issue-automatically/issue-created-by-action.jpg)

# 複数の Issue を立てる

さて、ここまででめでたくリポジトリの作成時に Issue を立てられるようになったわけですが、最後に複数の Issue をまとめて立てられるようにしましょう。

もちろん、Issue の数だけステップを書くこともできますが、これはあまりに不格好です。

```yaml
      - name: Create first template issue
        if: env.PARENT_HASH == ''
        uses: JasonEtco/create-an-issue@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          filename: .github/issues/issue-1.md
      - name: Create second template issue
        if: env.PARENT_HASH == ''
        uses: JasonEtco/create-an-issue@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          filename: .github/issues/issue-2.md
      - name: Create third template issue
        if: env.PARENT_HASH == ''
        uses: JasonEtco/create-an-issue@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          filename: .github/issues/issue-3.md
      # ...
```

そこで利用するのが、[matrix](https://docs.github.com/ja/actions/using-jobs/using-a-matrix-for-your-jobs) です。matrix を利用することで、ワークフロー内で配列と同じように利用可能な変数を定義できるため、for-each のような挙動を実現できるようになります。

matrix を用いて、作成したい Issue を記述した Markdown ファイルのファイル名を列挙した変数 `filename` を用意します。この変数には `${{ matrix.filename }}` としてアクセスします。

実装はこのようになります。

```yaml:.github/workflows/create-issue.yaml
name: create template issue
on:
  - push
permissions:
  contents: read
  issues: write 
jobs:
  create-template-issue:
    strategy:
      matrix:
        filename: ["issue-1", "issue-2"]
    runs-on: ubuntu-latest
    steps:
      - name: Check out the repo
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - name: Get parent hash
        run: echo "PARENT_HASH=$(git rev-parse HEAD^@)" >> $GITHUB_ENV
      - name: Create template issue
        if: env.PARENT_HASH == ''
        uses: JasonEtco/create-an-issue@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          filename: .github/issues/${{ matrix.filename }}.md
```

:::details Diff
```diff yaml:.github/workflows/create-issue.yaml
 name: create template issue
 on:
   - push
 permissions:
   contents: read
   issues: write 
 jobs:
   create-template-issue:
+    strategy:
+      matrix:
+        filename: ["issue-1", "issue-2"]
     runs-on: ubuntu-latest
     steps:
       - name: Check out the repo
         uses: actions/checkout@v3
         with:
           fetch-depth: 0
       - name: Get parent hash
         run: echo "PARENT_HASH=$(git rev-parse HEAD^@)" >> $GITHUB_ENV
       - name: Create template issue
         if: env.PARENT_HASH == ''
         uses: JasonEtco/create-an-issue@v2
         env:
           GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
         with:
-          filename: .github/issues/license.md
+          filename: .github/issues/${{ matrix.filename }}.md
```
:::

これで、リポジトリの作成時に自動で好きな内容の Issue を好きなだけ立てられるようになりました！

# おわりに

2023年は従量課金に追われない穏やかな生活を送りたいものですが、果たして。ここまでご覧いただきありがとうございました。今年もたくさん CI 回していきましょう。

[^1]: [テンプレートからリポジトリを作成する | GitHub Docs](https://docs.github.com/ja/repositories/creating-and-managing-repositories/creating-a-repository-from-a-template)
[^2]: GitHub には [Issue テンプレート](https://docs.github.com/ja/communities/using-templates-to-encourage-useful-issues-and-pull-requests/configuring-issue-templates-for-your-repository) と呼ばれる機能があります。これは、新しく Issue を作成する際に予め内容を入力しておくことができるもので、今回実現したい機能そのものが提供されているわけではありません。
[^3]: [ワークフローをトリガーするイベント | GitHub Docs](https://docs.github.com/ja/actions/using-workflows/events-that-trigger-workflows)
[^4]: [Usage - actions/checkout | GitHub](https://github.com/actions/checkout#usage)
[^5]: [Other <rev>^ Parent Shorthand Notations | Git - git-rev-parse Documentation](https://git-scm.com/docs/git-rev-parse#_other_rev_parent_shorthand_notations)
[^6]: [環境変数の設定 - GitHub Actions のワークフロー コマンド | GitHub Docs](https://docs.github.com/ja/actions/using-workflows/workflow-commands-for-github-actions#setting-an-environment-variable)
[^7]: [jobs.<job_id>.steps[*].if - GitHub Actions のワークフロー構文 | GitHub Docs](https://docs.github.com/ja/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsif)
[^8]: [式 | GitHub Docs](https://docs.github.com/ja/actions/learn-github-actions/expressions)
[^9]: [Custom templates - Create an issue | GitHub Marketplace](https://github.com/marketplace/actions/create-an-issue#custom-templates)
