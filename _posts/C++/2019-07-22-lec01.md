---
layout: post
title:  "C++ 01 - 기초 (1)"
date:   2019-07-22 20:57:00 +0900
author: Gnues
categories: language
tags:	cpp
---

그간 사용해오던 언어들을 다시 정리하고 애매모호한 내용들을 바로잡기 위한 글입니다.

## Variables - 변수

`C++`는 타입 제약이 엄격한(`Strongly-Typed`) 언어이다.

모든 변수는 타입을 가지며 한 번 정해진 타입은 절대 변하지 않는다.

기본 타입은 내장타입(`Intrinsic Type`)이라고 하며 `char`, `int` 등이 존재한다.

| Type names*            | Notes on size / precision                          |
| ---------------------- | -------------------------------------------------- |
| char                   | Exactly one byte in size. At least 8 bits.         |
| char16_t               | Not smaller than char. At least 16 bits.           |
| char32_t               | Not smaller than char16_t. At least 32 bits.       |
| wchar_t                | Can represent the largest supported character set. |
| signed char            | Same size as char. At least 8 bits.                |
| signed short int       | Not smaller than char. At least 16 bits.           |
| signed int             | Not smaller than short. At least 16 bits.          |
| signed long int        | Not smaller than int. At least 32 bits.            |
| signed long long int   | Not smaller than long. At least 64 bits.           |
| unsigned char          | (same size as their signed counterparts)           |
| unsigned short int     | (same size as their signed counterparts)           |
| unsigned int           | (same size as their signed counterparts)           |
| unsigned long int      | (same size as their signed counterparts)           |
| unsigned long long int | (same size as their signed counterparts)           |
| float                  |                                                    |
| double                 | Precision not less than float                      |
| long double            | Precision not less than double                     |
| bool                   |                                                    |
| void                   | no storage                                         |
| decltype(nullptr)      |                                                    |

각 타입의 정확한 범위는 아키텍쳐에 따라 다르다. `int`는 16, 32, 64비트가 될 수 있다.

크로스 플랫폼 환경에서 작동하게 하기 위해서 동일한 크기의 변수를 사용하려는 노력이 필요할 때도 있다.

`Microsoft`에서는 크기가 지정된 변수 타입을 지원한다.

```c++
__int8 nSmall;      // Declares 8-bit integer = char
__int16 nMedium;    // Declares 16-bit integer = short
__int32 nLarge;     // Declares 32-bit integer = int
__int64 nHuge;      // Declares 64-bit integer = long long
```

_int8, __int16 및 __int32 형식은 동일한 크기를 가진 ANSI 형식에 대한 동의어이다.

이런 고정크기의 타입은 여러 플랫폼에서 동일하게 동작하는 이식 가능한 코드 작성에 유용하다.

***

## const - 상수

상수는 문법적으로 분변이라는 추가 속성을 갖는 특별한 변수이다.

```c++
const int ci1 = 2;
const int ci;     // error: not initialized
const float pi = 3.14159;
const char cc = 'a';
const bool cmp = ci1 < pi;
```

상수는 변경할 수 없으므로 선언과 동시에 값을 반드시 할당해야한다.

***

## 리터럴

`2`나 `3.14` 같은 리터럴도 입력 할 수 있다.

정수는 자릿수에 따라 `int`, `long`, `unsigned long` 타입으로 취급된다.

소수나 지수는 `double`타입으로 취급된다.

다른 리터럴 타입은 접미사를 추가하면 된다.

| Literal            | Type          |
| ------------------ | ------------- |
| 2                  | int           |
| 2u, 2U             | unsigned      |
| 2l, 2L             | long          |
| 2ul, 2uL, 2Ul, 2UL | unsigned long |
| 2.0                | double        |
| 2.0f, 2.0F         | float         |
| 2.0l, 2.0L         | long double   |

내장된 숫자 타입들 사이의 **묵시적 변환**을 통해 기대하는 타입으로 값을 설정해준다.

때문에 리터럴의 타입을 명시적(강제 변환, `Coercion`)으로 선언할 필요는 없다.

리터럴 타입에 주목해야 하는 경우는 아래와 같다.

### 유용함

특정 타입 사이의 연산만 지원하는 경우 리터럴의 타입을 명시해서 사용한다.

```c++
std::complex<float> z(1.3, 2.4), z2;
z2 = 2 * z;     // err : int * complex<float> 없음
z2 = 2.0 * z;   // err : double * complex<float> 없음
z2 = 2.0f * z;  // good : float * complex<float> 존재
```

### 애매모호함

함수를 서로 다른 인수 타입으로 오버로딩했을 때, `0`과 같은 인수는 애매모호하다.

`0U`와 같이 한정된 인수로 정확하게 일치하는 인수를 넘겨줄 수 있다.

### 정확함

`long double`타입으로 작업할 경우 정확도 문제가 발생한다.

한정되지 않은 리터럴은 `double`타입이므로 `long double`변수로 할당전에 데이터를 잃는다.

```c++
long double th = 0.33333333333333333333333;   // 자릿수를 잃어버림
long double th2 = 0.33333333333333333333333L; // 정확
```

이외에도 8진수를 표시하는 리터럴,

```c++
int o1 = 042;   // int o1 = 34;
int o2 = 089;   // err: 8진수는 자리마다 0 ~ 7
```

또는 16진수를 표시하는 리터럴이 있다.

```c++
int h1 = 0x42;  // int h1 = 66;
int h2 = 0xfa;  // int h2 = 250;
```

> ### **C++14**
>
> C++14는 `0b`, `0B` 접두사를 붙여 2진수 리터럴을 지원한다.
>
> ```c++
> int b1 = 0b11111010;  // int b1 = 250;
> ```
>
> 또한, 길이가 긴 리터럴의 가독성을 향상하기 위해 `'`로 숫자를 분리할 수 있다.
>
> ```c++
> long d = 6'546'687'616'861'129L;
> unsigned long ulx = 0x139'ae3b'2ab0'94f3;
> int b = 0b101'1001'0011'1010'1101'1010'0001;
> const long double pi = 3.141'592'653'589'793'238'462L;
> ```

문자열 리터럴은 `char` 배열에 할당하거나 `string` 타입을 이용한다.

```cpp
char s1[] = "Old C style";
std::string s2 = "C++ style";
std::string longStr = "This is very long and clumsy text"
                      "that is too long for one line.";
                      // 매우 긴 문자열은 분할할 수 있다.
```

***

## References

- Modern C++ 입문, 피터고츠슐링
- [MicroSoft Visual C++](https://docs.microsoft.com/ko-kr/cpp/cpp/int8-int16-int32-int64?view=vs-2019)
- [Cplusplus](http://www.cplusplus.com/doc/tutorial/variables/)
