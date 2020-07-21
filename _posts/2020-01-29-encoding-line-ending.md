---
layout: post
title: 멀티 플랫폼에서 소스 Encoding, Line Ending 오류 해결법
date: 2020-01-29 23:35:00
image:
comments: true
categories: cross-platform, multi-platform
tag:
---

# 멀티 플랫폼(크로스 플랫폼)에서 소스 Encoding, Line Ending 오류 해결법

여러 플랫폼에서 작성하는 저장소를 사용하다보면 IDE 설정에 따라 자동으로 소스의 Encoding,
Line Ending, BOM(Byte Order Mark)이 변경되는 문제가 있다.
한글이 깨지는 문제도 발생할 수 있고, 이럴 경우 자신이 수정하지 않은 코드도 수정된 것 처럼
나타나기 때문에 커밋 히스토리를 엉망으로 만들어 나중에는 머지할 때마다 문제가 된다.
또한 BOM이 제거될 경우에는 Windows VS Warning Level에 따라 빌드오류까지 야기시킨다.
이를 해결하기 위해 아래 해결법을 제시해본다.

## 첫 번째로, 이를 해결하기 저장소의 컨벤션을 맞추는 일이다.

프로그래머끼리 convention을 정하여 Encoding, Line Ending을 동일하게 맞추어 개발한다.
* Encoding: UTF-8 BOM
* Line Endings: CRLF

CR(Carriage Return, \r): 현재 위치에서 바로 아래로 이동.  
LF(Line Feed, \n): 커서의 위치를 앞으로 이동.  
  
윈도우/DOS: CRLF 조합으로 Line Ending 정의  
Unix계열(Linux, Darwin): LF로 Line Ending 정의  

이래서 Windows에서 작성한 코드를 Unix계열에서 vi로 편집했을 때 ^M이 붙는 것을 볼 수 있다.  

## 두 번째로, Line Ending의 경우 git 설정을 이용하는 방법이 있다.

#### Windows
core.autocrlf를 true로 설정하면 Checkin할 때 LF를 CRLF로 바꿔준다.

```bash
$ git config --global core.autocrlf true
```

#### Linux/macOS
core.autocrlf를 input으로 설정하면 CRLF를 LF로 바꿔서 Commit해준다.

```bash
$ git config --global core.autocrlf input
```

#### .gitattributes 파일을 이용하는 방법
저장소의 root에 .gitattributes파일을 만들어 설정하면 git config를 설정하지 않아도 된다. git config로 설정하는 방법보다는 .gitattributes 파일을 이용하는 방법을 추천한다.

```bash
* text=auto
```

```bash
# 아래와 같이 명시적으로 설정할 수도 있다.
* text eol=crlf
* text eol=lf
```

```bash
# 아래 명령어로 현재 저장소에 있는 모든 파일의 라인엔딩을 확인할 수 있다.
$ git ls-files --eol
```

## 세 번째로, git hook 기능을 이용하는 방법이 있다.
Git hook은 클라이언트 훅과 서버 훅이 있으며 그 중에 이번에 사용할 기능은 클라이언트 훅의 커밋 훅을 사용할 것이다. 커밋 훅은 4가지가 있는데 pre-commit을 사용할 것이다. pre-commit의 경우 커밋할 때 가장 먼저 호출되는 훅으로 커밋 메시지 작성하기 전에 호출된다.  
hook 기능의 자세한 내용은 추 후에 정리하도록 한다.  

사용방법은 저장소의 root에 .git/hooks/pre-commit 스크립트를 만들어 commit할 때 encoding, line-endings를 체크하도록 하여 commit할 때 잘못된 encoding일 경우 0이 아닌 값으로 리턴시키면 커밋이 취소된다. 기본적으로 .git/hook/pre-commit.sample을 제공하고 있으므로 참고할 수 있다.  
이를 응용하면 커밋하기전에 lint같은 프로그램으로 코드 스타일을 검사하거나, 특정 파일일 경우 압축할 수 있다.

```bash
# pre-commit.sample 기반으로 pre-commit을 작성한다.
$ cd .git/hooks
$ mv pre-commit.sample pre-commit
$ vi pre-commit
```

```bash
# pre-commit을 생략하는 방법으로 --no-verify 옵션을 사용하면 된다.
$ git commit --no-verify
```

## IDE에서 encoding line-ending 변경을 지원하지 않을 경우 vi, vim으로 수정하는 방법을 정리한다.

BOM 입력
```vim
:set bomb
```

BOM 제거
```vim
:set nobomb
```

^M 표시
```vim
:set ffs=unix
:e filepath
```

^M 입력
```
ctrl + v + m
```

^M 모두 제거
```vim
:%s/^M//g
```

## 서버와 로컬의 라인엔딩이 다를 경우 업데이트하는 방법
중간에 autocrlf 설정을 변경하여 서버와 로컬의 라인엔딩이 꼬이는 경우가 발생할 수 있다.  
각각의 파일의 lf와 crlf가 섞이는 경우가 발생한다.  
이 때 아래 커맨드로 서버의 라인엔딩을 받아오도록 한다.  
```bash
git config core.autocrlf false
git add --renormalize [dir|file]
```

### References
https://www.lesstif.com/pages/viewpage.action?pageId=20776404  
http://meonggae.blogspot.com/2017/03/git-git-hooks.html  
