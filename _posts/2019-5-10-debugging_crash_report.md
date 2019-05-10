---
layout: post
title: Debugging Crash Report
date: 2019-05-10 14:56:28
tags:
image:
comments: true
categories: macOS
tag: debug macOS crash_report
---

macOS에서 Crash report를 디버깅하는 방법을 간단히 공유해본다.

아래와 같은 Crash report를 받았다.

```
  0   com.example.app        0x000000010b76165c 0x10a6a5000 + 17548892 
  1   com.example.app        0x000000010b75a954 0x10a6a5000 + 17520980 
  2   com.example.app        0x000000010a7a5dea 0x10a6a5000 + 1052138 
  3   com.example.app        0x000000010a77d962 0x10a6a5000 + 887138 
  4   com.example.app        0x000000010b4bed2c 0x10a6a5000 + 14785836 
  5   com.example.app        0x000000010b849a0e 0x10a6a5000 + 18500110 
  6   com.example.app        0x000000010a6c31ff 0x10a6a5000 + 123391
```

1. 먼저 아카이브 안에 dSYM 바이너리 파일 위치로 이동한다.  
  (빌드 발행할 때 Symbol을 포함한 아카이브 빌드하도록 하여 해당 버전의 아카이브를 이용하여 디버깅을 진행한다.)
  ```
   $ cd app.xcarchive/dSYMs/exampleApp.app.dSYM/Contents/Resources/DWARF
  ```
2. atos 명령어를 사용하여 CallStack을 확인한다. 자세한 내용은 매뉴얼을 확인하도록 한다. ($ man atos)
  ```
   $ atos -o exampleApp -arch x86_64 -l 0x10a6a5000 0x000000010b76165c 0x000000010b75a954 0x000000010a7a5dea 0x000000010a77d962 0x000000010b4bed2c 0x000000010b849a0e 0x000000010a6c31ff
  ```
