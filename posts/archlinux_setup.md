---
title: "私の Arch Linux 初期設定"
date: "2026-03-28"
categories: [tech]
description: "Arch Linux をインストールするたびにする初期設定の備忘録です。"
---

WSL2 やデュアルブート、フルインストールなど、ことあるごとに Arch Linux を入れ直しています。
最初にやる設定の、本当に自分のための備忘録です。

## デュアルブート・インストール関連
ラップトップに Windows と Arch Linux をデュアルブートで入れた。

### パーティション構成
最終的なパーティション構成はこんな感じになる。

| パーティション | サイズ | 用途 |
|---------------|--------|------|
| `nvme0n1p1` | 200MB | EFI (Windows/Arch 共有) |
| `nvme0n1p2` | 16MB | Microsoft Reserved |
| `nvme0n1p3` | 179.2GB | Windows 11 |
| `nvme0n1p4` | 822MB | Windows 回復 |
| `nvme0n1p5` | 296.7GB | Arch Linux (ext4) |


詳細な Arch Linux のインストールは、色々な記事があるのでそちらを参照する。
または　AI にでも聞けばできる。

## CapsLock を Ctrl にリマップ
以下を実行する。
```zsh
localectl set-x11-keymap us "" "" caps:ctrl_modifier
```

設定とかでもできるけど、システムを直接書き換えるほうが確実な気がしてる。

## 日本語入力 (fcitx5 + Mozc)
```bash
sudo pacman -S fcitx5 fcitx5-mozc fcitx5-configtool fcitx5-qt fcitx5-gtk
```

環境変数を設定する。`~/.config/environment.d/fcitx5.conf` を作成：
```zsh
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```
- ログアウト・ログイン後、fcitx5 を起動して設定ツールを開く。
- 入力メソッドは「英語 (US)」と「Mozc」だけにする。
- 「キーボード - 日本語」が入っていると JIS 配列になるので削除する。
- 自動起動は KDE の「自動起動」設定から `fcitx5` を追加する。


## yay
AUR ヘルパーというやつ。
一時期、レポジトリが落ちてたので、バイナリをクローンしてくる。

```bash
cd /tmp
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si
```


## LaTeX
フルパッケージでインストールする。
```bash
sudo pacman -S texlive-meta biber
```

## Typst
```bash
sudo pacman -S typst
```

## VS Code
```zsh
yay -S visual-studio-code-bin 
```

## Dropbox
```zsh
dropbox
```

## SSH (Githubのために使う)
```zsh
mkdir ~/.ssh
cd ~/.ssh
ssh-keygen -t ed25519
cat ~/.ssh/id_ed25519.pub
# → GitHub に公開鍵を登録する
ssh -T git@github.com # 接続確認
```

## Git 設定

```bash
git config --global user.name "名前"
git config --global user.email "メール"
git config --global core.autocrlf input
git config --global init.defaultBranch main

# デュアルブート改行コード対策 (要るかわからないけどやった)
git config --global core.attributesfile ~/.gitattributes_global
cat > ~/.gitattributes_global << 'EOF'
* text=auto eol=lf
*.png binary
*.jpg binary
*.pdf binary
EOF
```

## R/RStudio
```zsh
sudo pacman -S r
yay -S rstudio-server-bin
```

サーバーを開始するときは、
```zsh
sudo rstudio-server start
```
をしたうえで、`http://localhost:8787/` にアクセスする。

毎回やるのが面倒なら、 systemd サービスにしちゃえばいい。`/etc/rstudio/rserver.conf` に

```txt
server-user=<username> # <username> はユーザー名
```

を追記したうえで、

```zsh
sudo systemctl enable --now rstudio-server
```

サーバーを開始してもログインできない場合は、 `/etc/pam.d/rstudio` の内容を次のように修正する
(修正前)
```zsh
#%PAM-1.0

auth       required     pam_securetty.so
auth       requisite    pam_nologin.so
auth       include      system-local-login
account    include      system-local-login
session    include      system-local-login
```

(修正後)
```zsh
#%PAM-1.0
auth       required     pam_unix.so
account    required     pam_unix.so
session    required     pam_unix.so
```

### パッケージ関連の注意点 (随時追加)
- 依存関係のエラーが出るときは、 `install.packages()` の引数に `dependencies = TRUE` を入れるとよい。
- `tidyverse` をインストールしようとすると手こずることが多い。さきに `stringi` パッケージをインストールするのがよい。
- `V8` 関連のエラーも多い。コンピュータに `imagemagick` をインストールするのと、静的ビルドを有効にする。
    - `sudo pacman -S imagemagick`
    - Rコンソールで `Sys.setenv(DOWNLOAD_STATIC_LIBV8 = 1)` を実行してから `install.packages("V8")`

## Python
```zsh
sudo pacman -S python python-pip python-pipx
```

一般環境という使い方は最近しないらしい。
原則として、どのプロジェクトでも仮想環境を使う。
ただし、システム全体に直接パッケージをインストールするときは、`pipx` でやる。