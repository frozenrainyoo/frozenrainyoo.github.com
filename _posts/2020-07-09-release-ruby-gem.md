---
layout: post
title: Ruby Gem 배포하기
date: 2020-07-09 12:47:30
image:
comments: true
categories: macOS iOS
tag:
---

xcodebuild 로그를 줄이기 위해 formatter를 사용하게 되었는데 우리 입맛에 바꾸기 위해 custom formatter를 구현하고 RubyGems에 배포하는 것 까지 기록을 남긴다.  
https://github.com/frozenrainyoo/xcpretty-custom-print-formatter  

### 구조
├── LICENSE  
├── README.md  
├── Rakefile  
├── bin  
│   └── xcpretty-custom-print-formatter  
├── lib  
│   └── custom_print_formatter.rb  
└── xcpretty-custom-print-formatter.gemspec  
  
bin: 실행파일  
lib: gem을 위한 ruby코드  
Rakefile: 보통 테스트 자동화, 코드 생성, 다른 작업을 수행하는데 사용  
README.md: 문서  
LICENSE: 라이선스  
xcpretty-custom-print-formatter.gemspec: gem에 관한 정보를 가지고 있는 gemspec.  
  
http://ruby-korea.github.io/rubygems-guides/specification-reference/ 를 참고하여 작성.  
xcpretty-custom-print-formatter의 경우 xcpretty open source를 참고하여 작성하였다.  

### 빌드하기
```bash
$ gem build xcpretty-custom-print-formatter.gemspec
  Successfully built RubyGem
  Name: xcpretty-custom-print-formatter
  Version: 1.0.0
  File: xcpretty-custom-print-formatter-1.0.0.gem
```

### Key 발급받기
먼저 https://rubygems.org/에 접속하여 가입을 합니다.  
아래 코드 frozenrainyoo를 가입한 user id로 치환하여 실행하면 인증서를 발급받을 수 있습니다.  
```bash
$ curl -u frozenrainyoo https://rubygems.org/api/v1/api_key.yaml >
~/.gem/credentials; chmod 0600 ~/.gem/credentials

Enter host password for user 'frozenrainyoo':
```

### 배포하기
```bash
$ gem push xcpretty-custom-print-formatter-1.0.0.gem
Enter your RubyGems.org credentials.
Pushing gem to RubyGems.org...
Successfully registered gem: xcpretty-custom-print-formatter (1.0.0)
```

### 설치하기
```bash
$ gem install xcpretty-custom-print-formatter # rubygems.org에 배포된 gem 설치
$ gem install ./xcpretty-custom-print-formatter-1.0.0.gem # 로컬에 있는 gem 설치
```

### References
http://ruby-korea.github.io/rubygems-guides
