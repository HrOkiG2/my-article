---
title: "Obsidianを導入してGitで同期する(タスク管理・メモ用)"
emoji: "🗒️"
type: "tech"
topics: ["obsidian", "git", "markdown", "productivity"]
published: false
published_at: "2025-05-05 12:49"
---

> 初出: 2025年5月5日(旧ブログの記事を移行・加筆)

流行りのObsidianを、タスク管理とメモに導入した手順をまとめました。
Notionの代わりになるメモツールを探している方や、Gitでメモを管理したい方に向けた内容です。

## 結論

Obsidianは、**オフラインで使えて、Markdownで書けて、プラグインが豊富**なメモツールです。
`obsidian-git` プラグインで、Gitによる自動のcommit・push・pullもできます。

## 導入のきっかけ

Google Workspaceを契約したものの、タスク管理に悩んでいたときに、Obsidianを見つけました。

* オフラインで使える
* Notionの代わりになる
* Markdownで書ける
* ユーザープラグインが多数ある

調べてみると、面白そうなツールだったので、タスク管理に導入してみることにしました。

## 導入

### 環境

* macOS(M1)
* メモリ16GB
* Sonoma 14.x

### インストール

```bash
brew install --cask obsidian
```

### 保存場所(Vault)を作成する

**Vault**と呼ばれる、データの保管庫を作ります。
管理用のディレクトリを作って、そこにデータを保管していく形です。Gitに似ています。
画面の案内どおりに操作すれば、簡単にできました。

### UIを変えたい

UIが味気ないので、プラグインやテーマを導入しました。
こうしたカスタマイズは、沼にはまると抜け出せなくなるので、次のことを大事にしました。(tmuxのときに沼にはまって、作業どころではなくなった反省を活かしました。)

* とにかく、設定は少ないほうがよい
* 目的はメモ帳なので、文字が見やすいことが大事
* 多少、気分が上がると嬉しい

参考にしたサイト: [10 Recommended Dark Themes(Obsidian Tips)](https://obsidiantips.com/10-recommended-dark-themes)

#### コミュニティプラグインの有効化

1. `Command + ,` で、設定を開きます。
2. 「Community plugins」を有効にします。

### Gitで管理する

Git管理が、とても楽になりました。

1. GitHubで、リポジトリを作ります。
2. コミュニティプラグインから、「Git」をインストールします。
3. ターミナルで、Vaultのディレクトリを、Git管理できるようにします。

さらに、pushやpullを、**自動で実行する**設定もできます。

* **Split automatic commit and push**: commitとpushを、別々のタイマーで行うかどうかです。
* **Auto pull interval (minutes)**: 自動でpullする間隔(分)です。`0`(デフォルト)は無効なので、注意してください。
* **Auto push interval (minutes)**: 自動でpushする間隔です。

### とりあえず入れたプラグイン

* [obsidian-calendar-plugin](https://github.com/liamcain/obsidian-calendar-plugin)
* [obsidian-git](https://github.com/Vinzent03/obsidian-git)
* [obsidian-mind-map](https://github.com/lynchjames/obsidian-mind-map)

## これはメモ帳だぞ

こんなに多機能だとは、想定外でした。できることが多すぎて、設定するのが、嫌になってきました。
カレンダーも表示できて、メモ管理もある程度設定できたので、ここで一旦、よしとします。

## まとめ

* Obsidianは、オフラインでMarkdownを書ける、プラグイン豊富なメモツールです。
* Vaultを作って、`obsidian-git` で、Gitによる同期ができます。
* カスタマイズの沼に、はまりすぎないように注意しましょう。
