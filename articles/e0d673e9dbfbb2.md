---
title: "GitHub Fine-grained Personal Access Token を使う"
topics: ["GitHub", "Token", "Ubuntu"]
type: "tech"
emoji: "🐚"
published: true
---
# GitHub Fine-grained Personal Access Token を使う

ゼロからの設定だと意外に手順が多い。

---

# 🌒️ 序

最近、 Ubuntu 24.04 LTS を 22.04 LTS にダウングレードした。Android Studio が libcurses5 を必要としているからだ。で、 git 使おうかと思ったら、そもそも git コマンドすら入ってない状態（ミニマムでクリーンインストールしたので）。そこから git が普通に使えるようになるまでの道のりをメモしておく。次回、また調べ回るのは嫌だし。

# 🌕️ 破

## git 導入

```bash
sudo apt install git-all
```

```bash
$ git --version
git version 2.34.1
```

## パスワードとか使ってんじゃねぇ！と怒られる

```bash
git clone https://github.com/USER/REPO.git
```

> remote: Support for password authentication was removed on August 13, 2021.
> remote: Please see https://docs.github.com/get-started/getting-started-with-git/about-remote-repositories#cloning-with-https-urls for information on currently recommended modes of authentication.

そうか。 git clone  する前に、トークンとか配置しないといけないやつか。

## Fine-grained Personal Access Token を作る

[Managing your personal access tokens - GitHub Docs](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token)

1. アカウントの Settings に入る。
2. かなり下の方までスクロールすると、 ＜＞ Developer settings の項目があるので入る。
3. Personal access tokens を開くと、 Fine-grained tokens (Preview) というのがあるので、それを選ぶ。
4. Generate new token
5. 諸々、入力する
    - 有効日数がデフォルト30日、選択肢は60日, 90日, カスタム, 無期限。
        - 月1回程度更新しなさいってことらしい。派遣エンジニアに契約期間だけ有効なパスを与える、みたいな感覚だな。契約更新したらトークンも更新するということか。1つのトークンを1人に1つずつ割り当てて使うことを前提にしている。
    - 許可するレポジトリを絞り込める。
        - public (read only), all, 個別に指定
    - 実際に与える権限を指定する
        - デフォルトでは権限なし
        - Repository permissions と Account permissions に分類されている。
        - それぞれを展開して項目を選ぶ。
        - Repository permissions の Contents に read/write を与えれば、通常の操作には十分。
            - Metadata read only は、必須なので勝手についてくる。
        - commit しない人には read only 権限だけで良い。
6. Generate token
7. 作られた token をコピーして保存する。
    - token 文字列そのものは一度しか表示されない。

権限は、あとから Edit で追加・削除も可能なので、できるだけ絞り込んで作るのが良い。認証確認だけなら権限を与えなくても可能。

