---
date: 2021-05-15
linktitle: Curious Case of Modulo Operator
next: /2021/05/build-it-with-spring-boot-crud-operations-part-1/
prev: /2021/04/unit-testing-in-maven-testng-tests/
title: Curious Case of Modulo Operator
weight: 10
tags: [ "coding", "programming", "mathematics", "modulus", "bitwise", "binary" ]
categories: [ "Coding", "Programming" ]
---


If you solve programming and coding problems on various platforms like [HackerRank][1], [CodingBat][2], [ProjectEuler][3], etc. there are a lot of problems which will require some sort of identification of even or odd numbers. Like [this problem][4] on [HackerRank][1].

The most basic and ubiquous solution to identify if a number is even or odd is done by most programmers with the help of the [modulus operator][5] - `%`. In essense, this operator returns the remainder value when a number is divided by another. For example,

```
7 % 3 = 1    // (3 x 2) + 1(this one is the remainer)
```

With this knowledge, if you have to detect if a number is even or odd, you can check if the modulus of the number with 2 leaves 1 or not. So, a trivial method in Java for this implementation would be as below.

```java
public static boolean isOddBuggy(final int number) {
  return number % 2 == 1;
}
```

You might be wondering why the method name is `isOddBuggy`, because the code is buggy. We will come to that in a bit, but let's run it with two sample inputs.

```java
System.out.println("10 is odd ? => " + isOddBuggy(10));
System.out.println("3 is odd ? => " + isOddBuggy(3));
```

And the output on the console is below.

```
10 is odd ? => false
3 is odd ? => true
```

You will say, hang-on, everything is working, what is wrong here?

Let's add one more case for negative numbers and see if our code works now.

```java
System.out.println("-3 is odd ? => " + isOddBuggy(-3));
```

and now it returns `false`.

```
-3 is odd ? => false
```

Wait, what? Why `false`





{{< articlead >}}

## Modulo Operator

The mathematical formula for Modulo `(a % b = r)` can be represented as

<div class="notification" style="text-align:center;">
  \(r = a - b \times trunc \left( \frac {a} {b} \right) \)
</div>

So, we have a division in picture and in simplest terms, division is successive subtraction. Also, most implementations of programming language preserve the sign in the result which matches to the sign of `a`.

That is why our modulo operation for `-3 % 2` returns `-1`, preserving the sign from `-3`.





## Fixing the issue

With the above knowlege, we can fix our code, to either check for absolute value or remainder or we also check to include `-1`. Or better, we check for event numbers always and use the inverse whereever we want odd behaviour, since 0 will always be sign independent. All these implementations are below.

```java
public static boolean isOddFixed1(final int number) {
  return Math.abs(number % 2) == 1;
}

public static boolean isOddFixed2(final int number) {
  return (number % 2 == 1) || (number % 2 == -1);
}

public static boolean isOddFixed3(final int number) {
  return !isEven(number);
}

public static boolean isEven(final int number) {
  return number % 2 == 0;
}

```





{{< articlead >}}

## Even better approach

The above implementations still have the successive division inherent to them. It is always going to be slower than addition or subtraction or bitwise comparison which are much faster for CPU to process. Can we use any of these?

If we take a look at the binary representation([in nibbles][6]) of a few numbers below.

```
 3 => 0011
 4 => 0100
 7 => 0111
14 => 1110
```

Did you see any pattern? All Odd numbers have the least significant bit as 1 while all even numbers have least significant bit as 0. If we do some operation and get rid of all the bits except the least significant bit, then we can do this even-odd identification without the modulus operator and it will be faster.

And to preserve only the least significant bit, we can perform logical AND operation on bits on the number and 1. Here is an example.

```
+-------------------------+-------------------------+
|       3 =>  0011        |      14 =>  1110        |
|       1 =>  0001        |       1 =>  0001        |
|     ------------        |     ------------        |
|     AND =>  0001        |     AND =>  0000        |
|     ------------        |     ------------        |
+-------------------------+-------------------------+
```

So, our fast even or odd methods will be as below.

```java
public static boolean isOddFast(final int number) {
  return (number & 1) == 1;
}

public static boolean isEvenFast(final int number) {
  return (number & 1) == 0;
}
```

Happy coding!




  [1]: https://www.hackerrank.com
  [2]: https://codingbat.com/java
  [3]: https://projecteuler.net/archives
  [4]: https://www.hackerrank.com/challenges/java-if-else/problem
  [5]: https://en.wikipedia.org/wiki/Modulo_operation
  [6]: https://en.wikipedia.org/wiki/Nibble
