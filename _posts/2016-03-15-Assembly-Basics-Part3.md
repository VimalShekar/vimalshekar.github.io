---
layout: post
category: Reverse-Engg
title: Assembly Language Basics - Part 3
date: 2016-03-15
---


Instructions are classified based on where the data for the source operand of the instruction originates, and where the destination is. The data for a source operand can be located in:
* The instruction itself (an immediate operand)
* A register
* A memory location
* An I/O port

When an instruction returns data to a destination operand, it can be returned to:
* A register
* A memory location
* An I/O port

Based on the source and destination, the following are some of the most common addressing modes available:

|Instruction | Example | Description |
|---------|--------|------|
|Implicit | CLO -- clear overflow flag | The operand is not specified, the instruction implicitely knows where the operands are. |
| Register | CALL EDI  | The operands are registers specified in the instruction. |
| Immediate | MOV EBX, 0x123  | One of the operands is an immediate value specified in the instruction |
| Direct (Displacement-Only) | MOV EBX, [0x123]  | The operand is an address specified in the instruction. |
| Register Indirect | MOV EAX, [EBP] | One of the operands is a  memory location pointed to by a register |
| Based or Indirect | MOV EAX, [EBP + 8] | Address is obtained as a sum of a base register and an offset  |
| Indexed | EAX, 0x1234S[ESI] | Address is obtained as a sum of an index register and an offset |
| Based Plus Indexed | MOV EAX, [EBP][ESI] |  Address is obtained as a sum of an index register and an offset | 
| Based Plus Indexed Plus Displacement | | combination of all of the above |

Based on the type of operation, instructions can be classified into:

**Data Transfer Instructions:**
The data transfer instructions move data between memory and the general-purpose and segment registers. 

| Instruction | Description |
|------------|-------------|
| CMOVE/CMOVZ	|Conditional move if equal/Conditional move if zero
| CMOVNE/CMOVNZ	| Conditional move if not equal/Conditional move if not zero
| CMOVA/CMOVNBE	| Conditional move if above/Conditional move if not below or equal
| CMOVAE/CMOVNB	| Conditional move if above or equal/Conditional move if not below
| CMOVB/CMOVNAE	| Conditional move if below/Conditional move if not above or equal
| CMOVBE/CMOVNA	| Conditional move if below or equal/Conditional move if not above
| CMOVG/CMOVNLE	| Conditional move if greater/Conditional move if not less or equal
| CMOVGE/CMOVNL	| Conditional move if greater or equal/Conditional move if not less
| CMOVL/CMOVNGE	| Conditional move if less/Conditional move if not greater or equal
| CMOVLE/CMOVNG	| Conditional move if less or equal/Conditional move if not greater
| CMOVC	| Conditional move if carry
| CMOVNC | Conditional move if not carry
| CMOVO	 | Conditional move if overflow
| CMOVNO	| Conditional move if not overflow
| CMOVS	| Conditional move if sign (negative)
| CMOVNS	| Conditional move if not sign (non-negative)
| CMOVP/CMOVPE	| Conditional move if parity/Conditional move if parity even
| CMOVNP/CMOVPO	| Conditional move if not parity/Conditional move if parity odd
| XCHG 	| Exchange
| BSWAP	| Byte swap
| XADD	| Exchange and add
| CMPXCHG	| Compare and exchange
| CMPXCHG8B	| Compare and exchange 8 bytes
| PUSH	| Push onto stack
| POP	| Pop off of stack
| PUSHA/PUSHAD	| Push general-purpose registers onto stack
| POPA/POPAD	| Pop general-purpose registers from stack
| CWD/CDQ	| Convert word to doubleword/Convert doubleword to quadword
| CBW/CWDE	| Convert byte to word/Convert word to doubleword in EAX register
| MOVSX	| Move and sign extend
| MOVZX	| Move and zero extend

**Arithmetic and Logical Instructions:**
The binary arithmetic instructions perform basic binary integer computations on byte, word, and double word integers located in memory and/or the general purpose registers. The decimal arithmetic instructions perform decimal arithmetic on binary coded decimal (BCD) data. The logical instructions perform basic AND, OR, XOR, and NOT logical operations on byte, word, and double word values. The shift and rotate instructions shift and rotate the bits in word and double word operands.

| Instruction | Description |
|------------|-------------|
| AAA	 | ASCII adjust after addition
| AAD	 | ASCII adjust before division
| AAM	 | ASCII adjust after multiplication
| AAS	 | ASCII adjust after subtraction
| ADC	 | Add with carry
| ADCX	 | Unsigned integer add with carry
| ADD	 | Integer add
| ADOX	 | Unsigned integer add with overflow
| AND	 | Perform bitwise logical AND
| CMP	 | Compare
| DAA	 | Decimal adjust after addition
| DAS	 | Decimal adjust after subtraction
| DEC	 | Decrement
| DIV	 | Unsigned divide
| IDIV	 | Signed divide
| IMUL	 | Signed multiply
| INC	 | Increment
| MUL	 | Unsigned multiply
| NEG	 | Negate
| NOT	 | Perform bitwise logical NOT
| OR	 | Perform bitwise logical OR
| ROL	 | Rotate left
| ROR	 | Rotate right
| SAL/SHL |	 Shift arithmetic left/Shift logical left
| SAR	 | Shift arithmetic right
| SBB	 | Subtract with borrow
| SHLD	 | Shift left double
| SHR	 | Shift logical right
| SHRD	 | Shift right double
| SUB	 | Subtract
| XOR	 | Perform bitwise logical exclusive OR


