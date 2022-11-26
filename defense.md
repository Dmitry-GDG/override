| [README](README.md) | [subject](subject_ru.md) | [How to start a web server](howTo.md) | defense |
|-|-|-|-|

# Defense the project

# LEVEL 00

From subject: "Then, you will be able to connect using the following couple of login:password: level00:level00."

In terminal:

```
ssh level00@192.168.56.4 -p 4242
>> yes
>> password: level00
```

<img width="930" alt="Screen Shot 2022-11-26 at 13 20 54" src="https://user-images.githubusercontent.com/84193980/204084033-b951cae2-1a89-4afd-b896-df757f926dad.png">

```
ls -la
```

<img width="930" alt="Screen Shot 2022-11-26 at 13 27 34" src="https://user-images.githubusercontent.com/84193980/204084308-e3e81a1f-ec1e-465b-bc8e-02f1ae96c0d2.png">

```
./level00
```

<img width="930" alt="Screen Shot 2022-11-26 at 13 30 12" src="https://user-images.githubusercontent.com/84193980/204084321-0029400b-d4eb-45f5-83a8-14bcf2e9b1a5.png">

We need a correct password

We find a binary ```level00``` with owner ```level01``` and SUID (A file with SUID always executes as the user who owns the file).

Lets try to gdb ```level00``` (The purpose of a debugger such as GDB is to allow you to see what is going on "inside" another program while it executes -- or what  another program was doing):

```
gdb level00 
(gdb) disass main
```

<img width="586" alt="Screen Shot 2022-11-26 at 15 29 33" src="https://user-images.githubusercontent.com/84193980/204092423-1d4b2e94-1cac-4ed2-a31b-f95c6c6208ba.png">

```main()``` calls ```scanf()``` to read the password from stdin


```
0x080484d3 <+63>:	lea    0x1c(%esp),%edx
0x080484d7 <+67>:	mov    %edx,0x4(%esp)                 // address to store password
0x080484db <+71>:	mov    %eax,(%esp)                    // load arg 2 - address to store password
0x080484de <+74>:	call   0x80483d0 <__isoc99_scanf@plt> // scanf() reads stdin
0x080484e3 <+79>:	mov    0x1c(%esp),%eax                // load scanf() return (password)
0x080484e7 <+83>:	cmp    $0x149c,%eax                   // compare password = 5276
```

then compares given password with value ```0x149c``` (5276 in decimal). (https://binary2hex.ru/numberconverter.html?id=1625627)


```
q
./level00 
Password:5276
```

<img width="581" alt="Screen Shot 2022-11-26 at 15 49 44" src="https://user-images.githubusercontent.com/84193980/204092462-ad51f3fd-67ab-4532-871c-0056f41a8eaa.png">

The 5276 was accepted!

```
whoami
```

<img width="584" alt="Screen Shot 2022-11-26 at 15 51 46" src="https://user-images.githubusercontent.com/84193980/204092495-ec9c1049-5741-4887-9f2b-f41d5b33d2ec.png">

Try to find the password for level01

```
ls -la
```

<img width="584" alt="Screen Shot 2022-11-26 at 15 53 03" src="https://user-images.githubusercontent.com/84193980/204092556-b536aeb7-0364-4b13-ab32-5d7cba50aa2f.png">

```
cd /home/users/level01
ls -la
cat .pass
```

<img width="584" alt="Screen Shot 2022-11-26 at 17 26 43" src="https://user-images.githubusercontent.com/84193980/204094116-e159d88d-68a5-429f-829c-906c27cf708f.png">

Password: uSq2ehEGT6c9S24zbshexZQBXUGrncxn5sD5QfGL

```
exit
su level01
Password: uSq2ehEGT6c9S24zbshexZQBXUGrncxn5sD5QfGL
```

<img width="851" alt="Screen Shot 2022-11-26 at 17 32 05" src="https://user-images.githubusercontent.com/84193980/204094109-83d52c04-5c54-4b29-a529-0639e7d5f3f0.png">

Sources:
- https://beta.hackndo.com/retour-a-la-libc/

# LEVEL 01

```
ls -la
```



```
./level01
```

We need a correct password

We find a binary ```level01``` with owner ```users``` and SUID (A file with SUID always executes as the user who owns the file).

Lets try to gdb ```level01``` (The purpose of a debugger such as GDB is to allow you to see what is going on "inside" another program while it executes -- or what  another program was doing):

```
gdb level01 
(gdb) disass main
```

# LEVEL 02


# LEVEL 03


# LEVEL 04


# LEVEL 05


# LEVEL 06


# LEVEL 07


# LEVEL 08


# LEVEL 09

| [README](README.md) | [subject](subject_ru.md) | [How to start a web server](howTo.md) | defense |
|-|-|-|-|
