---
layout: post
title:  "python调用C和C++接口动态库"
date:   2017-05-10 09:55:39 +0800
categories: 编程
---



# python调用C接口

编写c接口代码
```c
//capi.c
#include <stdio.h>
#include <stdlib.h>

char* foo(char* in)
{
  printf("foo function input: %s\n", in);
  return "OK";
}

char* exam(char *in)
{
  printf("exam function input: %s\n",in);
  return "OK";
}
```

编译c接口代码:
```log
# gcc -o libcapi.so -shared -fPIC capi.c
```
编译后在本地目录下生成libcapi.so为C接口代码的动态库
编辑python代码
```Python
#pycall.py
import ctypes

libcapi = ctypes.cdll.LoadLibrary("./libcapi.so")
libcapi.foo("good")
libcapi.exam("good")
print "success!"
```

调用Python测试代码
```log
# python pycall.py
foo function input: good
exam function input: good
success!
```
测试成功!

# python调用C++接口
编辑C++接口代码
```cpp
//cppapi.cpp
#include <iostream>

std::string foo(std::string in)
{
  std::cout << "foo function input: " << in << std::endl;
  return "OK";
}

char* exam(char *in)
{
  std::cout << "exam function input: " << in << std::endl;
  return (char*)"OK";
}
```
编译C++接口代码
```log
# g++ -o libcppapi.so -shared -fPIC  cppapi.cpp
```
编写python测试脚步
```python
#pycall.py
import ctypes

libcppapi = ctypes.cdll.LoadLibrary("./libcppapi.so")
libcppapi.foo("good")
libcppapi.exam("good")
print "success!"
```
执行脚步测试:
```log
# python pycall.py
Traceback (most recent call last):
  File "pycall.py", line 5, in <module>
    libcppapi.foo("good")
  File "/usr/lib/python2.7/ctypes/__init__.py", line 378, in __getattr__
    func = self.__getitem__(name)
  File "/usr/lib/python2.7/ctypes/__init__.py", line 383, in __getitem__
    func = self._FuncPtr((name_or_ordinal, self))
AttributeError: ./libcppapi.so: undefined symbol: foo

```
c++动态库中找不到foo方法,是因为c和c++的函数签名不一致，因为c++支持重载，所以按c++的方式是找不到同名的c函数的,先查看C接口libcapi.so动态库符号表
```log
# nm -D  libcapi.so
0000000000201038 B __bss_start
                w __cxa_finalize
0000000000201038 D _edata
0000000000201040 B _end
000000000000072d T exam
000000000000075c T _fini
0000000000000700 T foo
                w __gmon_start__
0000000000000598 T _init
                w _ITM_deregisterTMCloneTable
                w _ITM_registerTMCloneTable
                w _Jv_RegisterClasses
                U printf

```
C库中可以找到foo函数符号和exam函数符号,再看看C++动态库libcppapi.so 的符号表
```log
# nm -D libcppapi.so 0000000000202088 B __bss_start
                 U __cxa_atexit
                 w __cxa_finalize
0000000000202088 D _edata
0000000000202090 B _end
0000000000000dc8 T _fini
                 w __gmon_start__
                 U __gxx_personality_v0
0000000000000a58 T _init
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
                 w _Jv_RegisterClasses
                 U __stack_chk_fail
                 U _Unwind_Resume
0000000000000c50 T _Z3fooSs
0000000000000d17 T _Z4examPc
                 U _ZNSaIcEC1Ev
                 U _ZNSaIcED1Ev
                 U _ZNSolsEPFRSoS_E
                 U _ZNSsC1EPKcRKSaIcE
                 U _ZNSt8ios_base4InitC1Ev
                 U _ZNSt8ios_base4InitD1Ev
                 U _ZSt4cout
                 U _ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_
                 U _ZStlsIcSt11char_traitsIcESaIcEERSt13basic_ostreamIT_T0_ES7_RKSbIS4_S5_T1_E
                 U _ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc
```
找到foo和exam函数的符号表变成了_Z3fooSs和_Z4examPc,因此使用C++写的库函数在对外提供Python调用的接口是需要转换成C接口,需要对源代码做适当的处理.
```cpp
//cpp2capi.cpp
#include <iostream>

//Macro definition EXPORT_C_API is true export c api, otherwise cpp api
#define EXPORT_C_API true

//notify EXPORT_C_API when compiled
#if EXPORT_C_API
#warning("libcpp2capi export c api")
#else
#warning("libcpp2capi export cpp api")
#endif


#if EXPORT_C_API
namespace cppapi{
#endif

//cpp api follow
//----------------------------------------------------

std::string foo(std::string in)
{
  std::cout << "foo function input: " << in << std::endl;
  return "OK";
}

char* exam(char *in)
{
  std::cout << "exam function input: " << in << std::endl;
  return (char*)"OK";
}
//--------------------------------------------------
//cpp api above


#if EXPORT_C_API
}

extern "C" {
  #include <string.h>

  //convert string to char*
  char* convert(std::string str)
  {
    std::cout << "convert " << str << "..." << std::endl;

    int length = str.length();
    char* p_char = new char[length+1];
    memcpy(p_char, str.c_str(), length);
    strcat(p_char, "\0");
    return p_char;
  }

  char* foo(char* in)
  {
    std::cout << "foo(" << in << ")" << std::endl;
    std::string ret = cppapi::foo(in);

    return convert(ret);
  }

  char* exam(char *in)
  {
    return cppapi::exam(in);
  }

}
#endif


```  

然后编译cpp2capi.cpp:  
```log
# g++ -o libcpp2capi.so -shared -fPIC cpp2capi.cpp
cpp2capi.cpp:9:2: warning: #warning ("libcpp2capi export c api") [-Wcpp]
 #warning("libcpp2capi export c api")
  ^
# ls
cpp2capi.cpp  libcpp2capi.so  pycall.py

# python pycall.py
foo(good)
foo function input: good
convert OK...
exam function input: good
success!
```

调用成功,查看libcpp2capi.so动态库中的符号表
```log
# nm -D libcpp2capi.so
00000000002020d0 B __bss_start
0000000000001089 T convert
                 U __cxa_atexit
                 w __cxa_finalize
00000000002020d0 D _edata
00000000002020d8 B _end
00000000000012ae T exam
0000000000001328 T _fini
0000000000001137 T foo
                 w __gmon_start__
                 U __gxx_personality_v0
0000000000000cf0 T _init
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
                 w _Jv_RegisterClasses
                 U memcpy
                 w __pthread_key_create
                 U __stack_chk_fail
                 U _Unwind_Resume
0000000000000f70 T _ZN6cppapi3fooESs
0000000000001037 T _ZN6cppapi4examEPc
                 U _Znam
                 U _ZNKSs5c_strEv
                 U _ZNKSs6lengthEv
                 U _ZNSaIcEC1Ev
                 U _ZNSaIcED1Ev
                 U _ZNSolsEPFRSoS_E
                 U _ZNSsC1EPKcRKSaIcE
                 U _ZNSsC1ERKSs
                 U _ZNSsD1Ev
                 U _ZNSt8ios_base4InitC1Ev
                 U _ZNSt8ios_base4InitD1Ev
                 U _ZSt4cout
                 U _ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_
                 U _ZStlsIcSt11char_traitsIcESaIcEERSt13basic_ostreamIT_T0_ES7_RKSbIS4_S5_T1_E
                 U _ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc
```
符号表中存在foo和exam函数符号.
