---
layout: post
title:  "Hello, world!"
date:   2014-12-02 10:40:14
categories: posts
---
### Here is a picture of a parrot hanging on a tree branch

![Parrot](https://www.dropbox.com/s/jb2iekd1ho9v8j5/parrot.jpg?raw=1)

### And here is a chunk of python code

```python
import socket
import time
import sys
from threading import Thread

class ClientThread(Thread):
    def __init__(self, client, addr):
        Thread.__init__(self)
        self.client = client
        self.addr = addr

    def createHeader(self, status='200 OK', contentType='text/html; charset=utf-8', length=None):
        header = []
        header.append('HTTP/1.0 {0}\r\n'.format(status))
        header.append('Content-Type: {0}\r\n'.format(contentType))
        if length != None:
            header.append('Content-Length: {0}\r\n'.format(length))
        header.append('Connection: close\r\n')
        header.append('\r\n')

        return ''.join(header)

    def createResponse(self, body, status='200 OK', contentType='text/html; charset=utf-8', length='None'):
        response = []
        response.append(self.createHeader(status=status, contentType=contentType, length=length))
        response.append(body)

        return ''.join(response).encode(encoding='utf-8')

    def handleURL(self, url):
        try:
            if url == '/':
                url = '/index.html'

            f = open('./www' + url)
        except:
            body = '<h1>400 Not Found.</h1>'
            return self.createResponse(body,
                status='404 Not Found', length=len(body))

        f.seek(0, 2)
        length = f.tell()
        f.seek(0)
        body = f.read()
        f.close()

        return self.createResponse(body, length=length)

    def run(self):
        requestHeader = (self.client.recv(3000)).decode(encoding='utf-8')
        firstLine = requestHeader[:requestHeader.find('\r\n')]
        print(firstLine)

        response = b''
        try:
            url = firstLine.split(' ')[1]
            response = self.handleURL(url)
        except:
            body = '<h1>500 Server Error.</h1>'
            response = self.createResponse(body,
                status='500 Server Error', length=len(body))

        self.client.sendall(response)
        self.client.close()

class Server:
    def __init__(self, host='0.0.0.0', port=8080):
        self.host = host
        self.port = port

    def start(self):
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        s.bind((self.host, self.port))
        s.listen(5)

        print('Server starting http://{host}:{port}/'.format(host=self.host, port=self.port))

        while True:
            r = ClientThread(*s.accept())
            r.start()

if __name__ == '__main__':
    s = Server(port=8011)
    s.start()
```