Bit and Byte Instructions
Bit instructions test and modify individual bits in word and doubleword operands. Byte instructions set the value of
a byte operand to indicate the status of flags in the EFLAGS register.

| Instruction | Description |
|------------|-------------|
|BT	 | Bit test
|BTS	 | Bit test and set
|BTR	 | Bit test and reset
|BTC	 | Bit test and complement
|BSF	 | Bit scan forward
|BSR	 | Bit scan reverse
|SETE/SETZ	 | Set byte if equal/Set byte if zero
|SETNE/SETNZ	 | Set byte if not equal/Set byte if not zero
|SETA/SETNBE	 | Set byte if above/Set byte if not below or equal
|SETAE/SETNB/SETNC	| Set byte if above or equal/Set byte if not below/Set byte if not carry
|SETB/SETNAE/SETC	| Set byte if below/Set byte if not above or equal/Set byte if carry
|SETBE/SETNA	| Set byte if below or equal/Set byte if not above
|SETG/SETNLE	| Set byte if greater/Set byte if not less or equal
|SETGE/SETNL	| Set byte if greater or equal/Set byte if not less
|SETL/SETNGE	| Set byte if less/Set byte if not greater or equal
|SETLE/SETNG	| Set byte if less or equal/Set byte if not greater
|SETS	| Set byte if sign (negative)
|SETNS	| Set byte if not sign (non-negative)
|SETO	| Set byte if overflow
|SETNO	| Set byte if not overflow
|SETPE/SETP	| Set byte if parity even/Set byte if parity
|SETPO/SETNP	| Set byte if parity odd/Set byte if not parity
|TEST	| Logical compare


Control Transfer Instructions
The control transfer instructions provide jump, conditional jump, loop, and call and return operations to control
program flow.

| Instruction | Description |
|------------|-------------|
| JMP	| Jump|
| JE/JZ	| Jump if equal/Jump if zero|
| JNE/JNZ	| Jump if not equal/Jump if not zero|
| JA/JNBE	| Jump if above/Jump if not below or equal|
| JAE/JNB	| Jump if above or equal/Jump if not below|
| JB/JNAE	| Jump if below/Jump if not above or equal|
| JBE/JNA	| Jump if below or equal/Jump if not above|
| JG/JNLE	| Jump if greater/Jump if not less or equal|
| JGE/JNL	| Jump if greater or equal/Jump if not less|
| JL/JNGE	| Jump if less/Jump if not greater or equal|
| JLE/JNG	| Jump if less or equal/Jump if not greater|
| JC	| Jump if carry|
| JNC	| Jump if not carry|
| JO	| Jump if overflow|
| JNO	| Jump if not overflow|
| JS	| Jump if sign (negative)|
| JNS	| Jump if not sign (non-negative)|
| JPO/JNP	| Jump if parity odd/Jump if not parity|
| JPE/JP	| Jump if parity even/Jump if parity|
| JCXZ/JECXZ	| Jump register CX zero/Jump register ECX zero|
| LOOP	| Loop with ECX counter|
| LOOPZ/LOOPE	| Loop with ECX and zero/Loop with ECX and equal|
| LOOPNZ/LOOPNE	| Loop with ECX and not zero/Loop with ECX and not equal|
| CALL	| Call procedure|
| RET	| Return|
| IRET	| Return from interrupt|
| INT	| Software interrupt|
| INTO	| Interrupt on overflow|
| BOUND	| Detect value out of range|
| ENTER	| High-level procedure entry|
| LEAVE	| High-level procedure exit|


String Instructions
The string instructions operate on strings of bytes, allowing them to be moved to and from memory. .

| Instruction | Description |
|------------|-------------|
| MOVS/MOVSB 	 | Move string/Move byte string |
| MOVS/MOVSW 	 | Move string/Move word string |
| MOVS/MOVSD 	 | Move string/Move doubleword string |
| CMPS/CMPSB 	 | Compare string/Compare byte string |
| CMPS/CMPSW 	 | Compare string/Compare word string |
| CMPS/CMPSD 	 | Compare string/Compare doubleword string |
| SCAS/SCASB 	 | Scan string/Scan byte string |
| SCAS/SCASW 	 | Scan string/Scan word string |
| SCAS/SCASD 	 | Scan string/Scan doubleword string |
| LODS/LODSB 	 | Load string/Load byte string |
| LODS/LODSW 	 | Load string/Load word string |
| LODS/LODSD 	 | Load string/Load doubleword string |
| STOS/STOSB 	 | Store string/Store byte string |
| STOS/STOSW 	 | Store string/Store word string |
| STOS/STOSD 	 | Store string/Store doubleword string |
| REP | Repeat while ECX not zero |
| REPE/REPZ 	 | Repeat while equal/Repeat while zero |
| REPNE/REPNZ 	 | Repeat while not equal/Repeat while not zero |


I/O Instructions
These instructions move data between the processor’s I/O ports and a register or memory.

| Instruction | Description |
|------------|-------------|
| IN	 | Read from a port
| OUT	 | Write to a port
| INS/INSB	| Input string from port/Input byte string from port
| INS/INSW	| Input string from port/Input word string from port
| INS/INSD	| Input string from port/Input doubleword string from port
| OUTS/OUTSB	| Output string to port/Output byte string to port
| OUTS/OUTSW	| Output string to port/Output word string to port
| OUTS/OUTSD	| Output string to port/Output doubleword string to port


Note:
This is not the complete set! For a complete list of addressing modes and examples, refer to the intel manual available at:
http://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-manual-325462.pdf 



