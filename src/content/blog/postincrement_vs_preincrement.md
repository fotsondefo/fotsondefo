---
title: 'Post-incrementation vs pre-incrementation'
publishDate: '2025-08-21'
updatedDate: '2025-08-21'
description: 'Is it really necessary to go to the trouble of choosing?'
tags:
  - C++
  - Optimization
language: 'English'
draft: false
---

One day, a friend who also uses C++ looked at my code and said : 
*"Did you know that you can improve your performance by choosing pre-incrementation rather than post-incrementation?"*

The code was fairly simple, a classic when we write loop: 
```cpp
for (int i = 0; i < n; i++)
{
    // do something
}
```

I didn't really understand what he explained to me that day, but I remember being skeptical.

How it's possible that millions of programmers use this for-loop template if this loop is not the fastest? Isn't there a problem?

Intuitively, I know that things in computer science are never easy to explain in a few sentences.
I use to say that computer science is the science of *"it depends"*. Why ? Because, most of the time it's never what you think it is at first.

In this article, I would like to present some research on this topic to better understand (1) why some people claim that pre-increment is faster than post-increment, and (2) if it's really a big deal to choose pre-increment over post-increment.

## Is i++ slow?

Let's take this code as an example.

```cpp
int main()
{
    int i = 100;
    int x;

    x = i++; // post-increment

    x = ++i; // pre-increment

    return 0;
}
```


The argument that's generally put forward when comparing these two operations is thtat post-incrementation (i++), before increasing the value of the variable by 1, performs an extra step to create a copy of the value <a href="https://www.geeksforgeeks.org/dsa/gfact-which-is-faster-pre-increment-i-vs-post-increment-i/" target="_blank">[1]</a>.

If you doubt about this, just look at the generated machine code.

```asm
push    rbp
mov     rbp, rsp

;;;;  int i = 100
mov     DWORD PTR [rbp-4], 100          

;;;;  x = i++
mov     eax, DWORD PTR [rbp-4]  ; move the value of i (100) into eax
lea     edx, [rax+1]            ; load in edx the address of the rax+1 (101)
mov     DWORD PTR [rbp-4], edx  ; assign the value of edx (101) to i 
mov     DWORD PTR [rbp-8], eax  ; assign the old value of i (eax=100) to x

;;;;  x = ++i
add     DWORD PTR [rbp-4], 1    ; add 1 to i, and store the result in i
mov     eax, DWORD PTR [rbp-4]  ; assign the value of i (101) to eax
mov     DWORD PTR [rbp-8], eax  ; assign the eax (101) to x

mov     eax, 0
pop     rbp
ret
```

The **mov**, **lea** and **add** instructions are all executed in one CPU cycle. As we can see, post-incrementation (a++) takes 4 cyles while pre-incrementation (++a) just takes 3. In other words, post-incrementation takes longer to execute so it's slower, right?

Let's take another code example.

<div style="display: grid;grid-template-columns: repeat(2,minmax(0,1fr));gap: 2rem;">

```cpp
int main()
{
    int i = 100;
    int x;

    i++; // post-increment

    ++i; // pre-increment

    return 0;
}
```

```asm
push    rbp
mov     rbp, rsp

mov     DWORD PTR [rbp-4], 10 ; int i = 100

add     DWORD PTR [rbp-4], 1  ; i++

add     DWORD PTR [rbp-4], 1  ; ++i

mov     eax, 0
pop     rbp
ret
```

</div>

That's interesting. Now, pre-increment and post-increment have the same instructions. Both operations now simply add 1 to the memory address **[rbp-4]** corresponding to the variable **i**. But why, you may ask? Well, the reason is compiler optimization.

In the first case, the compiler must preserve the semantics of the language. Indeed, the expected behavior when there is assignment is different: one must first store the value before increasing it, while the other increases the value directly. So the compiler generates different instructions here.
In the second case, the return value of the increment operations is not used at all; we simply increase the value of i by 1. The two operations have the same effect, so we can generate the same machine code.

As you can see, the compiler is smart enough to recognize that the return value is not used, the final effect is identical in both cases, and it can therefore generate the same optimized code: a simple **add** instruction.


## So, should we really care?

Answer: no.

Most of the time, if not more than 99% of the time, we use incrementation without assigning its result to a variable. And as we have just seen, when the return value is not used, there is no difference in performance between i++ and ++i in the compiled code.

I could have recommended using ++i by default so you don't have to worry about choosing between the two, but I think that's a bit ridiculous. I mean, you don't have to worry about 1% of your code that will use increment + assignment, which won't really change the program speed much.


## References

| No. | Name          | Links                                                                                        | Consultation date |
| --- | ------------- | -------------------------------------------------------------------------------------------- | ----------------- |
| 1   | geeksforgeeks | https://www.geeksforgeeks.org/dsa/gfact-which-is-faster-pre-increment-i-vs-post-increment-i/ | 21/08/2025        |
