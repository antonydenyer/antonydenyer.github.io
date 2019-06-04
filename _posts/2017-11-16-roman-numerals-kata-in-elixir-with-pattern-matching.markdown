---
layout: post
title: "Roman Numerals Kata in Elixir with Pattern Matching"
date: 2017-11-16 14:24:09 +0000
comments: true
redirect_from: '/blog/2017/11/16/roman-numerals-kata-in-elixir-with-pattern-matching/'
categories: ["elixir", "tdd"]
---

## Roman Numerals Kata

I wanted to have a play about with pattern matching a bit more in Elixir so I thought I would see if I could do the roman numerals kata using pattern matching and no looping.

### Basic tests I -> III

I started off in the usual TDD way of writing a test for "I"

```
  test "one is I" do
    assert Roman.to_roman(1) == "I"
  end

  def to_roman(number) 
    String.duplicate("I", number) 
  end
```
[18d663](https://github.com/antonydenyer/elixir-roman-numerals/commit/18d663c16b4004221a3f267dd5d9e1935fc7b9fa)

No great shakes, we can print up to "III". So far so simple. The next step I normally take is "IV". It's a bit more interesting in elixir as I wanted to exercise [Guard Clauses](https://hexdocs.pm/elixir/master/guards.html) in elixir.

### Pattern Matching IV and V

What you need to do is define a guard for when you want to execute the function.

```
  test "four is IV" do
    assert Roman.to_roman(4) == "IV"
  end

  def to_roman(number) when number == 4 do
    "IV"
  end
```
[9dcb82](https://github.com/antonydenyer/elixir-roman-numerals/commit/9dcb82d840fd9809d4943a08c9ad9540b931697a)

All we have here is a conditional guard clause to say if the number is equal to 4 then run this function. If it doesn't match, we don't care. It is not our responsibility to deal with, which for me is extremely interesting, we are in effect following the [SRP](https://en.wikipedia.org/wiki/Single_responsibility_principle) at a function level. 

One thing that caught me out was guard precedence, basically, it's top to bottom.

Let's move on, next is 5

```
  def to_roman(number) when number == 5 do
    "V"
  end
```
[b427555](https://github.com/antonydenyer/elixir-roman-numerals/commit/b4275554cbd2a0b4710a061ed0e30967c2d80b7f)

Boom, done. 

### Implementing VI

You may have noticed that we haven't implmeneted anything interesting yet. We have some behaviour for 1-3, but that is all. 4 and 5 are just lookups. The next exciting thing to implement is 6, which is funny because 4 hasn't added anything to help us drive out the behaviour. For 6 (VI) we need to accumulate 5 and 1. So let's get started

```
  test "six is VI" do
    assert Roman.to_roman(6) == "VI"
  end

```
What I want to do is match anything greater than 5 and return "V" along with the remaining match for 1. This what I came up with:

```
  def to_roman(number) when number >= 5 do
    to_roman(number - 5, "V")
  end

```
The problem is we don't have anything that will match ```to_roman/2```, so let's implement that. 

```
  def to_roman(number, roman_accumulator) do
    roman_accumulator <> String.duplicate("I", number)
  end

```
[155de6](https://github.com/antonydenyer/elixir-roman-numerals/commit/155de6aab419d61703c545adb44ec9b7fba933c4)

And now are concatenating the two string together, the "V" and the "I". Now, I've taken a bit of leap, we have implemented 7 and 8. But, I think that's okay. 

Now, our code is looking a little bit messy. Let's go ahead and refactor it a bit.

``` 
  def to_roman(number, roman_accumulator) do
    roman_accumulator <> String.duplicate("I", number)
  end
  
  def to_roman(number) when number >= 5 do
    to_roman(number - 5, "V")
  end

  def to_roman(number) when number == 4 do
    to_roman(number - 4, "V")
  end

  def to_roman(number) do
    to_roman(number, "")
  end

```
[c79d40](https://github.com/antonydenyer/elixir-roman-numerals/commit/c79d40b282fc62fdcc6583a9395d0abefff7b31e)

We still don't quite have anything generic yet. Let's implement "IX" and "X" in the same way, get some tests round them.

[238510](https://github.com/antonydenyer/elixir-roman-numerals/commit/2385106fd1d5fc222dc665f2b626d8430aa7c1dd)

### A more generic algorithm

Now there's still nothing going on to help us design our algorithm, let's move onto "XIV". There's a bit more happening, we will need to combine "X" and "IV". When we run our test for 14 we get a test failure:
```
    Assertion with == failed
    code:  assert Roman.to_roman(14) == "XIV"
    left:  "XIIII"
    right: "XIV"
```
[e055fc](https://github.com/antonydenyer/elixir-roman-numerals/commit/7e055fcf054b6b1c90b57e3ee3d6a25839b59141)

So what we need to have is a way for "X" to fall through into "IV". To achieve this, I'm just going to add in the parameter. 

```
  def to_roman(number, roman_accumulator) when number >= 4 do
    to_roman(number - 4, roman_accumulator <> "IV")
  end
```
Now we are starting to see some design emerging, but we have two tests failing. 

```
  code:  assert Roman.to_roman(4) == "IV"
     left:  "IIII"
     right: "IV"
```
The reason is simple; we need to ensure that
```
  def to_roman(number, roman_accumulator) do
    roman_accumulator <> String.duplicate("I", number)
  end
```
doesn't catch the case for 4. 
We can achieve this in two ways; I'm going to implement both as I think it will make the code clearer.
We will implement a guard clause and change the order of the code.

[403e18](https://github.com/antonydenyer/elixir-roman-numerals/commit/403e1864bd363fbce0e69d9aee5caaf16a71631c)

And now our test passes. Now I think we're getting closer to something more generic. Let's implement "XVI" and tidy up. Again when we write our test for 16 we get
```
    Assertion with == failed
    code:  assert Roman.to_roman(16) == "XVI"
    left:  "XIVII"
    right: "XVI"
```
Now, what is going on here? It's fairly simple when you dig into it. We get the match for 10 (X), remainder 6. We then get match a match on the 6 to "IV" which is incorrect. We need it to match "V". So let's go ahead and implement that
```
  def to_roman(number, roman_accumulator) when number >= 5 do
    to_roman(number - 5, roman_accumulator <> "V")
  end
```

[da5e6a](https://github.com/antonydenyer/elixir-roman-numerals/commit/da5e6a7b6ef75dcd6eb718596abcdcaf3234993c)

Now I'm going to tidy up the 9 and 10 so that it follows the same pattern as 4 and 5. 

[78a7c4](https://github.com/antonydenyer/elixir-roman-numerals/commit/78a7c459ce47d1788e8a5c4f60215136af5d6286)

I think we've got something that will cover us until the next numeral, let's just double check with another test.

[8a7f9d](https://github.com/antonydenyer/elixir-roman-numerals/commit/8a7f9df649473aca38f2bb319f4ba80f1c00d8ea)

What I'm going to do now is go ahead and implement the rest of the algorithm and test it using a [doctest](https://elixir-lang.org/getting-started/mix-otp/docs-tests-and-with.html)

### Solution

[4a463fe](https://github.com/antonydenyer/elixir-roman-numerals/commit/4a463fe585c5ba0397dd724873471513f47442a1)

```
def to_roman(number, roman_accumulator) when number >= 1000 do
    to_roman(number - 1000, roman_accumulator <> "M")
  end
  
  def to_roman(number, roman_accumulator) when number >= 900 do
    to_roman(number - 900, roman_accumulator <> "CM")
  end
  
  def to_roman(number, roman_accumulator) when number >= 500 do
    to_roman(number - 500, roman_accumulator <> "D")
  end
  
  def to_roman(number, roman_accumulator) when number >= 400 do
    to_roman(number - 400, roman_accumulator <> "CD")
  end
  
  def to_roman(number, roman_accumulator) when number >= 100 do
    to_roman(number - 100, roman_accumulator <> "C")
  end

  def to_roman(number, roman_accumulator) when number >= 90 do
    to_roman(number - 90, roman_accumulator <> "XC")
  end

  def to_roman(number, roman_accumulator) when number >= 50 do
    to_roman(number - 50, roman_accumulator <> "L")
  end

  def to_roman(number, roman_accumulator) when number >= 40 do
    to_roman(number - 40, roman_accumulator <> "XL")
  end

  def to_roman(number, roman_accumulator) when number >= 10 do
    to_roman(number - 10, roman_accumulator <> "X")
  end

  def to_roman(number, roman_accumulator) when number >= 9 do
    to_roman(number - 9, roman_accumulator <> "IX")
  end

  def to_roman(number, roman_accumulator) when number >= 5 do
    to_roman(number - 5, roman_accumulator <> "V")
  end

  def to_roman(number, roman_accumulator) when number == 4 do
    roman_accumulator <> "IV"
  end

  def to_roman(number, roman_accumulator) when number <= 3 do
    roman_accumulator <> String.duplicate("I", number)
  end

  def to_roman(number) do
    to_roman(number, "")
  end
```
Pretty cool I think :)

### Removing the String.duplicate
As a bit of a mental exercise, I thought it would be interesting to remove the ```String.duplicate``` basically you end up with a pattern match for 0,1,2 and 3. Interestingly most people don't test 0 when doing this Kata. It becomes obvious that you need it when you don't implement a pattern match for 0. 

[e75fd6](https://github.com/antonydenyer/elixir-roman-numerals/commit/e75fd6c4f82bf9e7e9e094cbf350d57330fbec00)

The other thing I wanted to look at was removing the unnecessary fall through for 4 down to 0.

[358fade](https://github.com/antonydenyer/elixir-roman-numerals/commit/358fadebaaab2514bf04cc8b10f2e0971b20f13e)

### Summary
We've learnt a bit about pattern matching and guard clauses in Elixir as well as introducing doctest. This sort of thing can be done in a few different ways. 

My personal favourite is 

```
String.duplicate("I", number)
    |> String.replace("IIIII", "V")
    |> String.replace("VV", "X")
    |> String.replace("XXXXX", "L")
    |> String.replace("LL", "C")
    |> String.replace("CCCCC", "D")
    |> String.replace("DD", "M")
    |> String.replace("IIII", "IV")
    |> String.replace("VIV", "IX")
    |> String.replace("XXXX", "XL")
    |> String.replace("LXL", "XC")
    |> String.replace("CCCC", "CD")
    |> String.replace("DCD", "CM")
```

It's probably a bit terse; you have to understand whats going on. It's a little hard to figure our but it's kind of fun. 