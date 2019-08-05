---
layout: post
title: Windows 한영키 Shift + Space로 변경하는 방법
date: 2019-08-05 23:35:00
tags: windows
image:
comments: true
categories: windows
tag: windows
---

1. 레지스트리 편집기 실행 (윈도우 파일 검색창 > regedit 검색하여 실행)
2. \HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\i8042prt\Parameters 선택
3. "LayerDriver KOR"의 값을 "kbd101c.dll"로 변경
4. "OverrideKeyboardSubtype"의 값을 "5"로 변경
5. 재부팅

### 옵션 설명

|      키보드 종류     | LayerDriver KOR | Override Keyboard Subtype | Override Keyboard Identifier | 한영키        | 한자키       |
|:--------------------:|-----------------|:-------------------------:|:----------------------------:|---------------|--------------|
| PC/AT 101키 (종류 1) | kbd101a.dll     | 3                         | PCAT_101AKEY                 | 오른쪽 Alt    | 오른쪽 Ctrl  |
| PC/AT 101키 (종류 2) | kbd101b.dll     | 4                         | PCAT_101BKEY                 | 오른쪽 Ctrl   | 오른쪽 Alt   |
| PC/AT 101키 (종류 3) | kbd101c.dll     | 5                         | PCAT_101CKEY                 | Sheft + Space | Ctrl + Space |
| 한국어 103/106 키    | kbd101.dll      | 6                         | PCAT_103KEY                  | 한영 키       | 한영 키      |
