---
title: SSHセッションをOnetime Passwordで保護する
tags: 
- SSH 
- GoogleAuthenticator
---
# クラウドサーバーを保護する
クラウド上にホストしているUbuntuマシンをSSH時にOnetimeパスワードを求めるようにしてみた
これはそのときやったことの備忘録メモ

## 環境
Ubuntu 16.04
Google Authenticatorアプリ(Authenticatorアプリなら大抵何でも良い

## Google Authenticatorの設定
インストール準備：
`sudo apt-get update && sudo apt-get upgrade`


Google authenticatorをサーバーに入れる
`sudo apt-get install libpam-google-authenticator`

Google Authenticatorアプリを実行する
`google-authenticator`

```bash
$ sudo apt-get install libpam-google-authenticator
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Get:1 http://archive.ubuntu.com/ubuntu xenial/universe amd64 libqrencode3 amd64 3.4.4-1 [23.9 kB]
Get:2 http://archive.ubuntu.com/ubuntu xenial/universe amd64 libpam-google-authenticator amd64 20130529-2 [35.3 kB]
Fetched 59.3 kB in 1s (51.6 kB/s)                 
Selecting previously unselected package libqrencode3:amd64.
(Reading database ... 97411 files and directories currently installed.)
Preparing to unpack .../libqrencode3_3.4.4-1_amd64.deb ...
Unpacking libqrencode3:amd64 (3.4.4-1) ...
Selecting previously unselected package libpam-google-authenticator.
Preparing to unpack .../libpam-google-authenticator_20130529-2_amd64.deb ...
Unpacking libpam-google-authenticator (20130529-2) ...
Processing triggers for libc-bin (2.23-0ubuntu9) ...
Processing triggers for man-db (2.7.5-1) ...
Setting up libqrencode3:amd64 (3.4.4-1) ...
Setting up libpam-google-authenticator (20130529-2) ...
Processing triggers for libc-bin (2.23-0ubuntu9) ...
```

実行するとこんな画面が出る　（この↓QRコードをスキャンすると良いことあるかも）
![ruby.png](https://qiita-image-store.s3.amazonaws.com/0/227518/ad017a07-32c6-7804-5b5a-89efadd310e9.png)

表示されているQRコードをAuthenticatorアプリでスキャンしてコードを追加する

なお、BackUPコードも出力されるので、それらは緊急時用に控えておく

あとは、表示されているプロンプトに答えていく。

```bash
Do you want me to update your "/users/rservice/.google_authenticator" file (y/n) y

Do you want to disallow multiple uses of the same authentication
token? This restricts you to one login about every 30s, but it increases
your chances to notice or even prevent man-in-the-middle attacks (y/n) y
```
同じトークンで複数セッションを貼れないようにする。

```bash
By default, tokens are good for 30 seconds and in order to compensate for
possible time-skew between the client and the server, we allow an extra
token before and after the current time. If you experience problems with poor
time synchronization, you can increase the window from its default
size of 1:30min to about 4min. Do you want to do so (y/n) n
```
デフォルトでOnetimeパスワードの有効時間は30秒だそうです。これを伸ばすことも可能だが、特に必要を感じないので、30のままで続ける

```bash
If the computer that you are logging into isn't hardened against brute-force
login attempts, you can enable rate-limiting for the authentication module.
By default, this limits attackers to no more than 3 login attempts every 30s.
Do you want to enable rate-limiting (y/n) y
```
Brute force対策を実施する。YES

## SSHDをいじる
SSHの認証設定をGoogle Authenticatorを使うよう変える
```sudo nano /etc/pam.d/sshd```
*普段はVIMを使っているのですが、最近NANOコマンドがあることを覚えたので使い心地をテスト中。

以下の行を追加する。

```
auth    required      pam_unix.so     no_warn try_first_pass
auth    required      pam_google_authenticator.so
```
上記の意味は、ログイン時、まずはパスワード認証を行うように指示
二行目はOnetime認証を求める用設定


### onetime認証を有効可する

`sudo nano /etc/ssh/sshd_config`
`ChallengeResponseAuthentication yes #デフォルトNOになっているはずなので、YESに変更`

```bash
Match User hogehogeuser #←onetimeを有効可するユーザー名を追加
    AuthenticationMethods keyboard-interactive
```

SSHを再起動
`sudo systemctl restart ssh`


これで以降のSSH開始時Onetime pwが必要になった。
やったー！

```bash
$ ssh hogehogeuser@xx.xxx.xxx.xxx
Password: 
Verification code: <= onetime pw入力欄
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-109-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

80 packages can be updated.
0 updates are security updates.


Last login: Sat Jan 13 05:37:35 2018 from 
```
