---
title: "UbuntuとLaravelでできるブルートフォースアタック対策"
emoji: "🛡️"
type: "tech"
topics: ["ubuntu", "laravel", "php", "security", "ssh"]
published: false
published_at: "2023-04-04 07:35"
---

> 初出: 2023年4月4日(旧ブログの記事を移行・加筆)

ブルートフォースアタックへの対策を、サーバー側(Ubuntu)とアプリケーション側(Laravel)の両方からまとめました。
自分でサーバーを立ててLaravelアプリを公開している方に向けた内容です。

## 結論

* サーバー側は、パスワードでのSSHログインを無効にして、**鍵認証のみ**にするのがベストです。
* アプリ側は、Laravelのレート制限で、ログイン試行回数と停止時間を設定します。

## ブルートフォースアタックとは

簡単に言うと、IDとパスワードを何度も試して、ログイン認証を突破する攻撃手法です。

実施方法は単純ですが、パスワードが強力でも、時間とコンピュータリソースがあれば突破される可能性があります。
最近は、クラウドサーバーやレンタルサーバーなど、誰でも簡単にサーバーを立てられるので、特に注意が必要です。

## Ubuntuでの対策(サーバー側)

**パスワードでのログインをやめて、鍵認証でのログインだけを有効にする**のがベストな対策です。
むしろ、やらないとまずいレベルだと思います。

:::message alert
設定を変える前に、鍵認証でログインできることを必ず確認してください。
確認しないまま `PasswordAuthentication no` にすると、自分もログインできなくなります。
:::

SSHの設定ファイルを編集します。

```bash
sudo nano /etc/ssh/sshd_config
```

### パスワード認証を無効にする

パスワードによるSSH接続を許可するかどうかは、`PasswordAuthentication` で制御します。
次のように、`no` を指定します。

```
PasswordAuthentication no
```

元の記事では `#PasswordAuthentication yes` のコメントアウトと書いていましたが、**コメントアウトだけでは、デフォルト値の `yes` が有効なままです。** 明示的に `no` を指定しましょう。

### 空パスワードを禁止する

パスワードが空の場合でも接続できないよう、次を追記します。

```
PermitEmptyPasswords no
```

`PasswordAuthentication no` で制御しているので、必須ではありません。
むしろ不要かもしれませんが、念のための設定です。

### SSHサービスを再起動する

```bash
sudo systemctl restart sshd
```

## Laravelでの対策(アプリケーション側)

Laravelでは、既存のファイルを編集するだけで対策できます。
編集するのは `app/Http/Requests/Auth/LoginRequest.php` です。

### ログイン試行回数を設定する

`RateLimiter::tooManyAttempts` の第2引数が、ログイン試行回数です。

```php
public function ensureIsNotRateLimited(): void
{
    if (! RateLimiter::tooManyAttempts($this->throttleKey(), 5)) {
        return;
    }

    event(new Lockout($this));

    $seconds = RateLimiter::availableIn($this->throttleKey());

    throw ValidationException::withMessages([
        'email' => trans('auth.throttle', [
            'seconds' => $seconds,
            'minutes' => ceil($seconds / 60),
        ]),
    ]);
}
```

上のコードだと、5回失敗するとロックされます。

### ログイン停止時間を設定する

`RateLimiter::hit` の第2引数が、停止時間(秒)です。
10分にしたいなら `600` を指定します。

```php
public function authenticate(): void
{
    if (! Auth::guard($guard)->attempt(
        [$credentials_key['id'] => $email, $credentials_key['pw'] => $password],
        $remenber)
    ) {
        RateLimiter::hit($this->throttleKey(), 60);

        throw ValidationException::withMessages([
            'email' => trans('auth.failed'),
        ]);
    }
}
```

上のコードは60秒の指定です。

## まとめ

* ブルートフォースアタックは、時間とリソースがあれば強いパスワードでも突破され得ます。
* サーバー側は、`PasswordAuthentication no` で鍵認証のみにしましょう。
* アプリ側は、`LoginRequest.php` のレート制限で試行回数と停止時間を設定します。
* サーバーとアプリの両方で対策すると、より安心です。
