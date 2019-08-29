Web: Super Efficient Key Store
-----------------------------

Challenge description:

```
Check out this super cool Key store I made, it uses UDP to be super eficient and remove all that pesky tcp overhead. Mine is running on challenges.openctf.cat but I blocked the port so you will have to run your own. Feel free to remove that bit about "flag" some guy asked me to add it for his special project.
```

We were given the source code a a key/value store that is written in Go. There is a main thread running on TCP port 5000. For every incoming connection the server start a new thread on a UDP port in the range 12000 till 13000.

The relevant code:

```go
var START_PORT = 12000
var END_PORT = 13000

func main() {
    listener, err :=  net.Listen("tcp", ":5000")
    // ..
    for {
        conn, err := listener.Accept()
        go handleConnection(conn)
    }
}

func handleConnection(conn net.Conn) {
    // ..
    go runUdpServer(secret, ln)
}

func startUdpServer() (string,*net.UDPConn) {
    for port := START_PORT; port <= END_PORT; port++ {
        / /..
        ln, err := net.ListenUDP("udp", listenAddr) 
    }
}     
```

The challenge description hints that "the port" is blocked. We guessed the firewall only blocks connections to TCP port 5000, but not to all the threads running on UDP ports.

When connecting to the UDP server we still need a secret, or else the connection is terminated right away:

```go
log.Printf("runUdpServer expected secret: %v\n", secret)
if strings.TrimSpace(string(buf[:n])) != secret{
    return
}
```

The secret is generated based on a call to `rand.Intn`:

```go
b[i] = letterRunes[rand.Intn(len(letterRunes))]
```

From the Go documentation (https://golang.org/pkg/math/rand/) we learn that `rand.Intn` produces a deterministic sequence as the pseudo-random generator is never seeded.

However, while looking into this we also noticed the challenge server seemed different from the supplied source code. It never checked the secret value:


```bash
$ nc -u challenges.openctf.cat 12000
> help

< get
< exit
< set
< help

> get flag
<
```

The only part left was to find the server that actually contained the flag. We found it at port 12021:

```bash
$ echo "get flag" | nc -u challenges.openctf.cat 12021
flag{n0th!ng_could_g0_wrong}
```
