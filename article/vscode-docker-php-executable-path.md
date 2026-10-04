---
title: "VSCodeでDocker環境のPHPが参照できないエラーの解消方法"
emoji: "🐳"
type: "tech"
topics: ["vscode", "docker", "php", "shellscript"]
published: false
published_at: "2024-01-03 22:31"
---

> 初出: 2024年1月3日(旧ブログの記事を移行・加筆)

Docker環境で開発しているときに、VSCodeで「PHP executable not found」というエラーが出た場合の解消方法をまとめました。
DockerでPHP環境を作っていて、VSCodeを使っている方に向けた内容です。

## 結論

PHPコンテナを呼び出すシェルスクリプトを作り、VSCodeのプロジェクト設定(`.vscode/settings.json`)の `php.debug.executablePath` に指定すれば解消します。

## 症状

ある日突然、VSCodeで次のエラーが出ました。(業務ではPHPStormを使っているので、VSCodeの勝手がよく分かりませんでした。)

```
PHP executable not found. Install PHP and add it to your PATH or set the php.debug.executablePath setting
```

「PHPの実行ファイルがない」という意味です。
Dockerで環境を構築していて、ローカルにPHPがないことが原因だと想像できます。

:::message
あとで知ったのですが、Dockerで環境を構築していても、**ローカルにPHPがインストールされていて、VSCodeにパスが通っていればエラーは出ない**ようです。
今すぐ動けばいい、という場合は、ローカルにインストールしてパスを通せば動きます。
:::

## 解決方法: PHPコンテナへのパスを追加する

先輩に相談したところ、「コンテナへのパスを定義するだけで解決する」と教えてもらいました。
半信半疑でしたが、そのとおりに試してみました。

### 手順1: プロジェクト固有の設定ディレクトリを作る

VSCodeは、プロジェクト単位で設定ファイルを作れます。
プロジェクトの第一階層に、`.vscode` ディレクトリを作りましょう。

VSCodeのドキュメントはかなり充実しています。時間があるときに読んでみてください。

* [VSCode ドキュメント](https://code.visualstudio.com/docs)

### 手順2: コンテナを呼び出すシェルを作る

プロジェクト直下に、PHPが動いているコンテナを呼び出すシェルファイル(`container-php.sh`)を作ります。

```bash
#!/bin/bash
docker exec -it `コンテナ名` php "$@"
```

実行権限も付けておきましょう。

```bash
chmod +x container-php.sh
```

各部分の意味はこちらです。

| 部分 | 意味 |
| --- | --- |
| `docker exec` | 実行中のDockerコンテナ内でコマンドを実行します。 |
| `-it` | `-i`(`--interactive`)は、コンテナの標準入力を開いたままにします。`-t`(`--tty`)は、仮想ターミナルを割り当てます。 |
| `$@` | シェルスクリプトの特別な変数で、渡されたすべての引数を表します。すべての引数を `php` コマンドに渡します。 |

### 手順3: VSCodeの設定ファイルを作る

`.vscode` ディレクトリの中に、`settings.json` を作ります。
VSCode本体に設定することもできますが、1つのプロジェクトしか触らない人は少ないと思うので、プロジェクト固有の設定として書くのがおすすめです。

```json
{
    // 手順2で作成したシェルファイルを指定
    "php.debug.executablePath": "./container-php.sh"
}
```

ファイル名は **`settings.json`** にしてください。
`setting.json`(sがない)だと、VSCodeに認識されません。

## まとめ

* Dockerだけで環境を作ると、VSCodeからPHPの実行ファイルが見つからないことがあります。
* コンテナ内のPHPを呼び出すシェルを作って、`php.debug.executablePath` に指定すれば解消します。
* 設定ファイル名は `settings.json` です。
* 先輩への感謝は、コーヒーで伝えました。
