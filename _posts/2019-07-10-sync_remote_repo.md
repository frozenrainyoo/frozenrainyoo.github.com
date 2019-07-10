---
layout: post
title: Fork 한 repository 최신으로 동기화하기
date: 2019-07-10 09:00:00
tags:
image:
comments: true
categories: git
tag: git
---

```
$ git remote -v
origin  https://github.com/frozenrainyoo/settings (fetch)
origin  https://github.com/frozenrainyoo/settings (push)
```

```
$ git remote add upstream https://github.com/jglee1027/settings
```

```
$ git remote -v
origin  https://github.com/frozenrainyoo/settings (fetch)
origin  https://github.com/frozenrainyoo/settings (push)
upstream        https://github.com/jglee1027/settings (fetch)
upstream        https://github.com/jglee1027/settings (push)
```

```
$ git fetch upstream
remote: Enumerating objects: 189, done.
remote: Counting objects: 100% (189/189), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 814 (delta 187), reused 188 (delta 187), pack-reused 625
Receiving objects: 100% (814/814), 150.02 KiB | 0 bytes/s, done.
Resolving deltas: 100% (226/226), completed with 26 local objects.
From https://github.com/jglee1027/settings
 * [new branch]      master     -> upstream/master
```

```
$ git merge upstream/master
```
