---
title: "iTerm2でディレクトリごとにテーマを自動で切り替える方法(Zsh)"
emoji: "🎨"
type: "tech"
topics: ["iterm2", "zsh", "terminal", "macos", "productivity"]
published: false
published_at: "2025-07-04 15:27"
---

> 初出: 2025年7月4日(旧ブログの記事を移行・加筆)

iTerm2で、ディレクトリごとに、ターミナルのテーマ(背景色、フォント、カラースキームなど)を自動で切り替える方法をまとめました。
複数のプロジェクトや副業を、並行して進めている方に向けた内容です。

## 結論

iTerm2で**プロファイルを複数作って**、`~/.zshrc` に、**ディレクトリに応じてプロファイルを切り替える関数**を書けば、自動で切り替わります。

## なぜディレクトリごとに切り替えるのか

* **視覚的な区別**: 複数のプロジェクトを並行して進めているとき、ターミナルの色が変わると、いまどの環境で作業しているかが、ひと目で分かります。
* **集中力の向上**: 作業モードに合わせた色にすることで、集中力を高める効果も期待できます。
* **誤操作の防止**: 本番環境や重要なディレクトリに入ったときに、警告色にすれば、不用意なコマンド実行を防げます。

## 必要なもの

* **iTerm2**: macOS用の高機能なターミナルエミュレータです。
* **Zsh**: macOSのデフォルトのシェルです。(Bashでも可能ですが、Zshのほうが設定が簡単です。)

## 設定手順

設定は、大きく2つのステップです。

1. iTerm2で、複数のプロファイル(テーマ)を作ります。
2. Zshの設定ファイルで、ディレクトリに応じたプロファイル切り替えのスクリプトを書きます。

### ステップ1: iTerm2でプロファイルを作る

まず、切り替えたいテーマごとに、iTerm2のプロファイルを作ります。

1. iTerm2を開いて、`Cmd + ,` で設定(Preferences)を開きます。
2. 「Profiles」タブを選びます。
3. 左下の「+」ボタンで、新しいプロファイルを作ります。
   * デフォルトの「Default」とは別に、仕事用の「**Work Theme**」、個人用の「**Personal Theme**」など、分かりやすい名前を付けましょう。
4. 各プロファイルで、テーマを設定します。
   * **Colors**: 「Color Presets」から、好きなカラースキームを選んだり、背景色を調整したりできます。
   * **Text**: フォントや文字サイズを設定します。
   * **Window**: 背景画像や透明度も設定できます。

### ステップ2: Zshの設定ファイルにスクリプトを書く

シェルがディレクトリを移動するたびに、iTerm2に「このプロファイルに切り替えて」と指示するスクリプトを書きます。
`~/.zshrc` を開いて、次のコードを追記します。

```zsh
# iTerm2のプロファイルを切り替える関数
# iTerm2独自の制御シーケンス(OSC 1337;SetProfile)で、プロファイルを変更する
function iterm_set_profile() {
  local profile_name="$1"
  echo -e "\033]1337;SetProfile=${profile_name}\a"
}

# コマンド実行前に呼ばれる関数
# 現在のディレクトリ($PWD)をチェックして、適切なプロファイルを呼び出す
function precmd_iterm_profile_switcher() {
  if [[ "$PWD" == *"/Users/yourname/projects/work"* ]]; then
    # '/Users/yourname/projects/work' 配下なら 'Work Theme' に
    iterm_set_profile "Work Theme"
  elif [[ "$PWD" == *"/Users/yourname/documents/personal"* ]]; then
    # '/Users/yourname/documents/personal' 配下なら 'Personal Theme' に
    iterm_set_profile "Personal Theme"
  else
    # それ以外のディレクトリでは、デフォルトのプロファイルに戻す
    iterm_set_profile "Default"
  fi
}

# Zshがコマンド実行前に、関数を実行するように登録
precmd_functions+=(precmd_iterm_profile_switcher)
```

:::message alert
**コードは、ご自身の環境に合わせて修正してください。**

* `"/Users/yourname/projects/work"` や `"/Users/yourname/documents/personal"` は、テーマを切り替えたい実際のディレクトリパスにします。
* `"Work Theme"`、`"Personal Theme"`、`"Default"` は、ステップ1で作ったプロファイルの名前と、正確に合わせます。
:::

### 設定の反映

`~/.zshrc` に追記したら、ターミナルを再起動するか、次のコマンドで再読み込みします。

```bash
source ~/.zshrc
```

## 動作確認

テーマを切り替えたいディレクトリに `cd` してみましょう。
たとえば、`/Users/yourname/projects/work` に移動すると、「Work Theme」が適用されて、背景色やフォントが切り替わります。
別のディレクトリに移動すると、「Default」に戻ります。

## ちょっとしたヒント

* `if` 文の条件を増やせば、さらに多くのディレクトリやプロジェクトに、異なるテーマを設定できます。
* `*"..."*` は、パスのどこかに含まれていればマッチする、という意味です。厳密に特定のディレクトリだけにしたい場合は、`"$PWD" == "/Users/yourname/exact/path"` のように書きます。

## まとめ

* iTerm2のプロファイルを複数作って、`~/.zshrc` から切り替えます。
* ディレクトリごとに色が変わると、作業環境の取り違えを防げます。
* 本番環境のディレクトリだけ警告色にすると、誤操作の防止にもなります。