[Permissions required for fine-grained personal access tokens - GitHub Docs](https://docs.github.com/rest/overview/permissions-required-for-fine-grained-personal-access-tokens)

## 認証ストアに登録する

.gitconfig とかに平文で直接書く方法は推奨されていない。

https://github.com/git-ecosystem/git-credential-manager/blob/release/docs/credstores.md#plaintext-files

Windows とか Mac だとOS標準の認証ストアが使えるが、 Ubuntu Linux だと自前で構築しないといけない。

[Ubuntu 22.04 LTS でGit Credential Managerを使う - 発声練習](https://next49.hatenadiary.jp/entry/20231128/1701175320)

だいたい上記の説明のとおりだ。Git 公式の説明だと、次の部分だが、複数ページに飛んでいかないといけないので、上記のまとめが助かる。

- https://github.com/git-ecosystem/git-credential-manager
- https://github.com/git-ecosystem/git-credential-manager/blob/release/docs/install.md

### ポイント

1. Ubuntu のリポジトリには無いので、Debian のパッケージを使う
2. 22.04 LTS が古い = git バージョン 2.34.1 と古いのは撹乱要因

### 前提条件のコマンドを入れる

```bash
sudo apt install gpg
sudo apt install pass
```

### gpg キーを作る

- パスフレーズを用意しておく。
- 8文字以上で、アルファベット以外を最低1文字選べ、などとおっしゃる。

```bash
$ gpg --gen-key
gpg (GnuPG) 2.2.27; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Note: Use "gpg --full-generate-key" for a full featured key generation dialog.

GnuPG needs to construct a user ID to identify your key.

Real name: MyName
Email address: myname@mydomain.com
You selected this USER-ID:
    "MyName <myname@mydomain.com>"

Change (N)ame, (E)mail, or (O)kay/(Q)uit? o
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key 1234567890123456 marked as ultimately trusted
gpg: directory '/home/User/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/home/USER/.gnupg/openpgp-revocs.d/F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0.rev'
public and secret key created and signed.

pub   rsa3072 2025-02-21 [SC] [expires: 2027-02-21]
      F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0
uid                      MyName <myname@mydomain.com>
sub   rsa3072 2025-02-21 [E] [expires: 2027-02-21]
```

ここで指定したパスフレーズは、初回認証時に求められる。

### 保管庫を設定する

```bash
$ git config --global credential.credentialStore gpg
$ pass init USER
mkdir: created directory '/home/USER/.password-store/'
Password store initialized for USER
```

これで必要な .gitconfig が作られるはずだったのだが、どうやらそこができておらず、後々、面倒になった。git と credential manager の世代不一致が原因かも。

git config するよりも、 ~/.gitconfig の完成形を理解して直接作った方が確実。

### git-credential-manager パッケージを導入する

- https://github.com/git-ecosystem/git-credential-manager
- https://github.com/git-ecosystem/git-credential-manager/releases

上記から最新リリースの deb パッケージを探す。

- これが最新
- https://github.com/git-ecosystem/git-credential-manager/releases/download/v2.6.1/gcm-linux_amd64.2.6.1.deb

```bash
wget https://github.com/git-ecosystem/git-credential-manager/releases/download/v2.6.1/gcm-linux_amd64.2.6.1.deb
sudo dpkg -i gcm-linux_amd64.2.6.1.deb
git-credential-manager configure
```

### ~/.gitconfig を下記のようにする

```bash
[user]
	name = MyName
	email = myname@mydomain.com
[credential]
	helper = manager
	credentialStore = gpg
```

設定状態の確認。レポジトリの固有設定があり、複数ファイルに分かれている場合の確認に良い。

```bash
$ git config --list --show-origin
file:/home/USER/.gitconfig      user.name=MyName
file:/home/USER/.gitconfig      user.email=myname@mydomain.com
file:/home/USER/.gitconfig      credential.helper=store
file:/home/USER/.gitconfig      credential.credentialStore=gpg
```

### 初回認証

git clone, git pull など、 remote にアクセスするコマンドを打つと認証画面が開く。
ポップアップが開かないなら、ターミナルなり端末なりを起動しなおすと良いかもしれない。
認証確認だけしたいというなら、次のログインコマンドを使っても、認証画面を出すことができる。

```bash
git credential-manager github login
```

![git_auth_browser.png](https://nyosak.github.io/article-base-doc/media/70302_github_token_git_auth_browser.png)

ブラウザまたはスマホアプリによる認証が表示される。こちらを使うとトークンを使わずに、一時的な認証をキャッシュできる。

今回はトークンを使うので、 Token のタブを選び、トークン入力画面に切り替える。

![git_auth_token.png](https://nyosak.github.io/article-base-doc/media/70302_github_token_git_auth_token.png)

先程、 GitHub の管理画面で作ったトークンを貼り付け、 Sign in を実行する。

### 確認

```bash
$ git credential-manager github list
GitUser
```

自分のアカウント名がでてきたら、認証情報の追加ができている。

### ようやく git clone

```bash
git clone https://github.com/USER/REPO.git
```

これで終わりかと思ったが。。。

### No commits yet

```bash
$ git status
On branch main

No commits yet

nothing to commit (create/copy files and use "git add" to track)
```

> No commits yet

いつもは、新しいレポジトリを作るときに README.md を自動で作らせておくのだが、今回はそれをやらなかった。なので、空っぽのレポジトリが作られた。
ところが、おそらく、 commits[commits.length - 1] が NULL でない何か（最新のコミット）を返すことを git コマンドは期待して設計されているみたいで、何も無いと各種動作が正常に働かない。たとえばブランチを作ることができない。
解決策としては README.md でいいから、とにかく add して commit しておく。これだけでいい。
最初に README.md を作ってくれるオプションがあるのは深い意味があったようだ。

[Git エラー「fatal: Not a valid object name: 'master'.」の対処法 - Qiita](https://qiita.com/takuma-jpn/items/fe205ff59cfa10746aa8)

# 🌖️ 急

## 複数のトークンを使う

レポジトリ単位でトークンを切り替える。

### レポジトリごとに認証情報を保存する

~/.gitconfig に下記を追加する

```bash
[credential "https://github.com"]
	useHttpPath = true
```

認証情報の無いレポジトリにアクセスすると、認証を求めるポップアップが開くので、新しいトークンを登録する。
これにより、次のようなファイルに認証情報が保存される。

```bash
~/.password-store/git/https/github.com/USER/REPO.git/USER.gpg
```

### GitHub アカウントごとに認証情報を保存する

```bash
useHttpPath = false
```

つまり、以前のままの ~/.gitconfig だと、下記のファイルに認証情報が保存されている。

```bash
~/.password-store/git/https/github.com/USER.gpg
```

そこで、先程作られた新しいアカウントの認証ファイルをこの階層にコピーしてみる。
しかし、これでは認証に失敗する。
認証ファイルには階層情報も入っているようだ。

そこで、既存の認証情報を一度退避させてから、追加したいアカウントの全体認証ファイルを作ってみる。

```bash
cd ~/.password-store/git/https/github.com
mv USER.gpg USER/
```

認証情報リストを出して、消えたことを確認する。

```bash
git credential-manager github list
```

何も出力されなければ、認証が消えている。
追加したいアカウントの git 操作をする。

```bash
git clone https://github.com/USER2/REPO2.git
```

ポップアップが出るので、トークンを入れる。

```bash
git credential-manager github list
```

追加したアカウントが表示される。
退避した以前の認証ファイルを戻す。

```bash
cd ~/.password-store/git/https/github.com
mv USER/USER.gpg ./
```

```bash
$ git credential-manager github list
USER2
USER
```

2つのアカウントが表示された。いける！

これで git 操作をしてみる。
アカウントを選択するポップアップが表示された。

![git_auth_switch.png](https://nyosak.github.io/article-base-doc/media/70302_github_token_git_auth_switch.png)

これで選択すれば動作するが、面倒！

https://github.com/git-ecosystem/git-credential-manager/blob/main/docs/multiple-users.md

個別レポジトリの origin を https://USER@github.com…. みたいにする解決策が載っているが、それもいちいち面倒。

別の方法が載っていた。

```bash
git config --global credential.https://github.com.username alice
```

これを今の環境に適用するとすれば、次のものを ~/.gitconfig に追加すれば良い。

```bash
[credential "https://github.com/USER"]
	username = USER

[credential "https://github.com/USER2"]
	username = USER2
```

これで、アカウント選択のポップアップが出なくなった。

そうか。退避とかややこしいことしないで、いきなりこれでいいのだった。
認証ポップアップが出るので、トークンを登録できる。

```bash
git clone https://USER3@github.com/USER3/REPO3.git
```

