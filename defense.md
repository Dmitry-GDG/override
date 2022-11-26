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

We find a binary ```level00``` with owner ```level01``` and SUID.

Lets try to gnb ```level00```:

```
gdb level00 
(gdb) disass main
```

```main()``` calls ```scanf()``` to read the password from stdin


```
0x080484d3 <+63>:	lea    0x1c(%esp),%edx
0x080484d7 <+67>:	mov    %edx,0x4(%esp) //address to store password
0x080484db <+71>:	mov    %eax,(%esp) // load arg 2 - address to store password
0x080484de <+74>:	call   0x80483d0 <__isoc99_scanf@plt> // scanf() reads stdin
0x080484e3 <+79>:	mov    0x1c(%esp),%eax //load scanf() return (password)
0x080484e7 <+83>:	cmp    $0x149c,%eax //compare password = 5276
```

then compares given password with value ```0x149c``` (5276 in decimal). (https://binary2hex.ru/numberconverter.html?id=1625627)


```
./level00 
Password:5276
```

The 5276 was accepted!

```
whoami
```

```
ls -la
```

```
cd /
ls -la


Sources:
- https://beta.hackndo.com/retour-a-la-libc/

# LEVEL 01


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
