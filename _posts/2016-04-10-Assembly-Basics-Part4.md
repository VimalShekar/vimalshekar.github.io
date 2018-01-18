---
layout: post
category: Reverse-Engg
title: Assembly Language Basics - Part 4
date: 2016-04-10
---

# Anatomy of a Function

Reverse engineering is an art as much as it is a skill. To learn reverse engineering you should have strong knowledge of at least one programming language and should know the inner workings of at least one compiler. If you don't, then the knowledge you gain will always have gaps. That statement may dishearten you at first, but you will realize its importance when you take up revering projects that are more complex in nature.

Anyways, everyone has to start from somewhere, right ? My journey of learning reverse engineering and assembly, started with learning the anatomy of a function.


A Function or Procedure is a programing construct which consists of a set of statements that  perform a task. Every modern programming language has such a construct. Functions allow us to modularize tasks, so that it can be present in a single location in memory and can be called whenever and wherever necessary. 

Let's look at the anatomy of a function, taking a simple function written in C as an example:

    {% highlight cpp %}
	//-- This function just takes two integers and returns the difference
	int Diff(int x, int y)  =>The head
	{ //=> the body starts
		int R;
		R = x - y;
		return R;
	} //=> the body ends
    {% endhighlight %}

Without going into the details, a basic function in C, consists of a head and a body. The head of a function defines its name, the arguments passed and its return value. The body of a function has the set of statements that the function performs.

Now let's look at this function's assembly listing:

    {% highlight asm %}
	//-- Equivalent assembly
	push ebp
	mov ebp,esp
	sub esp, 04h
	push ecx
	mov eax,dword ptr [ebp+8]
	sub eax,dword ptr [ebp+0Ch]
	mov dword ptr [ebp-4],eax
	mov eax,dword ptr [ebp-4]
	mov esp,ebp
	pop ebp
	ret 
    {% endhighlight %}

The assembly listing of the function is very much different from the C code, there is no head which tells what the arguments are and what the return value is. The above listing is from a binary compiled for x86 platform. The disassembly of every function only has three parts:

* prolog
{% highlight asm %}
	push ebp
	mov ebp,esp
	sub esp, 04h
{% endhighlight %}

The Prolog does the following:
>	* Saves the frame pointer (EBP) of the caller to stack.
>	* Sets up the frame for the current function, including space for local variables.
>	* Optionally, it also saves any non-volatile registers from the previous frame


* body of the function
{% highlight asm %}
	push ecx
	mov eax,dword ptr [ebp+8]
	sub eax,dword ptr [ebp+0Ch]
	mov dword ptr [ebp-4],eax
	mov eax,dword ptr [ebp-4]
{% endhighlight %}	
The body of the function contains the assembly equivalent of the code written in the higher level language.


* epilog
{% highlight asm %}
	mov esp,ebp
	pop ebp
	ret 
{% endhighlight %}

The Epilog reverses the effect of the prolog, it performs the following:
>   * Restore any saved registers
>   * De-allocates the space allocated for variables
>   * Gets the caller's frame pointer and puts it back into the EBP register.


Some of these terms might confuse you now, but we'll come back to this in a later post. 
