# CTF: Defcon 27 OpenCTF
Challenge: "Web: Stylist"

Description:
>Decode a flag within an HTML page

stylist.html:

```html
<!DOCTYPE html>
<html>
  <head>
  <title>Stylist</title>
  <meta charset="utf-8">
  <style>
    #blocks div {
      height: 2em;
      width: 2em;
      margin: 1em;
      display: inline-block;
    }
  </style>
  </head>
  <body>
    <div id="blocks">
      <div id="n11" style="background: #000062;"></div>
      <div id="n29" style="background: #000074;"></div>
      <div id="n26" style="background: #00005f;"></div>
      <div id="n17" style="background: #00006f;"></div>
      <div id="n08" style="background: #00005f;"></div>
      <div id="n20" style="background: #000061;"></div>
      <div id="n06" style="background: #000068;"></div>
      <div id="n09" style="background: #000072;"></div>
      <div id="n14" style="background: #000074;"></div>
      <div id="n04" style="background: #00007b;"></div>
      <div id="n27" style="background: #000063;"></div>
      <div id="n24" style="background: #000072;"></div>
      <div id="n23" style="background: #000061;"></div>
      <div id="n10" style="background: #000061;"></div>
      <div id="n15" style="background: #00005f;"></div>
      <div id="n01" style="background: #00006c;"></div>
      <div id="n16" style="background: #000067;"></div>
      <div id="n03" style="background: #000067;"></div>
      <div id="n13" style="background: #000069;"></div>
      <div id="n00" style="background: #000066;"></div>
      <div id="n02" style="background: #000061;"></div>
      <div id="n05" style="background: #000074;"></div>
      <div id="n12" style="background: #000062;"></div>
      <div id="n28" style="background: #000075;"></div>
      <div id="n19" style="background: #00005f;"></div>
      <div id="n22" style="background: #000068;"></div>
      <div id="n18" style="background: #000074;"></div>
      <div id="n30" style="background: #00007d;"></div>
      <div id="n21" style="background: #00005f;"></div>
      <div id="n07" style="background: #000065;"></div>
      <div id="n25" style="background: #000065;"></div>
    </div>
  </body>
</html>
```

Solution:
```python
#!/usr/bin/env python

import sys
import re

def main(argv = None):
    if argv is None:
        argv = sys.argv

    filename = "stylist.html"

    try:
        with open(filename, "r") as f:
            lines = f.readlines()
    except:
        return 1

    _dict = {}
    for line in lines:
        m = re.match(r"\s+<div id=\"n(\d{2})\" style=\"background: #0000([\da-f]{2});\"><\/div>", line.strip("\r\n"))
	if m is not None:
            _dict[int(m.group(1))] = m.group(2)

    flag_hex = ""
    for k in sorted(_dict):
        flag_hex = "".join([flag_hex, _dict[k]])

    print "".join(["Flag: ", flag_hex.decode("hex")])

    return 0

if __name__ == "__main__":
    sys.exit(main())
```

The flag:
```
$ python ./doit.py 
Flag: flag{the_rabbit_got_a_hare_cut}
```
