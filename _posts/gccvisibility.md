---
layout: post
title: About GCC visibility=hidden Flag
---

## -fvisibility=hidden 옵션

이전에 불필요하게 공개되었던 ELF 심볼을 대부분 숨깁니다.
visibility="default"옵션이 설정되지 않은 함수, 클래스들은 export되지 않도록 합니다.

장점:
1. DSO(Dynamic Shared Object)로드 시간이 향상된다. (바인딩 라이브러리인 Boost.Python의 경우 로드 시간이 6분에서 6초로 향상되었다.)
2. 컴파일러가 코드 최적화할 때 더 좋은 코드로 최적화된다.
3. DSO 크기가 5~20% 줄어든다.
4. 심볼 충돌이 감소된다.


### How to use

먼저 GCC 4.0 이상 컴파일러가 필요하다.

1. 외부로 공개할 구조체, 클래스, 함수 선언 앞에 \_\_attribute\_\_ ((visibility("default") 키워드를 붙여준다.
2. 컴파일 옵션에 -fvisibility=hidden 옵션을 추가한다.
   
이렇게 생성된 DSO 파일과 원래 방식으로 생성한 파일을 nm -C -D 명령으로 확인해보면 차이를 알 수 있다.

```
#ifdef _MSC_VER
  #ifdef BUILDING_DLL
    #define DLLEXPORT __declspec(dllexport)
  #else
    #define DLLEXPORT __declspec(dllimport)
  #endif
  #define DLLLOCAL
#else
  #ifdef HAVE_GCCVISIBILITYPATCH
    #define DLLEXPORT __attribute__ ((visibility("default")))
    #define DLLLOCAL __attribute__ ((visibility("hidden")))
  #else
    #define DLLEXPORT
    #define DLLLOCAL
  #endif
#endif

extern "C" DLLEXPORT void function(int a);
class DLLEXPORT SomeClass
{
   int c;
   DLLLOCAL void privateMethod();  // Only for use within this DSO
public:
   Person(int _c) : c(_c) { }
   static void foo(int a);
};
```

### reference
https://www.nedprod.com/programs/gccvisibility.html
