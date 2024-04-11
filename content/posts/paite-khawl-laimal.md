---
layout: post
title: "Paite Khawl Laimal: Programming Language in Paite"
slug: "paite-khawl-laimal"
date: "2024-02-18"
comments: false
categories:
  - programming
  - project
tags:
  - paite
  - programming language
  - computers
  - golang
---

## Documentation

Paite Khawl Laimal, a procedural programming language is an interpreted language. This language keywords are based on [Paite](https://en.wikipedia.org/wiki/Paite_language).

The source code for this project is in [github](https://github.com/nga1hte/paite-khawl-laimal).

#### Installation

Pre-requisite: Golang

For linux and windows.

```bash
git clone https://github.com/nga1hte/paite-khawl-laimal.git
cd paite-khawl-laimal/
go build .
./main
```


#### Chibai Zogam

```bash
>> suahkhia("Chibai Zogam")
Chibai Zogam
```
#### Variables

The verb `huchin` is used to declare variables and initialise values. The language is not static, so we don't have to declare data types, we just have to declare its name and assign its value. INTEGER, STRINGS and BOOLEAN are supported at the moment.

```bash
>> huchin greeting = "Chibai Zogam"; 
>> huchin numbat = 8;              
>> huchin boolean = zuau;          
>> huchin name = "Joypu";
```

To retrieve the values of the variable we can call them using their identifier name.

```bash
>> greeting;   
Chibai Zogam
>> numbat;
8
>> boolean;
zuau
>> name;
Joypu
>> huchin total = numbat + 10; 
18
```

To print the value of variables to the screen(stdout) using `suahkhia` (print); a function built into the language.

```bash
>> suahkhia(variable)
Chibai Zogam
>> huchin total = 10 + 10;
>> suahkhia(total);
20
```

The language also supports arrays and hashmap to store different data types.

```bash
>> huchin min = ["Thangboi", "Lianboi", "Joypu", "Mungboi"];
>> huchin kum = [25, 24, 23, 22];
>> suahkhia(min[1]);
Lianboi
>> suahkhia(kum[2]);
23
>> huchin mihing = {"Thangboi": 25, "Lianboi": 24, "Joypu": 23, "Mungboi": 22};
>> suahkhia(mihing["Joypu"])
23
```

#### Operators

The language supports basic arithmetic operations like addition (+), subtraction (-), multiplication (*) and division (/). It also has prefix operator like negative (-) for negative integers.

```bash
>> huchin add = 2 + 2;
>> huchin subtract = 4 - 2;
>> huchin multi = add * subtract;
>> suahkhia(multi);
8
>> huchin div = -multi / 2;
>> suahkhia(div);
-4
```

There is also support for rational operators like < (less than), > (greater than), == (equal to), != (not equal to).

```bash
>> huchin check = 5 > 3;
>> suahkhia(check);
tak
>> huchin check2 = 5 < 3;
>> suahkhia(check);
zuau
>> huchin check3 = 5 == 5;
>> suahkhia(check3)
tak
>> huchin check4 = 5 != 10
>> suahkhia(check4)
tak
```

#### Conditionals

The language also supports conditions to check whether an expression is true. `ahihleh` (if) and `ahihkeileh` (else).

```bash
>> huchin val1 = 5;
>> huchin val2 = 10;
>> ahihleh (val1 > val2) {
    suahkhia("val1 is greater than val2");
  } ahihkeileh {
    suahkhia("val2 is greater than val1");
  };
val2 is greater than val 1
```

#### Functions

The language also supports functions. Function declarations are done using the verb `thilhihna` followed by the parameters. We use the verb `lehkik` to return a value.

```bash
>> huchin add = thilhihna(x, y) {
  lehkik x + y;
};
>> suahkhia(add(5, 10));
10
```

The language also comes with some builtin functions like `saudan` to return the length of arrays, strings.

```bash
>> huchin name = "Mary";
>> suahkhia(saudan(name));
4
>> huchin numbat = [1, 2, 3, 4, 5];
>> suahkhia(suadan(numbat));
5
```
`amasa`, `nanung`, `sawnlut`, `amasalouteng` are all functions that can be used on arrays to retrieve and manipulate the array.

```bash
>> huchin arr = [1, 2, 3, 4, 5];
>> suahkhia(amasa(arr));
1
>> suahkhia(nanung(arr));
5
>> suahkhia(sawnlut(arr, 6));
[1, 2, 3, 4, 5, 6]
>> suahkhia(amasalouteng(arr))
[2, 3, 4, 5]
```

The language currently doesn't have loops but can be implemented by using functional programming. Here is a code to print numbers.

```bash

>> huchin loop = thilhihna(x) {
    ahihleh (x > 10) {
      lehkik x;
   }
   suahkhia(x);
   loop(x + 1);
  }
>> loop(1)
1
2
3
4
5
6
7
8
9
10
```

The language is still in its infancy and is experimental at best. A work in progress, much updates await.

> This project is based on the book Writing an Interpreter in Go by Thorsten Ball.
