---
layout: post
title: 순환참조
date: 2019-02-26 18:07:35
tags:
image:
comments: true
---

두 값이 서로를 참조하는 상황으로 소멸이 제대로 안 이루어질 수 있다.

### 순환참조 회피 방법

Child에 해당하는 strong reference count를 올리지 않으면 된다.
weak reference count를 올린다.

Parents에 해당하는 객체에서 strong reference count를 올려 제어하자.

```
# Strong Reference counting
std::shared_ptr
    std::shared_ptr<Parent> parent;
retain : objective-c++
    @property(retain) Parent* parent;
strong for ARC : objective-c++
    @property(strong) Parent* parent;
StrongReference : Java
    Parent parent = new Parent();

# Weak Reference counting
std::weak_ptr
    std::weak_ptr<Child> child;
assign : objective-c++
    @property(assign) Child* child;
weak for ARC : objective-c++
    @property(weak) Child* child;
WeakReference : Java
    WeakReference<Child> child = new WeakReference<Child>();

```

class간의 순환참조 방지하는 방법으로는 interface를 만들어 해당 interface를 의존하는 클래스와 구현되어 있는 클래스를 만들어 해결한다.

### Reference
https://www.javarticles.com/2016/10/java-weakreference-example.html
https://d2.naver.com/helloworld/329631
http://egloos.zum.com/sweeper/v/3059940
https://outofbedlam.github.io/swift/2016/01/31/Swift-ARC-Closure-weakself/