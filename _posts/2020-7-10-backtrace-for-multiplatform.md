---
layout: post
title: Release 빌드시 backtrace함수를 이용한 로그 출력 및 분석 방법.
date: 2020-07-1- 12:47:30
image:
comments: true
categories: macOS iOS Android Linux unix
tag: macOS iOS Android Linux unix
---

\_\_func\_\_, \_\_FILE\_\_, \_\_LINE\_\_ 매크로를 사용하여 로그를 출력하게 되면 소스코드 정보가 리터럴 형태로 외부에 노출될 수 있는 문제가 발생한다. 이를 해결하기 위해 Release빌드에서는 해당 매크로를 사용하지 않도록 하고  
릴리즈 빌드에서 정보를 얻기 위해서는 backtrace함수를 사용하여 심볼정보를 얻어오도록 한다.

### 리터럴 문자 확인 방법
```bash
# for macOS
$ objdump -section="__cstring" -s ./test.dylib
$ objdump -section="__cstring" -s ./test.o
# for Linux, Android
$ objdump -section=".rodata" -s ./test.so
$ objdump -section=".rodata" -s ./test.o
```

### backtrace 함수 사용시 문제점.
backtrace, dladdr를 이용하여 함수명, DLL 이름을 얻어올 수 있는데 테스트한 결과 macOS의 경우 export되지 않은 함수 이름을 얻어올 수 있는 반면에 Linux, Android의 경우에는 export되지 않은 함수의 경우 이름을 못 얻어오는 문제가 있다. export된 함수만 이름을 얻어올 수 있었다.  

또한 Android의 경우 execinfo.h를 제공하지 않는데 대신에 unwind.h를 사용하여 구현해야하는 문제도 있다.  
export되지 않은 함수란 shared library에서 visibility hidden으로 설정된 함수를 말한다.  

### 기존 코드
아래 코드는 리터럴 스트링이 생기는 문제가 있다.
```c++
printf("test %s, %s, %s", __FILE__, __LINE__, __func__);
```

### 아래는 테스트 코드이다.

```c++
// Linux/Darwin
#include <execinfo.h>
#include <dlfcn.h>
 
#define PRINT_TRACE
void *array[1];
backtrace(array, 1);
Dl_info info;
if (dladdr(array[0], &info)) {
    printf("test %s, %s, %p, %p \n", info.dli_fname, info.dli_sname, info.dli_fbase, info.dli_saddr);
}
```

```c++
// Android
#include <unwind.h>
#include <dlfcn.h>
 
struct
android_backtrace_state
{
    void **current;
    void **end;
};
 
_Unwind_Reason_Code
android_unwind_callback(struct _Unwind_Context* context, void* arg)
{
    android_backtrace_state* state = (android_backtrace_state *)arg;
    uintptr_t pc = _Unwind_GetIP(context);
    if (pc) {
        if (state->current == state->end) {
            return _URC_END_OF_STACK;
        } else {
            *state->current++ = reinterpret_cast<void*>(pc);
        }
    }
    return _URC_NO_REASON;
}
 
void *array[1];
android_backtrace_state state;
state.current = array;
state.end = array + 1;
_Unwind_Backtrace(android_unwind_callback, &state);
 
Dl_info info;
if (dladdr(array[0], &info)) {
    printf("test %s, %s, %p, %p \n", info.dli_fname, info.dli_sname, info.dli_fbase, info.dli_saddr);
}
```

### backtrace 로그 분석
위 코드 로그를 분석하기 위해서는 macOS, iOS의 경우 atos 커맨드를 이용하고
Linux, Android의 경우 addr2line 커맨드를 사용하여 디버그 심볼을 통해 정확한 소스코드의 라인정보를 얻어올 수 있다.

### References
https://en.cppreference.com/w/cpp/preprocessor/replace#.23and.23.23_operators  
