---
layout: post
title: calling convention
date: 2020-01-18 23:35:00
image:
comments: true
categories: c++
tag:
---

## Calling Convention

함수를 호출할 때 파라미터를 어떤 식으로 전달하는가에 대한 약속.  
함수 호출 후에 ESP(스택포인터)를 어떻게 정리해야하는지에 대한 약속.  
으로 정의할 수 있다.

디버그 심볼을 확인하면 Name Mangling되는 것을 확인할 수 있는데 아래 Calling Convention에 따라 Mangling되고 있던 것이였다. 
지금까지 c++filt같은 명령어로 Demangle하여 편하게 디버깅만 해봤지 자세히 살펴보지를 못 한것 같다.

그래서 아래에서 한 번 자세히 살펴보도록 한다.

## x86 아키텍처에 쓰이는 3가지 Calling Convention
x86아키텍처의 _cdecl, _stdcall, _fastcall에 대해서 확인해보록 한다. 해당 Calling Convention의 경우 MSVC는 물론 GNUC계열 컴파일러에서도 지원한다.

### _cdecl
* 인자파싱: 오른쪽 -> 왼쪽
* 스택관리: Caller, 가변인자 허용
* Name Mangling: 함수 이름 앞에 _
* C와 C++함수의 기본 호출 규약

```c
#if defined(__GNUC__)
#  define _cdecl __attribute__((cdecl))
#endif // defined(__GNUC__)

_cdecl int MyFunction1(int a, int b)
{
  return a + b;
}

x = MyFunction1(2, 3);
```

```asm
_MyFunction1:
push ebp
mov ebp, esp
mov eax, [ebp + 8]
mov edx, [ebp + 12]
add eax, edx
pop ebp
ret


push 3
push 2
call _MyFunction1
add esp, 8
```

### _stdcall
* 인자파싱: 오른쪽 -> 왼쪽
* 스택관리: Calle
* Name Mangling: 함수 이름 앞에 _ 함수 이름 뒤에 @ 추가되고 @ 뒤에 매개변수의 전체바이트에 해당하는 10진수가 추가된다.
* Win32 API의 표준 규약.

```c
#if defined(__GNUC__)
#  define _stdcall __attribute__((stdcall))
#endif // defined(__GNUC__)

_stdcall int MyFunction2(int a, int b)
{
   return a + b;
}

x = MyFunction2(2, 3);
```

```asm
:_MyFunction2@8
push ebp
mov ebp, esp
mov eax, [ebp + 8]
mov edx, [ebp + 12]
add eax, edx
pop ebp
ret 8

push 3
push 2
call _MyFunction2@8
```

### _fastcall
* 인자파싱: 처음 2개의 DWORD 또는 더 작은 인자들은 ecx와 edx에 전달, 나머지 인자들은 Right-> Left
* 스택관리: Calle
* Name Mangling: 함수 이름 앞과 끝에 @가 추가되고 @뒤에 매개변수의 전체바이트에 해당하는 10진수가 추가된다.
* 이름부터 빠른 호출 규약.

```c
#if defined(__GNUC__)
#  define _fastcall __attribute__((fastcall))
#endif // defined(__GNUC__)

_fastcall int MyFunction3(int a, int b)
{
   return a + b;
}

x = MyFunction3(2, 3);
```

fastcall은 레지스터를 직접 이용하기때문에 속도가 빠르다.

```asm
:@MyFunction3@8
push ebp
mov ebp, esp ;many compilers create a stack frame even if it isn't used
add eax, edx ;a is in eax, b is in edx
pop ebp
ret

;the calling function
mov eax, 2
mov edx, 3
call @MyFunction3@8
```


## References
https://en.wikibooks.org/wiki/X86_Disassembly/Calling_Conventions  
https://recoverlee.tistory.com/25  
https://wendys.tistory.com/22  
  MSVC Compiler Calling Convention Options은 아래 사이트에서 확인 가능하다.  
https://docs.microsoft.com/en-us/cpp/build/reference/gd-gr-gv-gz-calling-convention?redirectedfrom=MSDN&view=vs-2019  
GCC function attributes로 stdcall, fastcall를 정의할 수 있다. 자세한 내용은 아래 사이트에서 확인가능  
https://gcc.gnu.org/onlinedocs/gcc-4.7.2/gcc/Function-Attributes.html
