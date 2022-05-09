
The code of measuring loop ("Signal to High Prio Thread" bench) has been improved to minimize memory accesses.
</br>(Function low_prio_signal_early_proc_fn() in [apps/signal/src/main.c](https://github.com/malus-brandywine/sel4bench/blob/master/apps/signal/src/main.c))


Current version 05.09.2022 (with "min", "max" and bitmask allocated in stack removed) uses only register variables.
The only inevitable "load" instruction is used to read a volatile variable updated by other thread.

There are asm code of the loop generated for 4 processors:

Cortex-A9:
   ```
   10718:	e1a03fac 	lsr	r3, ip, #31
   1071c:	ee19af1d 	mrc	15, 0, sl, cr9, cr13, {0}
   10720:	e3e07004 	mvn	r7, #4
   10724:	e1a00008 	mov	r0, r8
   10728:	ef000000 	svc	0x00000000
   1072c:	e5992000 	ldr	r2, [r9]
   10730:	e24cc001 	sub	ip, ip, #1
   10734:	e37c0065 	cmn	ip, #101	; 0x65
   10738:	e042200e 	sub	r2, r2, lr
   1073c:	e042200a 	sub	r2, r2, sl
   10740:	e0030293 	mul	r3, r3, r2
   10744:	e0844003 	add	r4, r4, r3
   10748:	e0211393 	mla	r1, r3, r3, r1
   1074c:	1afffff1 	bne	10718 <low_prio_signal_early_proc_fn+0x4c>
```
Cortex-A57
   ```
  400890:	d37ffc24 	lsr	x4, x1, #63
  400894:	d53b9d05 	mrs	x5, pmccntr_el0
  400898:	92800087 	mov	x7, #0xfffffffffffffffb    	// #-5
  40089c:	aa1303e0 	mov	x0, x19
  4008a0:	d4000001 	svc	#0x0
  4008a4:	f9400280 	ldr	x0, [x20]
  4008a8:	d1000421 	sub	x1, x1, #0x1
  4008ac:	b101943f 	cmn	x1, #0x65
  4008b0:	cb060000 	sub	x0, x0, x6
  4008b4:	cb050000 	sub	x0, x0, x5
  4008b8:	9b047c00 	mul	x0, x0, x4
  4008bc:	8b000063 	add	x3, x3, x0
  4008c0:	9b000802 	madd	x2, x0, x0, x2
  4008c4:	54fffe61 	b.ne	400890 <low_prio_signal_early_proc_fn+0x58>  // b.any
   ```
U54
   ```
   10520:	03f75593          	srli	a1,a4,0x3f
   10524:	c0002873          	rdcycle	a6
   10528:	58ed                	li	a7,-5
   1052a:	8522                	mv	a0,s0
   1052c:	00000073          	ecall
   10530:	609c                	ld	a5,0(s1)
   10532:	177d                	addi	a4,a4,-1
   10534:	41c787b3          	sub	a5,a5,t3
   10538:	410787b3          	sub	a5,a5,a6
   1053c:	02b787b3          	mul	a5,a5,a1
   10540:	02f785b3          	mul	a1,a5,a5
   10544:	963e                	add	a2,a2,a5
   10546:	96ae                	add	a3,a3,a1
   10548:	fc671ce3          	bne	a4,t1,10520 <low_prio_signal_early_proc_fn+0x46>
   ```
i7-4770
   ```
  401738:	b8 00 00 00 00       	mov    $0x0,%eax
  40173d:	b9 00 00 00 00       	mov    $0x0,%ecx
  401742:	0f a2                	cpuid  
  401744:	0f 31                	rdtsc  
  401746:	89 d6                	mov    %edx,%esi
  401748:	41 89 c1             	mov    %eax,%r9d
  40174b:	b8 00 00 00 00       	mov    $0x0,%eax
  401750:	b9 00 00 00 00       	mov    $0x0,%ecx
  401755:	0f a2                	cpuid  
  401757:	48 c7 c2 fb ff ff ff 	mov    $0xfffffffffffffffb,%rdx
  40175e:	4c 89 ef             	mov    %r13,%rdi
  401761:	48 89 e3             	mov    %rsp,%rbx
  401764:	0f 05                	syscall 
  401766:	48 89 dc             	mov    %rbx,%rsp
  401769:	49 8b 07             	mov    (%r15),%rax
  40176c:	48 c1 e6 20          	shl    $0x20,%rsi
  401770:	45 89 c9             	mov    %r9d,%r9d
  401773:	4c 89 c2             	mov    %r8,%rdx
  401776:	49 09 f1             	or     %rsi,%r9
  401779:	48 c1 ea 3f          	shr    $0x3f,%rdx
  40177d:	49 ff c8             	dec    %r8
  401780:	4c 29 e0             	sub    %r12,%rax
  401783:	4c 29 c8             	sub    %r9,%rax
  401786:	48 0f af c2          	imul   %rdx,%rax
  40178a:	49 01 c2             	add    %rax,%r10
  40178d:	48 0f af c0          	imul   %rax,%rax
  401791:	49 01 c6             	add    %rax,%r14
  401794:	49 83 f8 9b          	cmp    $0xffffffffffffff9b,%r8
  401798:	75 9e                	jne    401738 <low_prio_signal_early_proc_fn+0x68>
   ```
   
