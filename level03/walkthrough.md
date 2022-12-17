[back](../defense.md)

```
level02@OverRide:~$ su level03
Password: Hh74RPnuQ9sa5JAEXgNWCqz7sXGnh5J5M9KfPg3H
	RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
	Partial RELRO   Canary found      NX enabled    No PIE          No RPATH   No RUNPATH   /home/users/level03/level03
level03@OverRide:~$
```

Доступ открыт.

```
level03@OverRide:~$ cd /home/users/level03
level03@OverRide:~$ ls
```

На этом уровне программа просит ввода только пароля

```
level03@OverRide:~$ ./level03
	***********************************
	*		level03		**
	***********************************
Password:admin
	Invalid Password

gdb level03
(gdb) disas main
```

В фунции мэйн есть интересная последовательность: вызов функции scanf, загрузки числа в регистр и вызова функции тест.

```
0x080488c1 <+103>:	call   0x8048530 <__isoc99_scanf@plt>
0x080488c6 <+108>:	mov    0x1c(%esp),%eax
0x080488ca <+112>:	movl   $0x1337d00d,0x4(%esp)
0x080488d2 <+120>:	mov    %eax,(%esp)
0x080488d5 <+123>:	call   0x8048747 <test>
```

При переводе числа $0x1337d00d в десятичное, получаем 322424845.

Вероятно, это пароль.

Пробуем:

```
(gdb) q
level03@OverRide:~$ ./level03
***********************************
*		level03		**
***********************************
Password:322424845
Invalid Password
```

Не подходит.

Заходим в функцию test

```
gdb level03
(gdb) disas test
	0x08048755 <+14>:    sub    %eax,%ecx
	0x08048757 <+16>:    mov    %ecx,%eax
	0x08048759 <+18>:    mov    %eax,-0xc(%ebp)
	0x0804875c <+21>:    cmpl   $0x15,-0xc(%ebp)
```

Запустим прогу с остановом здесь и посмотрим содержимое регистров
```
(gdb) b test
	Breakpoint 1 at 0x804874d
(gdb) r
	Starting program: /home/users/level03/level03
	***********************************
	*		level03		**
	***********************************
	Password:42
	Breakpoint 1, 0x0804874d in test ()
(gdb) p $eax
	$1 = 42
(gdb) p $ecx
	$2 = 1
(gdb) p $esp
	$3 = (void *) 0xffffd670
(gdb) p $edp
	$4 = void
```

Ага. Из введенного значения вычитается число 15 и сравнивается с числом 21

```
(gdb) info functions
	0x080485f4  clear_stdin
	0x08048617  get_unum
	0x0804864f  prog_timeout
	0x08048660  decrypt
	0x08048747  test
	0x0804885a  main
```

В функции decrypt есть интересные строки:

```
(gdb) disas decrypt
	0x08048673 <+19>:	movl   $0x757c7d51,-0x1d(%ebp)
	0x0804867a <+26>:	movl   $0x67667360,-0x19(%ebp)
	0x08048681 <+33>:	movl   $0x7b66737e,-0x15(%ebp)
	0x08048688 <+40>:	movl   $0x33617c7d,-0x11(%ebp)
	,
	0x080486f2 <+146>:	mov    $0x80489c3,%eax
	0x080486f7 <+151>:	mov    $0x11,%ecx
	0x080486fc <+156>:	mov    %edx,%esi
	0x080486fe <+158>:	mov    %eax,%edi
	0x08048700 <+160>:	repz cmpsb %es:(%edi),%ds:(%esi)
	и
	0x08048713 <+179>:	jne    0x8048723 <decrypt+195>
	0x08048715 <+181>:	movl   $0x80489d4,(%esp)
	0x0804871c <+188>:	call   0x80484e0 <system@plt>
```

В первой пачке кода явно зашито какое-то число.

переведем его в десятичный код:

```
0x757c7d51 = u|}Q
0x67667360 = gfs`
0x7b66737e = {fs~
0x33617c7d = 3a|}
```

получаем последовательность = "Q}|u`sfg~sf{}|a3"

Судя по коду нужно применить функцию XOR.

```
"Q}|u`sfg~sf{}|a3" == "Congratulations!"
```

Конвертируем обе строки в binary

```
01010001 01111101 01111100 01110101 01100000 01110011 01100110 01100111 01111110 01110011 01100110 01111011 01111101 01111100 01100001 00110011
01000011 01101111 01101110 01100111 01110010 01100001 01110100 01110101 01101100 01100001 01110100 01101001 01101111 01101110 01110011 00100001
```

Заметно, что отличаются только 2 и 5 биты -> key =  "00010010", или 18

уменьшим полученное ранее число на этот ключ: "322424845 - 18", новый ключ-пароль "322424827"

```
level03@OverRide:~$ ./level03
whoami
	level04
cd /home/users/level04
ls -la
cat .pass
	kgv3tkEb9h2mLkRsPkXRfc2mHbjMxQzvb2FrgKkf
Ctrl+C
su level04
	Password: kgv3tkEb9h2mLkRsPkXRfc2mHbjMxQzvb2FrgKkf
	RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
	Partial RELRO   No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/users/level04/level04
```

Мы получили доступ к следующему уровню.

[back](../defense.md)
