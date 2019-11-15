---
layout:     post
title:      websocket工作原理
subtitle:   
date:       2019-04-11
author:     P
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - python
---
自己写一个websocket(教学用)

{% raw %}
```
 1 import socket, base64, hashlib
 2 
 3 sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
 4 sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
 5 sock.bind(('127.0.0.1', 9527))
 6 sock.listen(5)
 7 # 获取客户端socket对象
 8 conn, address = sock.accept()
 9 # 获取客户端的【握手】信息
10 data = conn.recv(1024)
11 print(data)
12 """
13 b'GET /ws HTTP/1.1\r\n
14 Host: 127.0.0.1:9527\r\n
15 User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:62.0) Gecko/20100101 Firefox/62.0\r\n
16 Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n
17 Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2\r\n
18 Accept-Encoding: gzip, deflate\r\n
19 Sec-WebSocket-Version: 13\r\n
20 Origin: http://localhost:63342\r\n
21 Sec-WebSocket-Extensions: permessage-deflate\r\n
22 Sec-WebSocket-Key: jocLOLLq1BQWp0aZgEWL5A==\r\n
23 Cookie: session=6f2bab18-2dc4-426a-8f06-de22909b967b\r\n
24 Connection: keep-alive, Upgrade\r\n
25 Pragma: no-cache\r\n
26 Cache-Control: no-cache\r\n
27 Upgrade: websocket\r\n\r\n'
28 """
29 
30 # magic string为：258EAFA5-E914-47DA-95CA-C5AB0DC85B11
31 magic_string = '258EAFA5-E914-47DA-95CA-C5AB0DC85B11'
32 
33 
34 def get_headers(data):
35     header_dict = {}
36     header_str = data.decode("utf8")
37     for i in header_str.split("\r\n"):
38         if str(i).startswith("Sec-WebSocket-Key"):
39             header_dict["Sec-WebSocket-Key"] = i.split(":")[1].strip()
40 
41     return header_dict
42 
43 
44 def get_header(data):
45     """
46      将请求头格式化成字典
47      :param data:
48      :return:
49      """
50     header_dict = {}
51     data = str(data, encoding='utf-8')
52 
53     header, body = data.split('\r\n\r\n', 1)
54     header_list = header.split('\r\n')
55     for i in range(0, len(header_list)):
56         if i == 0:
57             if len(header_list[i].split(' ')) == 3:
58                 header_dict['method'], header_dict['url'], header_dict['protocol'] = header_list[i].split(' ')
59         else:
60             k, v = header_list[i].split(':', 1)
61             header_dict[k] = v.strip()
62     return header_dict
63 
64 
65 headers = get_headers(data)  # 提取请求头信息
66 # 对请求头中的sec-websocket-key进行加密
67 response_tpl = "HTTP/1.1 101 Switching Protocols\r\n" \
68                "Upgrade:websocket\r\n" \
69                "Connection: Upgrade\r\n" \
70                "Sec-WebSocket-Accept: %s\r\n" \
71                "WebSocket-Location: ws://127.0.0.1:9527\r\n\r\n"
72 
73 value = headers['Sec-WebSocket-Key'] + magic_string
74 print(value)
75 ac = base64.b64encode(hashlib.sha1(value.encode('utf-8')).digest())
76 response_str = response_tpl % (ac.decode('utf-8'))
77 # 响应【握手】信息
78 conn.send(response_str.encode("utf8"))
79 
80 while True:
81     msg = conn.recv(8096)
82     print(msg)
```
{% endraw %}

解密:

{% raw %}
```
 1 # b'\x81\x83\xceH\xb6\x85\xffz\x85'
 2 
 3 hashstr = b'\x81\x83\xceH\xb6\x85\xffz\x85'
 4 # b'\x81    \x83    \xceH\xb6\x85\xffz\x85'
 5 
 6 # 将第二个字节也就是 \x83 第9-16位 进行与127进行位运算
 7 payload = hashstr[1] & 127
 8 print(payload)
 9 if payload == 127:
10     extend_payload_len = hashstr[2:10]
11     mask = hashstr[10:14]
12     decoded = hashstr[14:]
13 # 当位运算结果等于127时,则第3-10个字节为数据长度
14 # 第11-14字节为mask 解密所需字符串
15 # 则数据为第15字节至结尾
16 
17 if payload == 126:
18     extend_payload_len = hashstr[2:4]
19     mask = hashstr[4:8]
20     decoded = hashstr[8:]
21 # 当位运算结果等于126时,则第3-4个字节为数据长度
22 # 第5-8字节为mask 解密所需字符串
23 # 则数据为第9字节至结尾
24 
25 
26 if payload <= 125:
27     extend_payload_len = None
28     mask = hashstr[2:6]
29     decoded = hashstr[6:]
30 
31 # 当位运算结果小于等于125时,则这个数字就是数据的长度
32 # 第3-6字节为mask 解密所需字符串
33 # 则数据为第7字节至结尾
34 
35 str_byte = bytearray()
36 
37 for i in range(len(decoded)):
38     byte = decoded[i] ^ mask[i % 4]
39     str_byte.append(byte)
40 
41 print(str_byte.decode("utf8"))
```
{% endraw %}

加密:

{% raw %}
```
 1 import struct
 2 msg_bytes = "hello".encode("utf8")
 3 token = b"\x81"
 4 length = len(msg_bytes)
 5 
 6 if length < 126:
 7     token += struct.pack("B", length)
 8 elif length == 126:
 9     token += struct.pack("!BH", 126, length)
10 else:
11     token += struct.pack("!BQ", 127, length)
12 
13 msg = token + msg_bytes
14 
15 print(msg)
```
{% endraw %}
