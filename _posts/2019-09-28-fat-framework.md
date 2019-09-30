---
layout: post
title: Fat(Universal) Framework(Library)
date: 2019-09-28 23:35:00
image:
comments: true
categories: iOS
tag: macOS, iOS
---

## Fat(Universal) Framework(Library)

빌드 속도 문제로 애플리케이션 빌드할 때마다 오픈소스들을 빌드하지 않고 Fat Framework를 만들어  prebuilt된 라이브러리를 사용하는 경우가 많다.

Universal(multi-architecture) Library를 만들 때 자주 사용하는 lipo 명령어 사용법은 아래와 같다.

iOS의 경우 시뮬레이터 아키텍쳐인 i386, x86_64가 포함되면 appstore에 upload되지 않아 strip되어야 하는데 strip시 아래 lipo -remove가 사용된다.

라이브러리의 정보 출력
```bash
$ lipo -info libicudata.a
 Architectures in the fat file: libicudata.a are: i386 armv7s armv7 x86_64 arm64
```

i386 아키텍처 제거
```bash
$ lipo -remove i386 -output libicudataoutput.a libicudata.a
$ lipo -info libicudataoutput.a
 Architectures in the fat file: libicudataoutput.a are: armv7 armv7s x86_64 arm64
```

universal library 생성
```bash
$ lipo -create -output libicudataoutput.a libicudata-i386.a libicudata-x86_64.a libicudata-armv7.a libicudata-armv7s.a libicudata-arm64.a
$ lipo -info libicudataoutput.a
 Architectures in the fat file: libicudata.a are: i386 armv7s armv7 x86_64 arm64
```