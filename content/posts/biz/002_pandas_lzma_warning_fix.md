---
date: 2021-08-13T00:37:47+09:00
draft: false
title: "lzmaが入っていないって怒られるwarningを解消する"
tags: ["Python", "pyenv", "pandas"]
categories: ["tech"]
---

pandasを使っていたら下記のwarningがでていたのを解決したので備忘録。

```
UserWarning: Could not import the lzma module. Your installed Python is incomplete. Attempting to use lzma compression will result in a RuntimeError.
```

# 環境
- OS: macOS Catalina 10.15.7
- Python: 3.8.1
- pandas: 1.3.1
- pyenv: 2.0.1

# 解決方法
どうやらpyenvを通してPythonをインストールしていると発生するようです。

https://github.com/pandas-dev/pandas/issues/27532

対処法は `xz` をインストールしてから改めてPythonをインストールします。  
xzはbrewにあるので簡単です。

```
brew install xz
```

これでxzが入るのでPythonをインストールしなおせばOK。

```
pyenv uninstall 3.8.1
pyenv install 3.8.1
```

これでwarningが消えました 🙌