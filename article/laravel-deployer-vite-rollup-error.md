---
title: "DeployerでLaravelをデプロイするとViteのビルドが失敗する問題の解消"
emoji: "🚀"
type: "tech"
topics: ["laravel", "deployer", "vite", "rollup", "php"]
published: false
published_at: "2024-08-14 06:40"
---

> 初出: 2024年8月14日(旧ブログの記事を移行・加筆)

Deployerでクラウドサーバーにデプロイしたときに起きた、Viteのビルドエラーの解消方法をまとめました。
ローカルではビルドが通るのに、サーバーで失敗して困っているLaravel開発者の方に向けた内容です。

## 結論

サーバー(Linux)用の Rollup のネイティブバイナリを、optionalDependenciesに追加すると解消しました。

```bash
npm install @rollup/rollup-linux-x64-musl --save-optional
```

## 概要

Deployerでクラウドサーバーにデプロイすると、ビルドエラーが発生しました。
ローカルのビルドは問題なく通るのに、サーバーに上げるときに失敗します。
初めて見るエラーで、解消に時間がかかったので、記事に残します。

### エラー内容

```
npm ERR! code E404
npm ERR! 404 Not Found - GET https://registry.npmjs.org/@rollup%2frollup-linux-x64-musl--save-optional - Not found
npm ERR! 404
npm ERR! 404  '@rollup/rollup-linux-x64-musl--save-optional@*' is not in this registry.
```

### 環境

* Laravel 10
* Vue 3
* TypeScript 5
* Tailwind
* DaisyUI
* MacBook Air(開発機)
* Amazon Linux 2023(本番サーバー)

## 原因

Viteのビルド中に、`@rollup/rollup-linux-x64-musl` モジュールが見つからない問題です。
ViteがRollupのネイティブバイナリに依存しているため、特定の環境(特にDockerコンテナやLinux環境)で発生することがあるようです。

* [Vite Discussions #15532](https://github.com/vitejs/vite/discussions/15532)

## 解決方法

### Rollupのモジュールをインストールする

```bash
npm install @rollup/rollup-linux-x64-musl --save-optional
```

`package.json` に、次の記載があることを確認します。

```json
"optionalDependencies": {
    "@rollup/rollup-linux-x64-musl": "^4.20.0"
}
```

### DeployerのBuildタスクを修正する

```php
task('deploy', [
    'deploy:prepare',
    'deploy:vendors',
    'deploy:shared',
    'deploy:writable',
    'build',
    'artisan:migrate',
    'deploy:publish',
]);

// Build task: run npm install and npm run build
desc('Build the project');
task('build', function () {
    run('cd {{release_path}} && npm install && npm run build');
});
```

## 番外編

次のようにBuildタスクを調整したところ、私の環境でも通りました。

```php
task('build', function () {
    run('cd {{release_path}} && rm -rf node_modules package-lock.json'); // FIXME: 原因は他にある
    run('cd {{release_path}} && npm install && npm run build');
});
```

ただし、この方法はビルドは成功しても、ブラウザでの表示に問題が出るという記事もありました。
そのため取りやめて、多くの人が解決している、Rollupの修正で対応することにしました。

Viteのバージョンを調整すれば、意外とすんなり解決するのかもしれません。
お困りの方は、試してみてください。

## まとめ

* ローカルで通るのに、サーバーのビルドだけ失敗する場合は、ネイティブバイナリの依存を疑いましょう。
* `@rollup/rollup-linux-x64-musl` を、`optionalDependencies` に追加すると解消しました。
* `node_modules` と `package-lock.json` を消してビルドする方法は、表示の問題が出るという報告もあるので、おすすめしません。
