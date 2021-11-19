---
layout: article
title: C++ Function Template Voodoo
key: cpp-function-template-voodoo
tags: C++ ProgrammingLanguage Hacks Template
---

<!-- more -->

I came across this in Ceph source code

```c++
class ConfigProxy {
  ...

  template<typename T>
  const T get_val(const std::string_view key) const {
    std::lock_guard l{lock};
    return config.template get_val<T>(values, key);
  }

  ...
};
```

Whoa wait! What? Did you ever seen a literal space ` ` in member function name?

```c++
    return config.template get_val<T>(values, key);
```


That extra `template ` seems to be around for quite some time, I found
[this StackOverflow question](https://stackoverflow.com/questions/1840253/template-member-function-of-template-class-called-from-template-function#)
from almost 12 years ago!

```c++
template<class X> struct A {
  template<int I> void f() {}
};

template<class T> void g()
{
  A<T> a;
  a.f<3>(); // Compilation fails here
}

int main(int argc, char *argv[])
{
  g<int>();
}
```

At a first glance, it seems pretty obvious that we want to call `a.f()` with
template argument specialized as `3`.

But it fails with

```console
$ g++ -std=c++20 test.cc
test.cc: In function ‘void g()’:
test.cc:17:16: error: expected primary-expression before ‘)’ token
   17 |         a.f<3>();
      |                ^
test.cc: In instantiation of ‘void g() [with T = int]’:
test.cc:26:8:   required from here
test.cc:17:12: error: invalid operands of types ‘<unresolved overloaded function type>’ and ‘int’ to binary ‘operator<’
   17 |         a.f<3>();
      |         ~~~^~
```

Why? Well, take a close look at the error log, you can see that the compiler
say otherwise: __It trying to compare `a.f` with `3`, and sees `a.f < 3` as
a translation unit!__ It's basically that nested-template-needs-to-be-declared-as-`vector<vector<X> >`-with-a-space tokenizer voodoo all over again! In 2021!

Therefore, as an adhoc patch, you need to explicitly give the keyword
`template` before your templated member function, so the tokenizer will
recognize that what's after `.` or `->` is a function, and can never be a
member variable, and treat `<3>` as a whole. The following example with extra
spaces may demonstrate this better

```c++
  // test.cc:19:14: error: invalid operands of types ‘<unresolved overloaded
  // function type>’ and ‘int’ to binary ‘operator<’
  //  19 |         (a.f < 3) > ();
  //     |         ~~~~~^~~~
  (a.f < 3) > ();       // equivalent to a.f<3>(), at least to the tokenizer
  operator<(a.f, 3) > (/*an empty parenthesized expression*/);

  a. template f <3> (); // a.template f<3>(), will compile
```

The more you know!
