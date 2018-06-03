---
layout: post
title: Assembly Basics - Part 13
date: 2017-10-14
category: Reverse-Engg
---

# do while statement translated into assembly

In this blog, we will look at how a do while loop is translated into assembly language by the compiler. For the purpose of this example, I have used the below code. 

{% highlight cpp %}
#include<stdio.h>

void main() {

	int index = 0;
	
	/*-while loop-*/
	do
	{
		index++;

		// some operations will happen here	
	
	} while(index != 5);
	
Outside:
	return;
}
{% endhighlight %}

In the above code sample, we have a do while loop and a set of instructions within it, that we are currently not concerned about. Let's try to disect the disassembly of this code and understand patterns. Here is the equivalent annotated assembly:

{% highlight asm %}
//void main() {
	push	ebp
	mov	ebp, esp
	sub	esp, 8

	mov	DWORD PTR [ebp-8], 0   		;<-- local variable index

$LN10@main:
	/*-do-while loop-*/
	mov	eax, DWORD PTR [ebp-8]
	inc eax	
	mov	DWORD PTR [ebp-8], eax      ; incremented and stored index
	
	...
	// other code within the do while
	...

	//while(index != 5);      condition checking
	mov	eax, DWORD PTR [ebp-8]
	test	eax, 5
	jne	SHORT $LN10@main

$Outside:
	xor	eax, eax
	mov	esp, ebp
	pop	ebp
	ret	0
	
{% endhighlight %}

The first few lines in the loop increment the index variable. This is a local variable, located on the stack at [ebp-8] in this frame.


{% highlight asm %}

	mov	eax, DWORD PTR [ebp-8]
	inc eax	
	mov	DWORD PTR [ebp-8], eax      ; incremented and stored index

{% endhighlight %}
	
	
As with the while loop, we see that microsoft compilers prefer using the **test, jmp/j<condition>** instructions whenever it evaluates conditions for a loop. In this case, the index variable is a local variable stored at [ebp-8]. This value is loaded into eax and then tested with 5. If the condition returns not equal, the loop continues.

{% highlight asm %}
$loopstart:
	/*-do-while loop-*/
	mov	eax, DWORD PTR [ebp-8]  ;
		
	...
	// other code within the do while
	...

	//while(index != 5);      condition checking
	mov	eax, DWORD PTR [ebp-8] 	;< -- index 
	test	eax, 5       		; checking if != 5
	jne	SHORT $loopstart		; if not equal, jump to loopstart
{% endhighlight %}

This is a useful pattern to note, but other compilers may do things differently, so do not assume this will be true always. That's it for now, we'll pick on another pattern in the next blog.
