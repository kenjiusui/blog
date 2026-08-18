---
date: 2023-04-09T00:39:49+09:00
draft: false
title: "ゼロからM2 MacにPython環境構築"
tags: ["mac", "python", "M2"]
---

プライベートで使っているPCをApple Silicon M2チップを搭載したMac mini 2023に変えたのでPythonの環境をゼロから構築しました。
macOSのバージョンはVentura 13.2.1です
特に大したことはしていないですが備忘録として。

# 環境構築
## Homebrew
まずはHomebrewから入れます。とにかくこれを入れないと何も進まないので。
公式のページにインストールするときのコマンドが書かれているのでこれをそのまま実行します。
インストール自体はこれだけで完了ですが、たぶんインストールの一番最後に「PATHを通せ」みたいなメッセージが表示されているのでそれも実行してください。

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## pyenv
Pythonのバージョン管理は定番のpyenvです。
これはbrewと使えば簡単にインストールできます。

```
brew install pyenv
```

これだけだとPythonの向き先がシステムのPythonになったままになってしまうのでPATHをとおします。
コマンドは公式のreadmeに書いてあるのでそれをそのまま実行しましょう。
https://github.com/pyenv/pyenv#set-up-your-shell-environment-for-pyenv

```
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
```

## 最新のPython
ほしいバージョンのPythonをpyenvをとおしてインストールします。
今回は何も考えずに一番新しいやつを入れます。

一点だけ注意ですが、pyenvを経由してPythonをインストールするときxzをインストールしないと `ModuleNotFoundError: No module named '_lzma'` って怒られるので先んじて対応しておきます。
（先に `pyenv install` しちゃった場合はxzを入れただけだと上手く動作しないの一度pyenvのPythonをアンインストールして `pyenv uninstall 3.x.x` そのあと再度インストール `pyenv install 3.x.x` してください）

```
brew install xz
```

あとは最新のpythonのバージョンを確認して

```
pyenv install --list
```

最新のpythonをインストール

```
pyenv install 3.x.x
```

pythonのバージョンをインストールしたものにする

```
pyenv global 3.x.x
```
## poetry
パッケージ管理はpoetryが好きなのでpoetry入れておく。
pipとかbrewでも入れられるけどとりあえず公式推奨の入れ方でやる。
たぶん、これもPATHを通せと言われるのでやってください。
https://python-poetry.org/docs/#installing-with-the-official-installer

```
curl -sSL https://install.python-poetry.org | python3 -
```

以上で最低限のPython環境が構築完了です。
お疲れ様でした 🙌

# おわり
この記事は本当に最低限のものしか入れてないのであとはlinterいれたり好きにしてください。
それにしてもM1がでた頃は環境構築が非常に大変で難儀したものですがいつの間にかスマートにできるようになっていました。
本当に開発者の方々に感謝です。


# Ref.
Homebrew
https://brew.sh/

pyenv
https://github.com/pyenv/pyenv

poetry
https://github.com/python-poetry/poetry

lzmaが入っていないと怒られる問題
https://github.com/pandas-dev/pandas/issues/27532