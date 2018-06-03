---
layout: post
title: Assembly Basics - Part 12
date: 2017-06-04
category: Reverse-Engg
---

# Loops and conditional statements translated into assembly

In this blog, we will look at how conditions statements and looping constructs in higher level languages are translated into assembly. For the purpose of this example, I have used the below code. 

{% highlight cpp %}
#include<stdio.h>

void main() {

	int index = 0;
	
	/*-while loop-*/
	while(1)
	{
		
		index++;
		
		/*-if-else-if ladder approach-*/
		if(index == 2)
		{
			printf("\nIndex is 2, hurrah! ");
		}
		
		else if(index == 5)
		{
			printf("\nIndex is now 5, half way there!");
		}
		
		else if(index == 9)
		{
			printf(" \nOnce More Baby!");
		}
		
		else if(index > 10)
		{
			index = 0;
			break;
		}
		
		else
		{
			printf("\n index = %d", index);
		}
	}
	

}

{% endhighlight %}

In the above code sample, we have a while loop and multiple "if" conditions within this loop. Let's try to disect the disassembly of this code and understand patterns. Here is the equivalent annotated assembly:

{% highlight asm %}

/*-while loop-*/
//while(1)

	mov	eax, 1
	test	eax, eax
	je	SHORT $LN11@main

	mov	ecx, DWORD PTR _index$[ebp]
	add	ecx, 1                                                          ; index++
	mov	DWORD PTR _index$[ebp], ecx

/*-if-else-if ladder approach-*/
	cmp	DWORD PTR _index$[ebp], 2		; else if(index==2)
	jne	SHORT $LN8@main
	push	OFFSET $SG2939
	call	_printf
	add	esp, 4
	jmp	SHORT $LN7@main                                 ; break
	cmp	DWORD PTR _index$[ebp], 5		; else if(index==5)
	jne	SHORT $LN6@main
	push	OFFSET $SG2942
	call	_printf
	add	esp, 4
	jmp	SHORT $LN7@main                                 ; break
	cmp	DWORD PTR _index$[ebp], 9		; else if(index==9)
	jne	SHORT $LN4@main
	push	OFFSET $SG2945
	call	_printf
	add	esp, 4
	jmp	SHORT $LN7@main                                 ; break
	cmp	DWORD PTR _index$[ebp], 10		; else if(index>10)
	jle	SHORT $LN2@main
	mov	DWORD PTR _index$[ebp], 0
	jmp	SHORT $LN11@main
	jmp	SHORT $LN7@main                                 ; break
	mov	edx, DWORD PTR _index$[ebp]
	push	edx
	push	OFFSET $SG2949
	call	_printf
	add	esp, 8
	jmp	SHORT $LN10@main                       ; end of while loop
	
	//End of main
	xor	eax, eax
	mov	esp, ebp
	pop	ebp
	ret	0
	
{% endhighlight %}


One pattern I found with microsoft compilers is that it tends to use the **cmp, jmp/j<condition>** instructions whenever there is an **"if(condition)"** and the **test, jmp/j<condition>** instructions whenever it evaluates conditions for a loop. 

Here's our example, our **while(true)** loop was translated to:
{% highlight asm %}
	test	eax, eax
	je	SHORT $LN11@main
{% endhighlight %}

Each of our **else if** conditions were translated to:
{% highlight asm %}
    cmp	DWORD PTR _index$[ebp], 10		; else if(index>10)
    jle	SHORT $LN2@main
{% endhighlight %}

This is a useful pattern to note, but other compilers may do things differently, so do not assume this will be true always.

Another pattern that you can observe is that whenever the code has an **else if** ladder structure, the will be a jump statement that takes you to the end of the ladder. This is a useful pattern to note.

{% highlight asm %}

/*-if-else-if ladder approach-*/
	cmp	DWORD PTR _index$[ebp], 2		; if(index==2)
	jne	SHORT $LN8@main
    ... //code
	jmp	SHORT $LN7@main                 ; Jump to code after all the else if/else statements
	cmp	DWORD PTR _index$[ebp], 5		; else if(index==5)
	jne	SHORT $LN6@main
    ... //code
	jmp	SHORT $LN7@main                 ; Jump to code after all the else if/else statements
	cmp	DWORD PTR _index$[ebp], 9		; else if(index==9)
	jne	SHORT $LN4@main
    ... // code
	jmp	SHORT $LN7@main            ; Jump to code after all the else if/else statements
	cmp	DWORD PTR _index$[ebp], 10		; next else if(index>10)
	jle	SHORT $LN2@main
	... //code
	jmp	SHORT $LN11@main
{% endhighlight %}

That's it for now, we'll pick on another pattern in the next blog.
