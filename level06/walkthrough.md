[back](../defense.md)

Запустим прогу:

```
level06@OverRide:~$ ./level06
***********************************
*		level06		  *
***********************************
-> Enter Login: Admin
***********************************
***** NEW ACCOUNT DETECTED ********
***********************************
-> Enter Serial: Admin
level06@OverRide:~$
```

Запустим gdb

```
(gdb) disas main
Dump of assembler code for function main:
   0x08048879 <+0>:	push   %ebp
   0x0804887a <+1>:	mov    %esp,%ebp
   0x0804887c <+3>:	and    $0xfffffff0,%esp
   0x0804887f <+6>:	sub    $0x50,%esp
   0x08048882 <+9>:	mov    0xc(%ebp),%eax
   0x08048885 <+12>:	mov    %eax,0x1c(%esp)
   0x08048889 <+16>:	mov    %gs:0x14,%eax
   0x0804888f <+22>:	mov    %eax,0x4c(%esp)
   0x08048893 <+26>:	xor    %eax,%eax
   0x08048895 <+28>:	push   %eax
   0x08048896 <+29>:	xor    %eax,%eax
   0x08048898 <+31>:	je     0x804889d <main+36>
   0x0804889a <+33>:	add    $0x4,%esp
   0x0804889d <+36>:	pop    %eax
   0x0804889e <+37>:	movl   $0x8048ad4,(%esp)
   0x080488a5 <+44>:	call   0x8048590 <puts@plt>
   0x080488aa <+49>:	movl   $0x8048af8,(%esp)
   0x080488b1 <+56>:	call   0x8048590 <puts@plt>
   0x080488b6 <+61>:	movl   $0x8048ad4,(%esp)
   0x080488bd <+68>:	call   0x8048590 <puts@plt>
   0x080488c2 <+73>:	mov    $0x8048b08,%eax
   0x080488c7 <+78>:	mov    %eax,(%esp)
   0x080488ca <+81>:	call   0x8048510 <printf@plt>
   0x080488cf <+86>:	mov    0x804a060,%eax
   0x080488d4 <+91>:	mov    %eax,0x8(%esp)
   0x080488d8 <+95>:	movl   $0x20,0x4(%esp)
   0x080488e0 <+103>:	lea    0x2c(%esp),%eax
   0x080488e4 <+107>:	mov    %eax,(%esp)
   0x080488e7 <+110>:	call   0x8048550 <fgets@plt>
   0x080488ec <+115>:	movl   $0x8048ad4,(%esp)
   0x080488f3 <+122>:	call   0x8048590 <puts@plt>
   0x080488f8 <+127>:	movl   $0x8048b1c,(%esp)
   0x080488ff <+134>:	call   0x8048590 <puts@plt>
   0x08048904 <+139>:	movl   $0x8048ad4,(%esp)
   0x0804890b <+146>:	call   0x8048590 <puts@plt>
   0x08048910 <+151>:	mov    $0x8048b40,%eax
   0x08048915 <+156>:	mov    %eax,(%esp)
   0x08048918 <+159>:	call   0x8048510 <printf@plt>
   0x0804891d <+164>:	mov    $0x8048a60,%eax
   0x08048922 <+169>:	lea    0x28(%esp),%edx
   0x08048926 <+173>:	mov    %edx,0x4(%esp)
   0x0804892a <+177>:	mov    %eax,(%esp)
   0x0804892d <+180>:	call   0x80485e0 <__isoc99_scanf@plt>
   0x08048932 <+185>:	mov    0x28(%esp),%eax
   0x08048936 <+189>:	mov    %eax,0x4(%esp)
   0x0804893a <+193>:	lea    0x2c(%esp),%eax
   0x0804893e <+197>:	mov    %eax,(%esp)
   0x08048941 <+200>:	call   0x8048748 <auth>
   0x08048946 <+205>:	test   %eax,%eax
   0x08048948 <+207>:	jne    0x8048969 <main+240>
   0x0804894a <+209>:	movl   $0x8048b52,(%esp)
   0x08048951 <+216>:	call   0x8048590 <puts@plt>
   0x08048956 <+221>:	movl   $0x8048b61,(%esp)
   0x0804895d <+228>:	call   0x80485a0 <system@plt>
``` 

Основная часть программы сначала вызывает printf() и fgets().

fgets() получает логин и помесчает его в буфер char[32]. 

Далее следует вызов scanf, чтобы получить серийный номер.

Затем основной вызов функции auth с логином и серийным номером для валидации. 

Имеется также вызов system ("/bin/sh")

Нам нужно пройти валидациюauth()

В функции auth есть алгоритм хеширования для вычисления серийника с логином (расчёт с каждым символом логина). 

Мы изменим его, чтобы сделать программу хеширования для генерации серийных номеров, или прочитаем память, когда функция auth сравнивает сгенерированный серийный номер с вводом!

```
0x0804878c <+68>:	mov    $0x1,%eax
0x08048791 <+73>:	jmp    0x8048877 <auth+303>
0x08048796 <+78>:	movl   $0x0,0xc(%esp)
0x0804879e <+86>:	movl   $0x1,0x8(%esp)
0x080487a6 <+94>:	movl   $0x0,0x4(%esp)
0x080487ae <+102>:	movl   $0x0,(%esp)
0x080487b5 <+109>:	call   0x80485f0 <ptrace@plt>
0x080487ba <+114>:	cmp    $0xffffffff,%eax         -> сравнение имени 

0x0804885b <+275>:	mov    -0x14(%ebp),%eax
0x0804885e <+278>:	cmp    -0xc(%ebp),%eax
0x08048861 <+281>:	jl     0x804880f <auth+199>
0x08048863 <+283>:	mov    0xc(%ebp),%eax
0x08048866 <+286>:	cmp    -0x10(%ebp),%eax          -> сравнение хэш и серийника

(gdb) b *0x080487ba
Breakpoint 1 at 0x80487ba
(gdb) b *0x08048866
Breakpoint 2 at 0x8048866
(gdb) r
Starting program: /home/users/level06/level06
***********************************
*		level06		  *
***********************************
-> Enter Login: adminer
***********************************
***** NEW ACCOUNT DETECTED ********
***********************************
-> Enter Serial: 1234
```

Для взлома делаем две точки останова. 

Одну после сверки логина

```
Breakpoint 1, 0x080487ba in auth ()
```

Смотрим, что в регистре

```
(gdb) p $eax
$1 = -1
```

пробуем поменять на 0

```
(gdb) set $eax=0
(gdb) c
Continuing.
```

На второй точке останова смотрим серйник.

```
Breakpoint 2, 0x08048866 in auth ()
(gdb) x/d $ebp-0x10
0xffffd5c8:	6231008
```

Пробуем подставить логин и серийник в прогу

```
level06@OverRide:~$ ./level06
***********************************
*		level06		  *
***********************************
-> Enter Login: adminer
***********************************
***** NEW ACCOUNT DETECTED ********
***********************************
-> Enter Serial: 6231008
Authenticated!
$ whoami
level07
$ cat /home/users/level07/.pass
GbcPDRgsFK77LNnnuh7QyFYA2942Gp8yKj9KrWD8
```

уровень пройден

[back](../defense.md)
