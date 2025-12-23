## TCPDUMP Field Guide

- [Basic Usage and Syntax](#1-basic-usage-and-syntax)
- [Capture HTTP GET and POST packets](#2-capture-only-http-get-and-post-packets)
- [Extract HTTP Request URLs](#extract-http-request-urls)
- [Extract HTTP Passwords in POST Requests](#extract-http-passwords-in-post-requests)
- [Capture Cookies from Server and Client](#capture-cookies-from-server-and-client)
- [Traffic Analysis Examples](#traffic-analysis-examples)
- [Unique/Interesting Usecase Examples](#uniqueinteresting-usecase-examples)
- [Security Specific Examples](#security-specific-examples)
- [Protocol Specific Examples](#protocol-specific-examples)

>  It is considered best practice to enclose your capture filters inside single-quotes
>  See the second example in the list highlihts below 

**Some Highlights:**
```sh
$ tcpdump -i eth0
$ tcpdump -D
$ tcpdump -c 100
$ tcpdump -w file.cap
$ tcpdump -r file.cap
$ tcpdump -v 
$ tcpdump -enni eth0 port 8116 
$ sudo tcpdump -i eth0 -v src port 443 and dst 192.168.3.107
$ sudo tcpdump -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420'
$ sudo tcpdump -nn -A -s1500 -l | grep "User-Agent:"
$ tcpdump -nni eth0 dst 10.2.1.3 and not port 22  
$ tcpdump -nni eth0 src 10.2.3.4 or src 10.7.1.2  
$ sudo tcpdump -s 0 -A -n -l | egrep -i "POST /|pwd=|passwd=|password=|Host:"
$ sudo tcpdump -nn -A -s0 -l | egrep -i 'Set-Cookie|Host:|Cookie:'
$ sudo tcpdump -s 0 -v -n -l | egrep -i "POST /|GET /|Host:"
$ sudo tcpdump -nn -v port ftp or ftp-data
$ sudo tcpdump 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
$ tcpdump -w /tmp/traffic.pcap -i eth0 -v 'tcp and net 192.168.2.0/24'
```

---

### Basic Usage and Syntax

**Common Flags**
```
 -i = interface  
 -v = verbose  (-v, -vv, -vvv)
 -s 0 = size of capture  (common sizes include 0 (all), 64, 1500, 65535 bytes)
 -n = no DNS resolution
 -nn = no DNS/protocol resolution
 -A = print ASCII
 -e = show mac addresses
 -c = number of bytes to capture
 -l = line buffered mode
```

**Filter by Port/Protocol**
Note: `protocol 17` is `udp`. Filtering on either of these will produce the same results.
The equivalent of the `tcp` filter is `protocol 6`
```
tcpdump -nni eth0 port 8116      // specify port
tcpdump -nni eth3 ip proto 17    // specify protocol by number
tcpdump -nni eth3 icmp           // specify protocol by name

tcpdump -i eth0 udp
tcpdump -i eth0 proto 17
tcpdump -i eth0 tcp
tcpdump -i eth0 proto 6
```

**Size examples**
```
-s 0		//Captures the entire packet (full frame), best when saving to a file for later analysis in Wireshark.
-s 64		//Captures only the first 64 bytes of each frame. (when only IP and protocol headers arequired.
-s 1500		//Captures up to 1500 bytes per packet, a common MTU of standard Ethernet frames.
-s 65535	//A very large number (e.g., 65535 or 65536) used to ensure the capture of full-sized packets -s 0.
```

---

#### ASCII AND HEX OUTPUT

Hex and ASCII output is useful when inspecting payload contents. 
ASCII is useful for using with grep and other string manipulation commands
Capture ICMP packets with full hex and ASCII output

**ASCII**
```sh
tcpdump -nni ens160 icmp -A -c 1
```
```
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ens160, link-type EN10MB (Ethernet), snapshot length 262144 bytes
16:37:25.589407 IP 192.168.2.98.53819 > ecorpserver89.ssh: Flags [P.], seq 1536768612:1536768648, ack 3169130755, win 9795, length 36
E..L..@...@....b..FY.;..[.:d....P.&C3............c...^s......d..vh...z@,.7YS
16:37:25.591019 IP ecorpserver89.ssh > 192.168.2.98.53819: Flags [P.], seq 1:37, ack 36, win 648, length 36
E..L..@.@.Y...FY...b...;....[.:.P....J..v.W..\......+.p.^...5..Vp~.v,.%...=m
2 packets captured
14 packets received by filter
0 packets dropped by kernel
```

```sh
tcpdump -nni ens160 icmp -XX -c 1
```
```
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ens160, link-type EN10MB (Ethernet), snapshot length 262144 bytes
16:37:34.587700 IP ecorpserver89.ssh > 192.168.2.98.53819: Flags [P.], seq 3169140439:3169140635, ack 1536777392, win 648, length 196
        0x0000:  4510 00ec 17b7 4000 4006 5839 c0a8 4659  E.....@.@.X9..FY
        0x0010:  c0a8 0262 0016 d23b bce5 3ed7 5b99 5cb0  ...b...;..>.[.\.
        0x0020:  5018 0288 caea 0000 7e16 7442 94fd dabf  P.......~.tB....
        0x0030:  b5d0 bb97 64a5 a89b d9f2 173d da00 7316  ....d......=..s.
        0x0040:  2ca8 4c5c b60d 27cb 43a0 8510 cd50 867a  ,.L\..'.C....P.z
        0x0050:  3d57 c866 22c4 da67 d692 d242 3666 70c3  =W.f"..g...B6fp.
        0x0060:  2815 ab8d 6d03 33e9 7ee9 e63f 4671 2c4f  (...m.3.~..?Fq,O
        0x0070:  c5b9 408a 5bf4 9868 c032 727e 83c2 4580  ..@.[..h.2r~..E.
        0x0080:  ab03 7086 21c2 b7ee d6f7 f513 7e1f 97f0  ..p.!.......~...
        0x0090:  eeeb dc2d b375 fc6a a74e ab47 b18f d85c  ...-.u.j.N.G...\
        0x00a0:  069e ca2e 99f7 2399 4d37 0dfa 354b daca  ......#.M7..5K..
        0x00b0:  f4e9 2d20 23d6 afa0 aaac 66ed 7394 9073  ..-.#.....f.s..s
        0x00c0:  694b c8bb d48c 8ff5 7c9b 65d8 a609 eea8  iK......|.e.....
        0x00d0:  d048 6782 6199 7f8e 6f95 6a9f de60 fd08  .Hg.a...o.j..`..
        0x00e0:  1510 f238 2016 1d65 90db a79c            ...8...e....
```
		
			
#### FILTERS
```
src 172.17.0.1       // outgoing traffic from this IP  
dst 172.17.0.1       // incoming traffic to this IP  
host 172.17.0.1      // in/out traffic for this host  
```
```
net 192.168.2.0/24   // entire subnet  
net 172.17.0.0/16  
net 10.0.0.0/8  
```
```
tcpdump -i docker0 -v tcp and dst 172.17.0.1
```

#### INTERFACE
```
tcpdump -D
```
```
1.eth0  
2.docker0  
3.veth2629542  
4.any  
5.lo  
```

#### SOURCE / DESTINATION EXAMPLES
```sh
# Capture traffic going from any source to DST:
sudo tcpdump -i ens160 -v dst 192.168.70.89

# Capture traffic from specific SRC going to any destination:
sudo tcpdump -i ens160 -v src 192.168.50.101

# Capture all traffic (incoming/outgoing) for a specific host:
sudo tcpdump -i ens160 -v host 192.168.70.89
```

#### CONDITIONAL FILTERING
Common directional and logical keywords:
```
- src   : source
- dst   : destination
- host  : either direction
- port  : specific port
```

#### Logical operators:
```
- and / &&
- or  / ||
- not / !
```

#### Examples:

Capture traffic destined for 10.2.1.3 except SSH
```sh
tcpdump -nni eth0 dst 10.2.1.3 and not port 22
```

Capture traffic originating from either source host
```sh
tcpdump -nni eth0 src 10.2.3.4 or src 10.7.1.2
```

### Line Buffered Mode

Without the option to force line (`-l`) buffered (or packet buffered `-C`) mode you will not always get the **expected response when piping** the `tcpdump` output to another command such as `grep`. By using this option the output is sent immediately to the piped command giving an immediate response when troubleshooting.
```sh
$ sudo tcpdump -i eth0 -s0 -l port 80 | grep 'Server:'
```

---

### Capture only HTTP GET and POST packets

#### GET
```sh
sudo tcpdump -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420'
```
```
19:50:51.030963 IP (tos 0x0, ttl 63, id 38978, offset 0, flags [DF], proto TCP (6), length 1070)
    192.168.50.101.42964 > ecorpserver89.http: Flags [P.], cksum 0x7a23 (correct), seq 46298:47316, ack 261967, win 501, options [nop,nop,TS val 2962034020 ecr 2577633262], length 1018: HTTP, length: 1018
        GET /api/messages/6 HTTP/1.1
        Host: 192.168.70.89
        User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:88.0) Gecko/20100101 Firefox/88.0
        Accept: */*
        Accept-Language: en-CA,en-US;q=0.7,en;q=0.3
        Accept-Encoding: gzip, deflate
        Referer: http://192.168.70.89/chat/6
        Connection: keep-alive
        Cookie: XSRF-TOKEN=eyJpdiI6IlVqU2xRU2RMY2R2ZkVJSGNYQUxMUWc9PSIsInZhbHVlIjoiaThOOXBzSmsySUhBQlVpWUc3Uk1sREt2eDBXZ3ZyVDA1U2hBMXlwZ3pSdERHcjJIUlBZam0wZFlLYWUvK0REQ0Y0UWhDTFcwM1hoTjNMbzB3eXlwWXFyOTZYOVVaUmgvK0hRaFlXSk5xOUFOM3FDRnRzVGlzMk15ZHczblRtQ0QiLCJtYWMiOiJmMzJmNWU1MDU3MzE0MzI4NzYwN2RhODY4MDI4M2E2MGI3NzhhOGU4YTA4Mjg4ZmMxNTRiZWJiNTc3ZmRmZjlmIiwidGFnIjoiIn0%3D; chatter-session=eyJpdiI6Imc5cVhhUTRtZmsvVzZjRXF1UTlOc0E9PSIsInZhbHVlIjoiZlNEY0NxaFlJUlRlamt6emRpYlhjcVNrWit1SWVnd0JtMUpkWTdBbTgzTFRFWEMxYnVkRVhjNFd6Qi9IdFBnUDlYcE0zZ1pyQTBXZ1RGNGhQQ3JjMko1RURQY2VRVVJHbDZvQis3alBCY2RpYm12dnNhZGtnVDBVeFJHbkNzVFUiLCJtYWMiOiJjNjU1MzVjOWE4ZDFiMDY3NjkwOGQyMWRiM2E2NjlkYzM2Y2RiYzMyNmQwMmRiZTI3MmQ5ZGNhNWI0OTI2NDQzIiwidGFnIjoiIn0%3D

E....B@.?..x..2e..FY...PD+....  .....z#.....
...d....GET /api/messages/6 HTTP/1.1
Host: 192.168.70.89
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:88.0) Gecko/20100101 Firefox/88.0
Accept: */*
Accept-Language: en-CA,en-US;q=0.7,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: http://192.168.70.89/chat/6
Connection: keep-alive
Cookie: XSRF-TOKEN=eyJpdiI6IlVqU2xRU2RMY2R2ZkVJSGNYQUxMUWc9PSIsInZhbHVlIjoiaThOOXBzSmsySUhBQlVpWUc3Uk1sREt2eDBXZ3ZyVDA1U2hBMXlwZ3pSdERHcjJIUlBZam0wZFlLYWUvK0REQ0Y0UWhDTFcwM1hoTjNMbzB3eXlwWXFyOTZYOVVaUmgvK0hRaFlXSk5xOUFOM3FDRnRzVGlzMk15ZHczblRtQ0QiLCJtYWMiOiJmMzJmNWU1MDU3MzE0MzI4NzYwN2RhODY4MDI4M2E2MGI3NzhhOGU4YTA4Mjg4ZmMxNTRiZWJiNTc3ZmRmZjlmIiwidGFnIjoiIn0%3D; chatter-session=eyJpdiI6Imc5cVhhUTRtZmsvVzZjRXF1UTlOc0E9PSIsInZhbHVlIjoiZlNEY0NxaFlJUlRlamt6emRpYlhjcVNrWit1SWVnd0JtMUpkWTdBbTgzTFRFWEMxYnVkRVhjNFd6Qi9IdFBnUDlYcE0zZ1pyQTBXZ1RGNGhQQ3JjMko1RURQY2VRVVJHbDZvQis3alBCY2RpYm12dnNhZGtnVDBVeFJHbkNzVFUiLCJtYWMiOiJjNjU1MzVjOWE4ZDFiMDY3NjkwOGQyMWRiM2E2NjlkYzM2Y2RiYzMyNmQwMmRiZTI3MmQ5ZGNhNWI0OTI2NDQzIiwidGFnIjoiIn0%3D
```

#### POST
```sh
sudo tcpdump -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354'
```
```
19:57:03.833509 IP (tos 0x0, ttl 127, id 15621, offset 0, flags [DF], proto TCP (6), length 1253)
    192.168.2.98.49627 > ecorpserver89.http: Flags [P.], cksum 0xc08b (correct), seq 9416:10629, ack 53974, win 510, length 1213: HTTP, length: 1213
        POST /api/messages HTTP/1.1
        Host: 192.168.70.89
        User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:143.0) Gecko/20100101 Firefox/143.0
        Accept: */*
        Accept-Language: en-US,en;q=0.5
        Accept-Encoding: gzip, deflate
        Referer: http://192.168.70.89/chat/5
        Content-Type: application/json
        X-CSRF-TOKEN: N04IDg7d1PNHSpow9ig6zXqr1BucJYclYXr7n8wE
        Content-Length: 51
        Origin: http://192.168.70.89
        Connection: keep-alive
        Cookie: XSRF-TOKEN=eyJpdiI6InN3UCs5Mk5IVVJsK2tMR1RDR0ZQNkE9PSIsInZhbHVlIjoiems2TzYvVlBlY3B2b3Y1OEsxclI1OXFTOHZDNXFyc0lBL3BleDMrM09OQjk3bXZzWkhwT0FRQ1lubU1lbkpiR1JzRWFkamNNV0dwaG9OMWMxOWV4OENXcmhRS2o3cnJKNGgzZ3hUK0dldy9sT0lSbEJXaWRMMnFBNC9qaTFGRlkiLCJtYWMiOiI4YzE0MDA4OWJmYzc2MGY5MTFlOTdhNjViYjQ5OWNlOTRmNjQ2ZDE1NzExNDg4NTg3ZGIzZGVlZWE4ZTllMGYyIiwidGFnIjoiIn0%3D; chatter-session=eyJpdiI6IlJMOHhDZE5RNjRMQ3g1UVBnS0FjcGc9PSIsInZhbHVlIjoiSGZZRVFpbythWWo2ZjF5MnVyUEhMc1VWNUFnRUh1QTZkNVZGU1EwTzNQN0VQNHlBdHRrYkNyemEwTUtFaDJ1OWU0dis3eGhUM2tPVjRqSEdpeUMvb00zR1JDcXdZV29hcEwyaTY5UXZLYVNvYzNrM0pRSlJ2b3lGeTJxcnZnb3YiLCJtYWMiOiIwMjQ2MmJjMzEyM2JkYzMwMzhhNmIyNzM4OWVlNjcwYmYyZTk3Y2ZhNmI0NGVmZWRmYjc4NTdhN2I5OTgyZGJiIiwidGFnIjoiIn0%3D
        Priority: u=0

        {"receiver_id":5,"message":"Hi Dude, this is Doug"} [|http]
E...=.@........b..FY...P...sj...P.......POST /api/messages HTTP/1.1
Host: 192.168.70.89
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:143.0) Gecko/20100101 Firefox/143.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://192.168.70.89/chat/5
Content-Type: application/json
X-CSRF-TOKEN: N04IDg7d1PNHSpow9ig6zXqr1BucJYclYXr7n8wE
Content-Length: 51
Origin: http://192.168.70.89
Connection: keep-alive
Cookie: XSRF-TOKEN=eyJpdiI6InN3UCs5Mk5IVVJsK2tMR1RDR0ZQNkE9PSIsInZhbHVlIjoiems2TzYvVlBlY3B2b3Y1OEsxclI1OXFTOHZDNXFyc0lBL3BleDMrM09OQjk3bXZzWkhwT0FRQ1lubU1lbkpiR1JzRWFkamNNV0dwaG9OMWMxOWV4OENXcmhRS2o3cnJKNGgzZ3hUK0dldy9sT0lSbEJXaWRMMnFBNC9qaTFGRlkiLCJtYWMiOiI4YzE0MDA4OWJmYzc2MGY5MTFlOTdhNjViYjQ5OWNlOTRmNjQ2ZDE1NzExNDg4NTg3ZGIzZGVlZWE4ZTllMGYyIiwidGFnIjoiIn0%3D; chatter-session=eyJpdiI6IlJMOHhDZE5RNjRMQ3g1UVBnS0FjcGc9PSIsInZhbHVlIjoiSGZZRVFpbythWWo2ZjF5MnVyUEhMc1VWNUFnRUh1QTZkNVZGU1EwTzNQN0VQNHlBdHRrYkNyemEwTUtFaDJ1OWU0dis3eGhUM2tPVjRqSEdpeUMvb00zR1JDcXdZV29hcEwyaTY5UXZLYVNvYzNrM0pRSlJ2b3lGeTJxcnZnb3YiLCJtYWMiOiIwMjQ2MmJjMzEyM2JkYzMwMzhhNmIyNzM4OWVlNjcwYmYyZTk3Y2ZhNmI0NGVmZWRmYjc4NTdhN2I5OTgyZGJiIiwidGFnIjoiIn0%3D
Priority: u=0

{"receiver_id":5,"message":"Hi Dude, this is Doug"}
```
Explanation: selects 4 bytes at the TCP header offset and matches ASCII for `GET` or `POST`.

---

### Extract HTTP Request URLs

```sh
sudo tcpdump -s 0 -v -n -l | egrep -i "POST /|GET /|Host:"
```
```
20:00:00.185347 IP 192.168.2.98.49742 > 192.168.70.89.80: Flags [P.], seq 1345605675:1345606700, ack 267291506, win 513, length 1025: HTTP:GET /api/messages/5 HTTP/1.1
20:00:03.080462 IP 192.168.50.101.42986 > 192.168.70.89.80: Flags [P.], seq 1018:2036, ack 6771, win 501, options [nop,nop,TS val 2962586052 ecr 2578185314], length 1018: HTTP: GET /api/messages/6 HTTP/1.1
20:00:03.262062 IP 192.168.2.98.49742 > 192.168.70.89.80: Flags [P.], seq 1025:2050, ack 6771, win 509, length 1025: HTTP: GET /api/messages/5 HTTP/1.1
20:00:06.080564 IP 192.168.50.101.42986 > 192.168.70.89.80: Flags [P.], seq 2036:3054, ack 13541, win 501, options [nop,nop,TS val 2962589053 ecr 2578188309], length 1018: HTTP: GET /api/messages/6 HTTP/1.1
20:00:06.267996 IP 192.168.2.98.49742 > 192.168.70.89.80: Flags [P.], seq 2050:3075, ack 13541, win 509, length 1025: HTTP: GET /api/messages/5 HTTP/1.1
20:00:10.172923 IP 192.168.2.98.49742 > 192.168.70.89.80: Flags [P.], seq 4100:5296, ack 27081, win 513, length 1196: HTTP: POST /api/messages HTTP/1.1
20:00:10.207985 IP 192.168.2.98.49742 > 192.168.70.89.80: Flags [P.], seq 5296:6321, ack 28426, win 508, length 1025: HTTP: GET /api/messages/5 HTTP/1.1
```

---

### Extract HTTP Passwords in POST Requests

```sh
sudo tcpdump -s 0 -A -n -l | egrep -i "POST /|pwd=|passwd=|password=|Host:"
```
```
Host: 192.168.70.89
20:12:01.213163 IP 192.168.2.98.50048 > 192.168.70.89.80: Flags [P.], seq 1:1296, ack 1, win 513, length 1295: HTTP: POST /login HTTP/1.1
E..7Os@....A...b..FY...P....5L..P....k..POST /login HTTP/1.1
Host: 192.168.70.89
_token=gZev3GUoBtLZcNZY1Y4Fe8ZJGOuc2T01a0dDNKzE&email=dude%40dude.com&password=password
```
---

### Capture Cookies from Server and Client

```
sudo tcpdump -nn -A -s0 -l | egrep -i 'Set-Cookie|Host:|Cookie:'
```
```
Set-Cookie: chatter-session=eyJpdiI6IlhHQ2d2L2pnaEFpZzFKTndwVk5sZ2c9PSIsInZhbHVlIjoiWk13R1FDTTQ3bkYrSzB2R3o4clNnT2lxdkx4eEx2ZGJoaHBJZzg0S2c2RHBQYWdCUlp3cUVJTjBkODZCOHZaR2o3MThiTDJMQWhoMWllSmI2K3p3V1dObUV0eFhtY0lieFNuOXYxQXZDeHQ2a2JCWmh3MUdDK25MY2prVGlPVjUiLCJtYWMiOiI0MjQyZDRkOGQ4ZDAzOTg4MWM2YTE0NTU1ODViMjE5MWQ5OGExNDU2YjVlMzk2MjBlYjU1ZGM1Yjc3NzQzMjkzIiwidGFnIjoiIn0%3D; expires=Mon, 22 Dec 2025 22:15:21 GMT; Max-Age=7200; path=/; httponly; samesite=lax
Host: 192.168.70.89
Cookie: XSRF-TOKEN=eyJpdiI6InJvRWZ3WmlDcFd4K1l5bDNzeThrM3c9PSIsInZhbHVlIjoiWkxXbkRURm9pTVBBaUlmYXkzM3pBTkU0dGw1VGxmU0MyNkpLYXZrU2xIU0owa1RrLytwRzJUT01yVm82YUFhOVJvUHFSY1JpKzlOT2FNSjN3OHd4ZlJkVnNob0ZyYWlReGZ1LzBCR0gzN01GZUtjQlBMZ1ZZeEhTT1hKdW9hVHYiLCJtYWMiOiIyNzA0MTcyNGYzZGNiYjQ5ODRiMTNhMzc3ODg1NjIxNDYyNGVmYTdhNmM4MDE0ZGYwZTg3OWRkMjRhZDVkNzg4IiwidGFnIjoiIn0%3D; chatter-session=eyJpdiI6IlhHQ2d2L2pnaEFpZzFKTndwVk5sZ2c9PSIsInZhbHVlIjoiWk13R1FDTTQ3bkYrSzB2R3o4clNnT2lxdkx4eEx2ZGJoaHBJZzg0S2c2RHBQYWdCUlp3cUVJTjBkODZCOHZaR2o3MThiTDJMQWhoMWllSmI2K3p3V1dObUV0eFhtY0lieFNuOXYxQXZDeHQ2a2JCWmh3MUdDK25MY2prVGlPVjUiLCJtYWMiOiI0MjQyZDRkOGQ4ZDAzOTg4MWM2YTE0NTU1ODViMjE5MWQ5OGExNDU2YjVlMzk2MjBlYjU1ZGM1Yjc3NzQzMjkzIiwidGFnIjoiIn0%3D
Set-Cookie: XSRF-TOKEN=eyJpdiI6IndSZzB1VWkya00zSlU5S3UxbVh2TXc9PSIsInZhbHVlIjoicyswR29kUEw2d2NieU5DelBUQXBwN2swMzRmb2NHZXZxWWlQeWc2ZjVOS09YSlppL2JoVTVyM0pLQUhtUEY4Y2lrOFRxM0p1bFh0VCt4Sk82MExQRlRhL3hlL3Q3Qm1rSHRpZ0YzZ1ozclNtaW52S1RSSXgxdTNyNGp4Q0VZTXYiLCJtYWMiOiI2Njc0MjU2ZWRjZDcyZjI2ZTgxN2ZmZGMxMjc1MDM3MzMyYjZhNTNjMmM1Nzg3YzE3NGNkMjA3ZTUyZGRkMjIyIiwidGFnIjoiIn0%3D; expires=Mon, 22 Dec 2025 22:15:24 GMT; Max-Age=7200; path=/; samesite=lax
Set-Cookie: chatter-session=eyJpdiI6IjNsaUtRc3RIbEFpZlpCSlRNaFhqOHc9PSIsInZhbHVlIjoiaGVmMTVhYXU0WWxpQnpjT0lJbmYxRDlJd3llTCtjSXdEMC9TL3ZRTnEwamlEN0ZvUmY2OVQ4L0Rtb2lrME4vQ1BCc0s4VGowYi9HZnZRWnRmWW1oQnJFUkY4Z09VNmVWTkxid21OTUtsL05HSmkzMXoxOTVCR3liRXVEUTU2dnoiLCJtYWMiOiI5N2Q2MWE4MTRkYTVjYTMzZGMyNmE4OWQ2YzI4MTFjMDY2ODU0ZWJlZjM2M2U1YjA4ZTIxNDZiYjgyZGQ0MjE5IiwidGFnIjoiIn0%3D; expires=Mon, 22 Dec 2025 22:15:24 GMT; Max-Age=7200; path=/; httponly; samesite=lax
Host: 192.168.70.89
Cookie: XSRF-TOKEN=eyJpdiI6IkZwQmllRURLNkV2SS9iZkhNN1ptNEE9PSIsInZhbHVlIjoidXdKTTVmUUREZjFUQ3VQU0RGU0tGRnlDWW9UY1dIZGJpUCtQdU9xb1pud2x2TTVrVWsxa3JpdWVqQ0ZQZjJaSzRTOEloOCtUaGVtY0wzYVJPT1pwMDhXZm9PUE14ZVBSYXprQ3VPbnRGdnNBQndidlFTY25IM2YydzFpOEN3V0YiLCJtYWMiOiI3MThlZGM3ODA4M2VjY2QzYTIyNjhlY2JhM2JkYTY1OWI4N2IzOWZlNTIwMmI0MjA2NDUyNmQyZjY4NTgxNGU0IiwidGFnIjoiIn0%3D; chatter-session=eyJpdiI6IlBRZUdRWkZtR2kwZ2dNMDYwdVNGRWc9PSIsInZhbHVlIjoiQ0ZJOVpJYzF6QTJoQ1N0UkNlSEJIRkE3ZG9TeGlCK0VDbnQ2Ky9zRytIVG1vOW95Z0V5NUdKaEZMRWVuNzArQTIxSWtvVHRxMUpteUJESXJBUjV3bVI1c3VoNThuT0VPN1RPdkl2TGhtYjVxMzN2ZGVSeHc5M0tFQWx5OEFlN0ciLCJtYWMiOiJiY2RiOWUzNDg4ZDQ2YzlhNDE3ZThlOTg4YzAwNzNjMDVlZDAwYjc1NzExYjIxNmRkYThlMDFhZmRkNTk2MTZiIiwidGFnIjoiIn0%3D
Set-Cookie: XSRF-TOKEN=eyJpdiI6IlZvVkJnQmNraWhmbVNabXRtQjRncVE9PSIsInZhbHVlIjoid1dyRllJOVZVcURub0wxYUdsb2Z4UjROcWlKL0UxUzFjVGhoSXVKN1k3cmwwN1ltenZtM09hVU9lZlYxNzdFNlVUYm0yZUJqME5OZ1Iwait2STZyZkFvS1JGNjRBWkVUWER6SG8rbytMeVlza1c5RTZMall1STQxa1Z0N0N4a3AiLCJtYWMiOiI4YmJjNDMzZDQ0MzM0NmNkZDE2ZjFiZWUxOTA5ZjdjMmM5YjAwZTc4MmZkYmRmMDk1NjY5NzE1M2U3OTUxMGU3IiwidGFnIjoiIn0%3D; expires=Mon, 22 Dec 2025 22:15:25 GMT; Max-Age=7200; path=/; samesite=lax
Set-Cookie: chatter-session=eyJpdiI6InRaU2ZoWWpJWU0wQXFUbTJ3cG1IMEE9PSIsInZhbHVlIjoidmtzUlE2YXRycmVVYjFXQ1pSSXpodHBGVFZnaFJweGRObU91UjNrUExEdFY1V2RKckNrWnI1blZrUnZSRTM4MGZKR05acXRoNTFJN0lYMjlxTXFYdU9YRWNTMWE0Y1JmUDBSL3NxZnF1TlRXcWZwV3BkNDFnU1dtTHl4d2lxS0EiLCJtYWMiOiJmYjE3NTg0YzVhMWU0NmU0MWFjMjMxMmRmZmZiYmZhMTEyMTFiNzM1YmZmOTUxZjYzYjJjZTkzZjk2MjU3NTlmIiwidGFnIjoiIn0%3D; expires=Mon, 22 Dec 2025 22:15:25 GMT; Max-Age=7200; path=/; httponly; samesite=lax
Host: 192.168.70.89
```

---

### Traffic Analysis Examples


#### Capture HTTP data packets only (avoid SYN/FIN)
```sh
tcpdump 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```
```
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ens160, link-type EN10MB (Ethernet), snapshot length 262144 bytes
20:24:44.927868 IP 192.168.2.98.50258 > ecorpserver89.http: Flags [P.], seq 2434379395:2434380420, ack 1803245960, win 513, length 1025: HTTP: GET /api/messages/5 HTTP/1.1
20:24:44.947148 IP ecorpserver89.http > 192.168.2.98.50258: Flags [P.], seq 1:2665, ack 1025, win 947, length 2664: HTTP: HTTP/1.1 200 OK
20:24:44.947760 IP ecorpserver89.http > 192.168.2.98.50258: Flags [P.], seq 2665:2670, ack 1025, win 947, length 5: HTTP
20:24:45.206792 IP 192.168.50.101.43018 > ecorpserver89.http: Flags [P.], seq 2083551530:2083552548, ack 2365063898, win 501, options [nop,nop,TS val 2964068133 ecr 2579667434], length 1018: HTTP: GET /api/messages/6 HTTP/1.1
20:24:45.235616 IP ecorpserver89.http > 192.168.50.101.43018: Flags [P.], seq 1:6943, ack 1018, win 476, options [nop,nop,TS val 2579670435 ecr 2964068133], length 6942: HTTP: HTTP/1.1 200 OK
20:24:45.236665 IP ecorpserver89.http > 192.168.50.101.43018: Flags [P.], seq 6943:6948, ack 1018, win 476, options [nop,nop,TS val 2579670436 ecr 2964068163], length 5: HTTP
20:24:47.933552 IP 192.168.2.98.50258 > ecorpserver89.http: Flags [P.], seq 1025:2050, ack 2670, win 513, length 1025: HTTP: GET /api/messages/5 HTTP/1.1
20:24:47.953644 IP ecorpserver89.http > 192.168.2.98.50258: Flags [P.], seq 2670:5334, ack 2050, win 963, length 2664: HTTP: HTTP/1.1 200 OK
20:24:50.962959 IP ecorpserver89.http > 192.168.2.98.50258: Flags [P.], seq 8003:8008, ack 3075, win 979, length 5: HTTP
20:24:51.206902 IP 192.168.50.101.43018 > ecorpserver89.http: Flags [P.], seq 2036:3054, ack 13895, win 501, options [nop,nop,TS val 2964074133 ecr 2579673436], length 1018: HTTP: GET /api/messages/6 HTTP/1.1
20:24:51.235925 IP ecorpserver89.http > 192.168.50.101.43018: Flags [P.], seq 13895:20837, ack 3054, win 476, options [nop,nop,TS val 2579676435 ecr 2964074133], length 6942: HTTP: HTTP/1.1 200 OK
20:24:51.236997 IP ecorpserver89.http > 192.168.50.101.43018: Flags [P.], seq 20837:20842, ack 3054, win 476, options [nop,nop,TS val 2579676436 ecr 2964074163], length 5: HTTP
20:24:52.225898 IP 192.168.2.98.50258 > ecorpserver89.http: Flags [P.], seq 3075:4186, ack 8008, win 513, length 1111: HTTP: GET /profile HTTP/1.1
20:24:52.256062 IP ecorpserver89.http > 192.168.2.98.50258: Flags [P.], seq 8008:15308, ack 4186, win 1002, length 7300: HTTP: HTTP/1.1 200 OK
20:24:52.256084 IP ecorpserver89.http > 192.168.2.98.50258: Flags [P.], seq 15308:17895, ack 4186, win 1002, length 2587: HTTP
20:24:52.257178 IP ecorpserver89.http > 192.168.2.98.50258: Flags [P.], seq 17895:17915, ack 4186, win 1002, length 20: HTTP
20:24:52.451169 IP 192.168.2.98.50258 > ecorpserver89.http: Flags [P.], seq 4186:5226, ack 17915, win 508, length 1040: HTTP: GET /web/connections HTTP/1.1
20:24:52.474192 IP ecorpserver89.http > 192.168.2.98.50258: Flags [P.], seq 17915:21505, ack 5226, win 1020, length 3590: HTTP: HTTP/1.1 200 OK
20:24:52.474919 IP ecorpserver89.http > 192.168.2.98.50258: Flags [P.], seq 21505:21510, ack 5226, win 1020, length 5: HTTP
20:24:54.206682 IP 192.168.50.101.43018 > ecorpserver89.http: Flags [P.], seq 3054:4072, ack 20842, win 501, options [nop,nop,TS val 2964077133 ecr 2579676436], length 1018: HTTP: GET /api/messages/6 HTTP/1.1
20:24:54.239027 IP ecorpserver89.http > 192.168.50.101.43018: Flags [P.], seq 20842:27784, ack 4072, win 476, options [nop,nop,TS val 2579679438 ecr 2964077133], length 6942: HTTP: HTTP/1.1 200 OK
20:24:54.239976 IP ecorpserver89.http > 192.168.50.101.43018: Flags [P.], seq 27784:27789, ack 4072, win 476, options [nop,nop,TS val 2579679439 ecr 2964077166], length 5: HTTP
20:24:57.206637 IP 192.168.50.101.43018 > ecorpserver89.http: Flags [P.], seq 4072:5090, ack 27789, win 501, options [nop,nop,TS val 2964080133 ecr 2579679439], length 1018: HTTP: GET /api/messages/6 HTTP/1.1
20:24:57.236411 IP ecorpserver89.http > 192.168.50.101.43018: Flags [P.], seq 27789:34731, ack 5090, win 476, options [nop,nop,TS val 2579682435 ecr 2964080133], length 6942: HTTP: HTTP/1.1 200 OK
```

#### Capture start and end packets of every non-local host
```sh
tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0 and not src and dst net 192.168.70.89'
```
```
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ens160, link-type EN10MB (Ethernet), snapshot length 262144 bytes
20:32:21.816403 IP 192.168.2.98.50546 > ecorpserver89.http: Flags [S], seq 2792114968, win 64240, options [mss 1460,nop,wscale 8,nop,nop,sackOK], length 0
20:32:21.816453 IP ecorpserver89.http > 192.168.2.98.50546: Flags [S.], seq 3127622320, ack 2792114969, win 64240, options [mss 1460,nop,nop,sackOK,nop,wscale 7], length 0
20:32:26.841507 IP ecorpserver89.http > 192.168.2.98.50546: Flags [F.], seq 3800, ack 1104, win 508, length 0
20:32:26.842620 IP 192.168.2.98.50546 > ecorpserver89.http: Flags [F.], seq 1104, ack 3801, win 513, length 0
20:32:29.222596 IP 192.168.2.98.50550 > ecorpserver89.http: Flags [S], seq 399659350, win 64240, options [mss 1460,nop,wscale 8,nop,nop,sackOK], length 0
20:32:29.222648 IP ecorpserver89.http > 192.168.2.98.50550: Flags [S.], seq 969895427, ack 399659351, win 64240, options [mss 1460,nop,nop,sackOK,nop,wscale 7], length 0
20:32:34.865130 IP ecorpserver89.http > 192.168.2.98.50550: Flags [F.], seq 13347, ack 4464, win 586, length 0
20:32:34.868797 IP 192.168.2.98.50550 > ecorpserver89.http: Flags [F.], seq 4464, ack 13348, win 513, length 0
20:32:42.102887 IP ecorpserver89.http > 192.168.50.101.43028: Flags [F.], seq 195975249, ack 3795472487, win 755, options [nop,nop,TS val 2580147302 ecr 2964540012], length 0
20:32:42.104237 IP 192.168.50.101.43028 > ecorpserver89.http: Flags [F.], seq 1, ack 1, win 501, options [nop,nop,TS val 2964545018 ecr 2580147302], length 0
20:32:43.907898 IP 192.168.50.101.43038 > ecorpserver89.http: Flags [S], seq 1404721746, win 64240, options [mss 1460,sackOK,TS val 2964546821 ecr 0,nop,wscale 7], length 0
20:32:43.907949 IP ecorpserver89.http > 192.168.50.101.43038: Flags [S.], seq 1513160921, ack 1404721747, win 65160, options [mss 1460,sackOK,TS val 2580149107 ecr 2964546821,nop,wscale 7], length 0
20:32:44.444070 IP 192.168.50.101.43038 > ecorpserver89.http: Flags [F.], seq 3436, ack 9327, win 501, options [nop,nop,TS val 2964547357 ecr 2580149479], length 0
20:32:44.444475 IP 192.168.50.101.43042 > ecorpserver89.http: Flags [S], seq 2339628451, win 64240, options [mss 1460,sackOK,TS val 2964547358 ecr 0,nop,wscale 7], length 0
20:32:44.444510 IP ecorpserver89.http > 192.168.50.101.43042: Flags [S.], seq 487364797, ack 2339628452, win 65160, options [mss 1460,sackOK,TS val 2580149644 ecr 2964547358,nop,wscale 7], length 0
20:32:44.444659 IP ecorpserver89.http > 192.168.50.101.43038: Flags [F.], seq 10746, ack 3437, win 573, options [nop,nop,TS val 2580149644 ecr 2964547357], length 0
20:32:47.214271 IP 192.168.2.98.50558 > ecorpserver89.http: Flags [S], seq 1404107130, win 64240, options [mss 1460,nop,wscale 8,nop,nop,sackOK], length 0
20:32:47.214320 IP ecorpserver89.http > 192.168.2.98.50558: Flags [S.], seq 655305131, ack 1404107131, win 64240, options [mss 1460,nop,nop,sackOK,nop,wscale 7], length 0
20:32:49.523782 IP ecorpserver89.http > 192.168.50.101.43042: Flags [F.], seq 2600, ack 1019, win 502, options [nop,nop,TS val 2580154723 ecr 2964547433], length 0
20:32:49.524502 IP 192.168.50.101.43042 > ecorpserver89.http: Flags [F.], seq 1019, ack 2601, win 501, options [nop,nop,TS val 2964552438 ecr 2580154723], length 0
20:32:50.204788 IP 192.168.50.101.43044 > ecorpserver89.http: Flags [S], seq 1127727092, win 64240, options [mss 1460,sackOK,TS val 2964553118 ecr 0,nop,wscale 7], length 0
20:32:50.204832 IP ecorpserver89.http > 192.168.50.101.43044: Flags [S.], seq 1240260882, ack 1127727093, win 65160, options [mss 1460,sackOK,TS val 2580155404 ecr 2964553118,nop,wscale 7], length 0
20:32:52.277895 IP ecorpserver89.http > 192.168.2.98.50558: Flags [F.], seq 10874, ack 2368, win 545, length 0
20:32:52.279938 IP 192.168.2.98.50558 > ecorpserver89.http: Flags [F.], seq 2368, ack 10875, win 513, length 0
^C
24 packets captured
24 packets received by filter
0 packets dropped by kernel
```

#### Top hosts by packets
```sh
sudo tcpdump -nnn -t -c 200 | cut -f 1,2,3,4 -d '.' | sort | uniq -c | sort -nr | head -n 20
```
```
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ens160, link-type EN10MB (Ethernet), snapshot length 262144 bytes
200 packets captured
322 packets received by filter
0 packets dropped by kernel
     88 IP 192.168.70.89
     75 IP 192.168.2.98
     37 IP 192.168.50.101
```

#### Rotate Capture Files
```sh
tcpdump -w /tmp/capture-%H.pcap -G 3600 -C 200

Requires permissions to test this:
https://askubuntu.com/questions/530920/tcpdump-permissions-problem

```

#### Capture a range of ports
```sh
tcpdump 'tcp portrange 1000-2000'
```
```
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ens160, link-type EN10MB (Ethernet), snapshot length 262144 bytes
21:22:45.517498 IP 192.168.2.98.51851 > ecorpserver89.1001: Flags [S], seq 2883458823, win 64240, options [mss 1460,nop,wscale 8,nop,nop,sackOK], length 0
21:22:45.517544 IP ecorpserver89.1001 > 192.168.2.98.51851: Flags [R.], seq 0, ack 2883458824, win 0, length 0
21:22:45.768567 IP 192.168.2.98.51852 > ecorpserver89.1001: Flags [S], seq 2093355131, win 64240, options [mss 1460,nop,wscale 8,nop,nop,sackOK], length 0
21:22:45.768609 IP ecorpserver89.1001 > 192.168.2.98.51852: Flags [R.], seq 0, ack 2093355132, win 0, length 0
21:22:46.029023 IP 192.168.2.98.51851 > ecorpserver89.1001: Flags [S], seq 2883458823, win 64240, options [mss 1460,nop,wscale 8,nop,nop,sackOK], length 0
21:22:46.029063 IP ecorpserver89.1001 > 192.168.2.98.51851: Flags [R.], seq 0, ack 1, win 0, length 0
```

#### Filter out noise (e.g., SSH and broadcast)
```sh
tcpdump not port 22 and not broadcast
```
```
21:23:50.006305 IP 192.168.50.101.43130 > ecorpserver89.http: Flags [P.], seq 81930424:81931442, ack 17157335, win 5657, options [nop,nop,TS val 2967612824 ecr 2583212229], length 1018: HTTP: GET /api/messages/6 HTTP/1.1
21:23:50.030506 IP ecorpserver89.http > 192.168.50.101.43130: Flags [P.], seq 1:6942, ack 1018, win 941, options [nop,nop,TS val 2583215230 ecr 2967612824], length 6941: HTTP: HTTP/1.1 200 OK
21:23:50.031220 IP 192.168.50.101.43130 > ecorpserver89.http: Flags [.], ack 1449, win 5657, options [nop,nop,TS val 2967612850 ecr 2583215230], length 0
21:23:50.031227 IP ecorpserver89.http > 192.168.50.101.43130: Flags [P.], seq 6942:6947, ack 1018, win 941, options [nop,nop,TS val 2583215230 ecr 2967612824], length 5: HTTP
21:23:50.031228 IP 192.168.50.101.43130 > ecorpserver89.http: Flags [.], ack 4345, win 5635, options [nop,nop,TS val 2967612850 ecr 2583215230], length 0
21:23:50.031263 IP 192.168.50.101.43130 > ecorpserver89.http: Flags [.], ack 5793, win 5626, options [nop,nop,TS val 2967612850 ecr 2583215230], length 0
21:23:50.031292 IP 192.168.50.101.43130 > ecorpserver89.http: Flags [.], ack 6942, win 5618, options [nop,nop,TS val 2967612850 ecr 2583215230], length 0
21:23:50.031440 IP 192.168.50.101.43130 > ecorpserver89.http: Flags [.], ack 6947, win 5657, options [nop,nop,TS val 2967612850 ecr 2583215230], length 0
```

#### Time-based filtering / limited duration
```sh
tcpdump -i eth0 -G 3600 -w capture-%H.pcap
```
---

### Unique/Interesting Usecase Examples

#### Limit capture by packet size
```sh
tcpdump -s 128 tcp
```

#### Monitoring bandwidth / packet sizes
```sh
tcpdump -i eth0 -nn -q
```
```
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ens160, link-type EN10MB (Ethernet), snapshot length 262144 bytes
21:25:43.571168 IP 192.168.70.89.22 > 192.168.2.98.65387: tcp 196
21:25:43.577393 IP 192.168.2.98.65387 > 192.168.70.89.22: tcp 36
21:25:43.577481 IP 192.168.70.89.22 > 192.168.2.98.65387: tcp 36
21:25:43.612639 IP 192.168.2.98.65387 > 192.168.70.89.22: tcp 36
21:25:43.612750 IP 192.168.70.89.22 > 192.168.2.98.65387: tcp 36
21:25:43.655511 IP 192.168.2.98.65387 > 192.168.70.89.22: tcp 0
21:25:43.655525 IP 192.168.2.98.65387 > 192.168.70.89.22: tcp 36
21:25:43.655620 IP 192.168.70.89.22 > 192.168.2.98.65387: tcp 36
21:25:43.669798 IP 192.168.70.89.22 > 192.168.2.98.65387: tcp 564
21:25:43.670608 IP 192.168.2.98.65387 > 192.168.70.89.22: tcp 0
21:25:43.701792 IP 192.168.2.98.65387 > 192.168.70.89.22: tcp 36
21:25:43.701871 IP 192.168.70.89.22 > 192.168.2.98.65387: tcp 36
21:25:43.732886 IP 192.168.2.98.65387 > 192.168.70.89.22: tcp 36
21:25:43.732976 IP 192.168.70.89.22 > 192.168.2.98.65387: tcp 36
21:25:43.764191 IP 192.168.2.98.65387 > 192.168.70.89.22: tcp 36
21:25:43.764304 IP 192.168.70.89.22 > 192.168.2.98.65387: tcp 36
21:25:43.773794 IP 192.168.70.89.22 > 192.168.2.98.65387: tcp 564
```

#### Custom complex expression
```sh
tcpdump 'tcp port 80 and host 10.0.0.5 and not src 10.0.0.1'
```

#### Capture with tcpdump and view in Wireshark
```sh
ssh root@remotesystem 'tcpdump -s0 -c 1000 -nn -w - not port 22' | wireshark -k -i -
```

#### Reading from multiple capture files
```sh
tcpdump -r 'capture-*.pcap'
```
---

### Security Specific Examples


#### Detect port scan in network traffic
```sh
tcpdump -nn
```
Look for SYN [S] packets to multiple ports followed by RST [R.] responses.

#### Example: Nmap NSE Script Testing
On Nmap machine:
```sh
nmap -p 80 --script=http-enum.nse $targetip
```
On target machine:
```sh
tcpdump -nn port 80 | grep "GET /"
```

#### Capture all plaintext passwords
```sh
sudo tcpdump port http or port ftp or port smtp or port imap or port pop3 or port telnet -l -A | egrep -i -B5 'pass=|pwd=|log=|login=|user=|username=|pw=|passw=|passwd=|password=|pass:|user:|username:|password:|login:|pass |user '
```
```
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ens160, link-type EN10MB (Ethernet), snapshot length 262144 bytes
Referer: http://192.168.70.89/login
Cookie: XSRF-TOKEN=eyJpdiI6Ik0xemhGQnRYVXhhUDI0S0xKU0RhUHc9PSIsInZhbHVlIjoiQnUvN2hvdFJiYWcvNmtHYXRRMXVmcStFdE5zU2Q5R1Z3ZjJDVEwrb3dobzArU0xtMGJvTmJLWlNneGNKTVE1MHZRazc5czhQcksxYXJOeEhBY2VDbDBSa2ZZVHJYclhVZWJFSy8yUjRPVWhHZ1g4dTk5WFpaTzZ4Vm9nRnpCeE4iLCJtYWMiOiIyNDc0OWMwZGYyODg0OWFiYmU4ODc1NTM1MTNkZTE3YjdhNjZmNTljMWIxMDFmMmY0ZWI5NjI5YWQ5NWQ4MTE3IiwidGFnIjoiIn0%3D; chatter-session=eyJpdiI6IlRpaDZrVEMxZXd6aDI2QzdiOVNhcnc9PSIsInZhbHVlIjoiaEZKc05hSjhtdXZHd3BXWnNQSktvNllqTDR4WGJBbDJwZDc2Uy9KcHNVTE5MRTRtbVZIVGZOZ2RVcVdEZXNaaTh1ODhVdnlZTGN0QnE5MkpYdHJiUm9ia3hHWTNXbmJOVXppNHRRZlcrTVgyWXU2bThwNGZ1Y0pjSHJYdThpNXgiLCJtYWMiOiI1OTM5NDRhOTE5Mjk3ZjBiYjc0YTU5Mjk2MWFmMTkzOTBlYzJlYzkwOGEzMGQxYzQyM2FmYzQwYWNkNGIxMWMzIiwidGFnIjoiIn0%3D
Upgrade-Insecure-Requests: 1
Priority: u=0, i

_token=uvmeXZe4Tc355HeVqGDOJHzeOD5W6QvsvJPa06UQ&email=bob%40bob.com&password=password123
```

#### Capture only SYN packets
```sh
tcpdump 'tcp[tcpflags] & tcp-syn != 0'
```

---

### Protocol Specific Examples

#### Capture IPv6 traffic
```sh
tcpdump -nn ip6 proto 6  
tcpdump -nr ipv6-test.pcap ip6 proto 17
```

#### Capture all ICMP packets
```sh
sudo tcpdump -n icmp
```

#### Show ICMP packets that are not echo/reply
```sh
sudo tcpdump 'icmp[icmptype] != icmp-echo and icmp[icmptype] != icmp-echoreply'
```

#### Capture SMTP / POP3 Email recipients
```sh
sudo tcpdump -nn -l port 25 | grep -i 'MAIL FROM\|RCPT TO'
```

#### Troubleshooting NTP Query/Response
```sh
sudo tcpdump dst port 123
```

#### Capture SNMP Query and Response
```sh
onesixtyone 10.10.1.10 public  
sudo tcpdump -n -s0 port 161 and udp
```

#### Capture FTP credentials and commands
```sh
sudo tcpdump -nn -v port ftp or ftp-data
```

#### Capture DNS request and response
```sh
sudo tcpdump -i wlp58s0 -s0 port 53
```

#### DHCP example
```sh
sudo tcpdump -v -n port 67 or 68
```

#### Capture ARP traffic
```sh
tcpdump -i eth0 arp
```

#### Capture TLS/SSL handshake traffic
```sh
tcpdump -i eth0 port 443 -w tls.pcap
```

#### Capture multicast / broadcast traffic
```sh
tcpdump 'udp and (dst 224.0.0.0/4 or broadcast)'
```
---





















---
## Unsorted Notes





---

### Combine Filters

Throughout these examples you can use standard logic to combine different filters.

**and** or **&&**
**or** or **||**
**not** or **!**

---

### Random Snippets from CheckPoint

```sh
tcpdump -D                     //available devices
tcpdump -c 100                 //capture 100 packets
tcpdump -w filename.cap
tcpdump -nnr filename.cap      //read the file, will apply the nn to playback
tcpdump -nni [eth0|any]  
tcpdump -nni eth3 port 8116    //capture CCP packets (UDP 17) on sync interface
tcpdump -nni eth3 ip proto 17  //capture UDP 17 (CCP) on sync interface

tcpdump -nni eth1 icmp               //grab icmp
tcpdump -nni eth0 ip proto 50        //grab ESP packets
tcpdump -vv | grep ‘proto: ESP (50)’ //requires vv for grep match 
                                     // ‘ESP(spi’ doesn’t require -vv

tcpdump -nni eth1 dst 10.2.10.223 and not port 22
dst - destination
src - source
host - either direction

tcpdump -nni eth1 host 10.2.10.223 and not port 22 and not dst 10.2.10.223
tcpdump -nni eth1 src 10.2.10.223 or src 10.1.10.223

tcpdump -nni eth1 icmp -XX           //hex and ascii content
tcpdump -nni eth1 ip proto 50 -XX    //seems to be same output as X? 

ROUTE
route -n //kernel routing table
route -Cn //kernel routing cache

fw getifs
```
---

## Practical Capture Filters Examples

In many of these examples there are a number of ways that the result could be achieved. As seen in some of the examples it is possible to focus the capture right down to individual bits in the packet.

The method you will use will depend on your desired output and how much traffic is on the wire. Capturing on a busy gigabit link may force you to use specific low level packet filters.

When troubleshooting you often simply want to get a result. Filtering on the port and selecting **ascii** output in combination with `grep`, `cut` or `awk` will often get that result. You can always go deeper into the packet if required.

For example when capturing HTTP requests and responses you could filter out all packets except the data by removing **SYN /ACK / FIN** however if you are using `grep` the noise will be filtered anyway. Keep it simple.

This can be seen in the following examples, where the aim is to get a result in the simplest (and therefore fastest) manner.

### 1. Extract HTTP User Agents

Extract HTTP User Agent from HTTP request header.
```sh
$ sudo tcpdump -nn -A -s1500 -l | grep "User-Agent:"
```

By using `egrep` and multiple matches we can get the User Agent and the Host (or any other header) from the request.
```sh
$ sudo tcpdump -nn -A -s1500 -l | egrep -i 'User-Agent:|Host:'
```

```sh
$ sudo tcpdump -i docker0 -nn -A -s1500 -l | grep "User-Agent:"

tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on docker0, link-type EN10MB (Ethernet), snapshot length 1500 bytes
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
```
---
### 2. Capture only HTTP GET and POST packets

Going deep on the filter we can specify only packets that match GET.
The hexadecimal being matched in these expressions matches the ascii for GET and POST.

```d
$ sudo tcpdump -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420'
```
Alternatively we can select only on POST requests. Note that the POST data may not be included in the packet captured with this filter. It is likely that a POST request will be split across multiple TCP data packets.
```d
$ sudo tcpdump -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354'
```

As an explanation `'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354'` first determine the size of the offset, and then selects the 4 bytes we wish to match against. See [[TCP Filter Breakdown]] for full breakdown of this command - very cool 💩

```
# sudo tcpdump -i docker0 -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420'

tcpdump: listening on docker0, link-type EN10MB (Ethernet), snapshot length 262144 bytes 16:20:47.308109 IP (tos 0x0, ttl 64, id 55880, offset 0, flags [DF], proto TCP (6), length 610)

172.17.0.1.54126 > 172.17.0.2.http: Flags [P.], cksum 0x5a7a (incorrect -> 0x9fd7), seq 708741811:708742369, ack 610411666, win 502, options [nop,nop,TS val 34580271 ecr 1601544035], length 558: HTTP, length: 558
        GET /vulnerabilities/exec/ HTTP/1.1
        Host: localhost
        User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
        Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
        Accept-Language: en-US,en;q=0.5
        Accept-Encoding: gzip, deflate, br, zstd
        Referer: http://localhost/index.php
        Connection: keep-alive
        Cookie: PHPSESSID=s59443q0q799oejfccdtg02mq7; security=low
        Upgrade-Insecure-Requests: 1
        Sec-Fetch-Dest: document
        Sec-Fetch-Mode: navigate
        Sec-Fetch-Site: same-origin
        Sec-Fetch-User: ?1
        Priority: u=0, i

E..b.H@.@..(.........n.P*>..$b$.....Zz.....
.../_u.cGET /vulnerabilities/exec/ HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br, zstd
Referer: http://localhost/index.php
Connection: keep-alive
Cookie: PHPSESSID=s59443q0q799oejfccdtg02mq7; security=low
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i

1 packet captured
1 packet received by filter
0 packets dropped by kernel

```

### 3. Extract HTTP Request URL's

Parse **Host** and **HTTP Request location** from traffic. By not targeting **port 80** we may find these requests on any port such as HTTP services running on high ports.
```d
$ sudo tcpdump -s 0 -v -n -l | egrep -i "POST /|GET /|Host:"
```
```
tcpdump: listening on enp7s0, link-type EN10MB (Ethernet), capture size 262144 bytes
	POST /wp-login.php HTTP/1.1
	Host: dev.example.com
	GET /wp-login.php HTTP/1.1
	Host: dev.example.com
	GET /favicon.ico HTTP/1.1
	Host: dev.example.com
	GET / HTTP/1.1
	Host: dev.example.com
```
---

### 4. Extract HTTP Passwords in POST Requests

Lets get some passwords from the POST data. Will include Host: and request location so we know what the password is used for.
```d
$ sudo tcpdump -s 0 -A -n -l | egrep -i "POST /|pwd=|passwd=|password=|Host:"
```
```
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp7s0, link-type EN10MB (Ethernet), capture size 262144 bytes
11:25:54.799014 IP 10.10.1.30.39224 > 10.10.1.125.80: Flags [P.], seq 1458768667:1458770008, ack 2440130792, win 704, options [nop,nop,TS val 461552632 ecr 208900561], length 1341: 
HTTP: POST /wp-login.php HTTP/1.1
.....s..POST /wp-login.php HTTP/1.1
Host: dev.example.com
.....s..log=admin&pwd=notmypassword&wp-submit=Log+In&redirect_to=http%3A%2F%2Fdev.example.com%2Fwp-admin%2F&testcookie=1
```
---
### 5. Capture Cookies from Server and from Client

MMMmmm Cookies! Capture cookies from the server by searching on Set-Cookie: (from Server) and Cookie: (from Client).
```d
$ sudo tcpdump -nn -A -s0 -l | egrep -i 'Set-Cookie|Host:|Cookie:'
```
```
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on wlp58s0, link-type EN10MB (Ethernet), capture size 262144 bytes
Host: dev.example.com
Cookie: wordpress_86be02xxxxxxxxxxxxxxxxxxxc43=admin%7C152xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxfb3e15c744fdd6; _ga=GA1.2.21343434343421934; _gid=GA1.2.927343434349426; wordpress_test_cookie=WP+Cookie+check; wordpress_logged_in_86be654654645645645654645653fc43=admin%7C15275102testtesttesttestab7a61e; wp-settings-time-1=1527337439
```

Logging out/in of DVWA 
```
$ sudo tcpdump -i docker0 -nn -A -s0 -l | egrep -i 'Set-Cookie|Host:|Cookie:'

tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on docker0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
Host: localhost
Cookie: PHPSESSID=s59443q0q799oejfccdtg02mq7; security=low
Host: localhost
Cookie: PHPSESSID=s59443q0q799oejfccdtg02mq7; security=low
Host: localhost
Cookie: PHPSESSID=s59443q0q799oejfccdtg02mq7; security=low
Host: localhost
Cookie: PHPSESSID=s59443q0q799oejfccdtg02mq7; security=low
```
---
### 6. Capture all ICMP packets

See all `ICMP` packets on the wire.
```d
$ sudo tcpdump -n icmp
```
```
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp7s0, link-type EN10MB (Ethernet), capture size 262144 bytes
11:34:21.590380 IP 10.10.1.217 > 10.10.1.30: ICMP echo request, id 27948, seq 1, length 64
11:34:21.590434 IP 10.10.1.30 > 10.10.1.217: ICMP echo reply, id 27948, seq 1, length 64
11:34:27.680307 IP 10.10.1.159 > 10.10.1.1: ICMP 10.10.1.189 udp port 59619 unreachable, length 115
```
---

### 7. Show ICMP Packets that are not ECHO/REPLY (standard ping)

Filter on the `icmp` type to select on `icmp` packets that are not standard `ping` packets.
```d
$ sudo tcpdump 'icmp[icmptype] != icmp-echo and icmp[icmptype] != icmp-echoreply'
```
```
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp7s0, link-type EN10MB (Ethernet), capture size 262144 bytes
11:37:04.041037 IP 10.10.1.189 > 10.10.1.20: ICMP 10.10.1.189 udp port 36078 unreachable, length 156
```
---

### 8. Capture SMTP / POP3 Email

It is possible to extract email body and other data, in this example we are only parsing the email recipients.
```d
$ sudo tcpdump -nn -l port 25 | grep -i 'MAIL FROM\|RCPT TO'
```
---

### 9. Troubleshooting NTP Query and Response

In this example we see the NTP query and response.
```d
$ sudo tcpdump dst port 123
```
```
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
21:02:19.112502 IP test33.ntp > 199.30.140.74.ntp: NTPv4, Client, length 48
21:02:19.113888 IP 216.239.35.0.ntp > test33.ntp: NTPv4, Server, length 48
21:02:20.150347 IP test33.ntp > 216.239.35.0.ntp: NTPv4, Client, length 48
21:02:20.150991 IP 216.239.35.0.ntp > test33.ntp: NTPv4, Server, length 48
```
---
### 10. Capture SNMP Query and Response

Using `onesixtyone` the fast SNMP protocol scanner we test an SNMP service on our local network and capture the `GetRequest` and `GetResponse`. For anyone who has had the (dis)pleasure of troubleshooting SNMP, this is a great way to see exactly what is happening on the wire. You can see the `OID` clearly in the traffic, very helpful when wrestling with `MIBS`.

```d
$ onesixtyone 10.10.1.10 public
```
```
Scanning 1 hosts, 1 communities
10.10.1.10 [public] Linux test33 4.15.0-20-generic #21-Ubuntu SMP Tue Apr 24 06:16:15 UTC 2018 x86_64
```
```d
$ sudo tcpdump -n -s0  port 161 and udp
```
```
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on wlp58s0, link-type EN10MB (Ethernet), capture size 262144 bytes
23:39:13.725522 IP 10.10.1.159.36826 > 10.10.1.20.161:  GetRequest(28)  .1.3.6.1.2.1.1.1.0
23:39:13.728789 IP 10.10.1.20.161 > 10.10.1.159.36826:  GetResponse(109)  .1.3.6.1.2.1.1.1.0="Linux testmachine 4.15.0-20-generic #21-Ubuntu SMP Tue Apr 24 06:16:15 UTC 2018 x86_64"
```
---

### 11. Capture FTP Credentials and Commands

Capturing FTP commands and login details is straight forward. After the authentication is established an FTP session can be **active** or **passive** this will determine whether the data part of the session is conducted over TCP port 20 or another ephemeral port. With the following command you will see USER and PASS in the output (which could be fed to grep) as well as the FTP commands such as LIST, CWD and PASSIVE.

```d
$ sudo tcpdump -nn -v port ftp or ftp-data
```
---
### 12. Rotate Capture Files

When capturing large amounts of traffic or over a long period of time it can be helpful to automatically create new files of a fixed size. This is done using the parameters 
	`-W    // write to file
	`-G    // Group a new file every n seconds`       
	`-C    // No idea TBFT`

In this command the file **capture-(hour).pcap** will be created every (-G) **3600 seconds** (1 hour). The files will be overwritten the following day. So you should end up with 
**capture-{1-24}.pcap**, if the hour was **15** the new file is (**/tmp/capture-15.pcap**).
```d
$ tcpdump  -w /tmp/capture-%H.pcap -G 3600 -C 200
```
---
### 13. Capture IPv6 Traffic

Capture IPv6 traffic using the `ip6` filter. In these examples we have specified the TCP and UDP protocols using `proto 6` and `proto 17`.
```d
# tcpdump -nn ip6 proto 6
```
IPv6 with UDP and reading from a previously saved capture file.
```d
# tcpdump -nr ipv6-test.pcap ip6 proto 17
```
---
### 14. Detect Port Scan in Network Traffic

In the following example you can see the traffic coming from a single source to a single destination. The Flags `[S]` and `[R.]` can be seen and matched against a seemingly random series of destination ports. These ports are seen in the RESET that is sent when the SYN finds a closed port on the destination system. This is standard behaviour for a port scan by a tool such as Nmap.
```d
$ tcpdump -nn
```
```
21:46:19.693601 IP 10.10.1.10.60460 > 10.10.1.199.5432: Flags [S], seq 116466344, win 29200, options [mss 1460,sackOK,TS val 3547090332 ecr 0,nop,wscale 7], length 0
21:46:19.693626 IP 10.10.1.10.35470 > 10.10.1.199.513: Flags [S], seq 3400074709, win 29200, options [mss 1460,sackOK,TS val 3547090332 ecr 0,nop,wscale 7], length 0
21:46:19.693762 IP 10.10.1.10.44244 > 10.10.1.199.389: Flags [S], seq 2214070267, win 29200, options [mss 1460,sackOK,TS val 3547090333 ecr 0,nop,wscale 7], length 0
21:46:19.693772 IP 10.10.1.199.389 > 10.10.1.10.44244: Flags [R.], seq 0, ack 2214070268, win 0, length 0
21:46:19.693783 IP 10.10.1.10.35172 > 10.10.1.199.1433: Flags [S], seq 2358257571, win 29200, options [mss 1460,sackOK,TS val 3547090333 ecr 0,nop,wscale 7], length 0
21:46:19.693826 IP 10.10.1.10.33022 > 10.10.1.199.49153: Flags [S], seq 2406028551, win 29200, options [mss 1460,sackOK,TS val 3547090333 ecr 0,nop,wscale 7], length 0
21:46:19.695567 IP 10.10.1.10.55130 > 10.10.1.199.49154: Flags [S], seq 3230403372, win 29200, options [mss 1460,sackOK,TS val 3547090334 ecr 0,nop,wscale 7], length 0
21:46:19.695590 IP 10.10.1.199.49154 > 10.10.1.10.55130: Flags [R.], seq 0, ack 3230403373, win 0, length 0
21:46:19.695608 IP 10.10.1.10.33460 > 10.10.1.199.49152: Flags [S], seq 3289070068, win 29200, options [mss 1460,sackOK,TS val 3547090335 ecr 0,nop,wscale 7], length 0
21:46:19.695622 IP 10.10.1.199.49152 > 10.10.1.10.33460: Flags [R.], seq 0, ack 3289070069, win 0, length 0
21:46:19.695637 IP 10.10.1.10.34940 > 10.10.1.199.1029: Flags [S], seq 140319147, win 29200, options [mss 1460,sackOK,TS val 3547090335 ecr 0,nop,wscale 7], length 0
21:46:19.695650 IP 10.10.1.199.1029 > 10.10.1.10.34940: Flags [R.], seq 0, ack 140319148, win 0, length 0
21:46:19.695664 IP 10.10.1.10.45648 > 10.10.1.199.5060: Flags [S], seq 2203629201, win 29200, options [mss 1460,sackOK,TS val 3547090335 ecr 0,nop,wscale 7], length 0
21:46:19.695775 IP 10.10.1.10.49028 > 10.10.1.199.2000: Flags [S], seq 635990431, win 29200, options [mss 1460,sackOK,TS val 3547090335 ecr 0,nop,wscale 7], length 0
21:46:19.695790 IP 10.10.1.199.2000 > 10.10.1.10.49028: Flags [R.], seq 0, ack 635990432, win 0, length 0
```
---
### 15. Example Filter Showing Nmap NSE Script Testing

In this example the Nmap NSE script `http-enum.nse` is shown testing for valid urls against an open HTTP service.

On the Nmap machine:
```d
$ nmap -p 80 --script=http-enum.nse $targetip
```

On the target machine:
```d
$ tcpdump -nn port 80 | grep "GET /"
```
```
GET /w3perl/ HTTP/1.1
GET /w-agora/ HTTP/1.1
GET /way-board/ HTTP/1.1
GET /web800fo/ HTTP/1.1
GET /webaccess/ HTTP/1.1
GET /webadmin/ HTTP/1.1
GET /webAdmin/ HTTP/1.1
```
---
### 16. Capture Start and End Packets of every non-local host

This example is straight out of the `tcpdump` man page. By selecting on the `tcp-syn` and `tcp-fin` packets we can show each established TCP conversation with timestamps but without the data. As with many filters this allows the amount of noise to be reduced in order to focus in on the information that you care about.

```d
$ tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0 and not src and dst net localnet'
```
---
### 17. Capture DNS Request and Response

Outbound DNS request to Google public DNS and the A record (ip address) response can be seen in this capture.
```d
$ sudo tcpdump -i wlp58s0 -s0 port 53
```
```
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on wlp58s0, link-type EN10MB (Ethernet), capture size 262144 bytes
14:19:06.879799 IP test.53852 > google-public-dns-a.google.com.domain: 26977+ [1au] A? play.google.com. (44)
14:19:07.022618 IP google-public-dns-a.google.com.domain > test.53852: 26977 1/0/1 A 216.58.203.110 (60)
```
---
### 18. Capture HTTP data packets

Only capture on HTTP data packets on port 80. 
Avoid capturing the TCP session setup (SYN / FIN / ACK).
```d
tcpdump 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```
---
### 19. Capture with tcpdump and view in Wireshark

Parsing and analysis of full application streams such as HTTP is much easier to perform with Wireshark or tshark rather than `tcpdump`. 

It is often more practical to capture traffic on a remote system using `tcpdump` with the write file option. Then copy the `pcap` to the local workstation for analysis with Wireshark.

Other than manually moving the file from the remote system to the local workstation it is possible to feed the capture to Wireshark over the SSH connection in real time. This **tip is a favorite**, pipe the raw `tcpdump` output right into `wireshark` on your local machine. **Don't forget** the `not port 22` so you are not capturing your SSH traffic.
```d
$ ssh root@remotesystem 'tcpdump -s0 -c 1000 -nn -w - not port 22' | wireshark -k -i -
```
Another tip is to use count `-c` on the remote `tcpdump` to allow the capture to finish otherwise hitting `ctrl-c` will not only kill `tcpdump` but also Wireshark and your capture.

---

### 20. Top Hosts by Packets

List the top talkers for a period of time or number of packets. Using simple command line field extraction to get the IP address, sort and count the occurrances. Capture is limited by the count option `-c`.
```d
# sudo tcpdump -nnn -t -c 200 | cut -f 1,2,3,4 -d '.' | sort | uniq -c | sort -nr | head -n 20
```
```
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp7s0, link-type EN10MB (Ethernet), capture size 262144 bytes
200 packets captured
261 packets received by filter
0 packets dropped by kernel
    108 IP 10.10.211.181
     91 IP 10.10.1.30
      1 IP 10.10.1.50
```
---
### 21. Capture all the plaintext passwords

In this command we are focusing on standard plain text protocols and chosing to grep on anything user or password related. By selecting the `-B5` option on `grep` the aim is to get the preceding 5 lines that may provide context around the captured password (hostname, ip address, system).
```d
$ sudo tcpdump port http or port ftp or port smtp or port imap or port pop3 or port telnet -l -A | egrep -i -B5 'pass=|pwd=|log=|login=|user=|username=|pw=|passw=|passwd=|password=|pass:|user:|username:|password:|login:|pass |user '
```
---
### 22. DHCP Example

And our final `tcpdump` example is for monitoring DHCP request and reply. DHCP requests are seen on **port 67** and the reply is on **68**. Using the verbose parameter `-v` we get to see the protocol options and other details.

```d
$ sudo tcpdump -v -n port 67 or 68
```
```
tcpdump: listening on enp7s0, link-type EN10MB (Ethernet), capture size 262144 bytes
14:37:50.059662 IP (tos 0x10, ttl 128, id 0, offset 0, flags [none], proto UDP (17), length 328)
    0.0.0.0.68 > 255.255.255.255.67: BOOTP/DHCP, Request from 00:0c:xx:xx:xx:d5, length 300, xid 0xc9779c2a, Flags [none]
	  Client-Ethernet-Address 00:0c:xx:xx:xx:d5
	  Vendor-rfc1048 Extensions
	    Magic Cookie 0x63825363
	    DHCP-Message Option 53, length 1: Request
	    Requested-IP Option 50, length 4: 10.10.1.163
	    Hostname Option 12, length 14: "test-ubuntu"
	    Parameter-Request Option 55, length 16: 
	      Subnet-Mask, BR, Time-Zone, Default-Gateway
	      Domain-Name, Domain-Name-Server, Option 119, Hostname
	      Netbios-Name-Server, Netbios-Scope, MTU, Classless-Static-Route
	      NTP, Classless-Static-Route-Microsoft, Static-Route, Option 252
14:37:50.059667 IP (tos 0x10, ttl 128, id 0, offset 0, flags [none], proto UDP (17), length 328)
    0.0.0.0.68 > 255.255.255.255.67: BOOTP/DHCP, Request from 00:0c:xx:xx:xx:d5, length 300, xid 0xc9779c2a, Flags [none]
	  Client-Ethernet-Address 00:0c:xx:xx:xx:d5
	  Vendor-rfc1048 Extensions
	    Magic Cookie 0x63825363
	    DHCP-Message Option 53, length 1: Request
	    Requested-IP Option 50, length 4: 10.10.1.163
	    Hostname Option 12, length 14: "test-ubuntu"
	    Parameter-Request Option 55, length 16: 
	      Subnet-Mask, BR, Time-Zone, Default-Gateway
	      Domain-Name, Domain-Name-Server, Option 119, Hostname
	      Netbios-Name-Server, Netbios-Scope, MTU, Classless-Static-Route
	      NTP, Classless-Static-Route-Microsoft, Static-Route, Option 252
14:37:50.060780 IP (tos 0x0, ttl 64, id 53564, offset 0, flags [none], proto UDP (17), length 339)
    10.10.1.1.67 > 10.10.1.163.68: BOOTP/DHCP, Reply, length 311, xid 0xc9779c2a, Flags [none]
	  Your-IP 10.10.1.163
	  Server-IP 10.10.1.1
	  Client-Ethernet-Address 00:0c:xx:xx:xx:d5
	  Vendor-rfc1048 Extensions
	    Magic Cookie 0x63825363
	    DHCP-Message Option 53, length 1: ACK
	    Server-ID Option 54, length 4: 10.10.1.1
	    Lease-Time Option 51, length 4: 86400
	    RN Option 58, length 4: 43200
	    RB Option 59, length 4: 75600
	    Subnet-Mask Option 1, length 4: 255.255.255.0
	    BR Option 28, length 4: 10.10.1.255
	    Domain-Name-Server Option 6, length 4: 10.10.1.1
	    Hostname Option 12, length 14: "test-ubuntu"
	    T252 Option 252, length 1: 10
	    Default-Gateway Option 3, length 4: 10.10.1.1
```

### Conclusion

These `tcpdump` examples, tips and commands are intended to give you a base understanding of the possibilities. Depending on what you are trying to achieve there are many ways that you could go deeper or combine different capture filters to suit your requirements.

Combining `tcpdump` with Wireshark is a powerful combination, particularly when you wish to dig into full application layer sessions as the decoders can assemble the full stream. 

---

Usually define interface to use, and use verbose in most commands
```
# tcpdump -i eth0 -v ...
```
### Help Contents 
Can export to pcap for analysis in Wireshark
```
Usage: tcpdump [-AbdDefhHIJKlLnNOpqStuUvxX#] [ -B size ] [ -c count ] [--count]
                [ -C file_size ] [ -E algo:secret ] [ -F file ] [ -G seconds ]
                [ -i interface ] [ --immediate-mode ] [ -j tstamptype ]
                [ -M secret ] [ --number ] [ --print ] [ -Q in|out|inout ]
                [ -r file ] [ -s snaplen ] [ -T type ] [ --version ]
                [ -V file ] [ -w file ] [ -W filecount ] [ -y datalinktype ]
                [ --time-stamp-precision precision ] [ --micro ] [ --nano ]
                [ -z postrotate-command ] [ -Z user ] [ expression ]
```
---
### Available Devices
```
# tcpdump -D

1.eth0 [Up, Running, Connected]
2.docker0 [Up, Running, Connected]
3.veth2629542 [Up, Running, Connected]
4.any (Pseudo-device that captures on all interfaces) [Up, Running]
5.lo [Up, Running, Loopback]
6.br-af4a685a82bf [Up, Disconnected]
7.bluetooth-monitor (Bluetooth Linux Monitor) [Wireless]
8.nflog (Linux netfilter log (NFLOG) interface) [none]
9.nfqueue (Linux netfilter queue (NFQUEUE) interface) [none]
10.dbus-system (D-Bus system bus) [none]
11.dbus-session (D-Bus session bus) [none]

// -i interface to monitor
# tcpdump -i docker0

// verbose output with -v
# tcpdump -i docker0 -v
```

---
### Filters

from `ifconfig` I can see the ip address of `docker0` is `172.17.0.1`

```
// HOSTS
src 172.17.0.1       // capture traffic OUTGOING FROM this ip address
dst 172.17.0.1       // capture traffic INCOMING TO this ip address
host 172.17.0.1      // capture traffic IN/OUT for this host, either direction

// NETWORKS
net 192.168.2.0/24   // monitor the whole network range
net 172.17.0.0/16    // monitor the whole network range
net 10.0.0.0/8       // monitor the whole network range

// PROTOCOLS/PORTS/BOOLEANS
// tcp, udp, icmp, etc, and, and not, ip proto 17, port 8116
tcpdump -i docker0 -v tcp and dst 172.17.0.1 

```
#### Destination example
Loading DVWA website
Note that this is the HTTP response from loading teh website (200) meaning this traffic was being returned to the listening ip address
```
$ sudo tcpdump -i docker0 -v dst 172.17.0.1

172.17.0.2.http > 172.17.0.1.43172: Flags [P.], cksum 0x639c (incorrect -> 0x260e), seq 1:2897, ack 547, win 505, options [nop,nop,TS val 3676837950 ecr 200809473], length 2896: HTTP, length: 2896
        HTTP/1.1 200 OK
        Date: Tue, 08 Apr 2025 01:49:45 GMT
        Server: Apache/2.4.25 (Debian)
        Expires: Tue, 23 Jun 2009 12:00:00 GMT
        Cache-Control: no-cache, must-revalidate
        Pragma: no-cache
        Vary: Accept-Encoding
        Content-Encoding: gzip
        Content-Length: 2659
        Keep-Alive: timeout=5, max=100
        Connection: Keep-Alive
        Content-Type: text/html;charset=utf-8
```
---

#### Source example
Loading DVWA website - note the reversing of addresses in the following compared to above, as well as the inclusion of the user-agent: header.  This is teh HTTP GET request meaning that this capture was traffic destined to the webserver at `172.17.0.2.

If I had performed the following command using the server ip `172.17.0.2` and reloaded the webpage, I would have recieved the payload from above. (The HTTP response)
```
$ sudo tcpdump -i docker0 -v src 172.17.0.1

172.17.0.1.42374 > 172.17.0.2.http: Flags [P.], cksum 0x5a6e (incorrect -> 0xc467), seq 0:546, ack 1, win 502, options [nop,nop,TS val 201175475 ecr 3677203929], length 546: HTTP, length: 546
        GET /index.php HTTP/1.1
        Host: localhost
        User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
        Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
        Accept-Language: en-US,en;q=0.5
        Accept-Encoding: gzip, deflate, br, zstd
        Referer: http://localhost/login.php
        Connection: keep-alive
        Cookie: PHPSESSID=58pcsoelmlge4kru6b5v7t5hl1; security=low
        Upgrade-Insecure-Requests: 1
        Sec-Fetch-Dest: document
        Sec-Fetch-Mode: navigate
        Sec-Fetch-Site: same-origin
        Sec-Fetch-User: ?1
        Priority: u=0, i

```

#### Host example
As above loading the DVWA webpage from the server - while monitoring the server IP.
You can see the GET request from client to server(`172.17.0.1.49676 > 172.17.0.2.http`), and the Response from teh server to the client (`172.17.0.2.http > 172.17.0.1.49676`)
```
$ sudo tcpdump -i docker0 -v host 172.17.0.2

    172.17.0.1.49676 > 172.17.0.2.http: Flags [P.], cksum 0x5a6e (incorrect -> 0x2f1b), seq 1:547, ack 1, win 502, options [nop,nop,TS val 201830448 ecr 3677858901], length 546: HTTP, length: 546
        GET /index.php HTTP/1.1
        Host: localhost
        User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
        Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
        Accept-Language: en-US,en;q=0.5
        Accept-Encoding: gzip, deflate, br, zstd
        Referer: http://localhost/login.php
        Connection: keep-alive
        Cookie: PHPSESSID=58pcsoelmlge4kru6b5v7t5hl1; security=low
        Upgrade-Insecure-Requests: 1
        Sec-Fetch-Dest: document
        Sec-Fetch-Mode: navigate
        Sec-Fetch-Site: same-origin
        Sec-Fetch-User: ?1
        Priority: u=0, i

	-- snip --

    172.17.0.2.http > 172.17.0.1.49676: Flags [P.], cksum 0x639c (incorrect -> 0xae2d), seq 1:2897, ack 547, win 505, options [nop,nop,TS val 3677858918 ecr 201830448], length 2896: HTTP, length: 2896
        HTTP/1.1 200 OK
        Date: Tue, 08 Apr 2025 02:06:46 GMT
        Server: Apache/2.4.25 (Debian)
        Expires: Tue, 23 Jun 2009 12:00:00 GMT
        Cache-Control: no-cache, must-revalidate
        Pragma: no-cache
        Vary: Accept-Encoding
        Content-Encoding: gzip
        Content-Length: 2659
        Keep-Alive: timeout=5, max=100
        Connection: Keep-Alive
        Content-Type: text/html;charset=utf-8

```

---


## Very Cool 😎🥶
Capture only HTTP GET and POST packets

```
$ sudo tcpdump -i docker0  -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420'

tcpdump: listening on docker0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
22:26:25.116292 IP (tos 0x0, ttl 64, id 55000, offset 0, flags [DF], proto TCP (6), length 598)
    172.17.0.1.47234 > 172.17.0.2.http: Flags [P.], cksum 0x5a6e (incorrect -> 0x4880), seq 3252076813:3252077359, ack 2351569263, win 502, options [nop,nop,TS val 203008748 ecr 3679037200], length 546: HTTP, length: 546
        GET /index.php HTTP/1.1
        Host: localhost
        User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
        Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
        Accept-Language: en-US,en;q=0.5
        Accept-Encoding: gzip, deflate, br, zstd
        Referer: http://localhost/login.php
        Connection: keep-alive
        Cookie: PHPSESSID=58pcsoelmlge4kru6b5v7t5hl1; security=low
        Upgrade-Insecure-Requests: 1
        Sec-Fetch-Dest: document
        Sec-Fetch-Mode: navigate
        Sec-Fetch-Site: same-origin
        Sec-Fetch-User: ?1
        Priority: u=0, i

E..V..@.@.      ............P.....*.o....Zn.....
.....I..GET /index.php HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br, zstd
Referer: http://localhost/login.php
Connection: keep-alive
Cookie: PHPSESSID=58pcsoelmlge4kru6b5v7t5hl1; security=low
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i


22:26:38.592692 IP (tos 0x0, ttl 64, id 49380, offset 0, flags [DF], proto TCP (6), length 610)
    172.17.0.1.32788 > 172.17.0.2.http: Flags [P.], cksum 0x5a7a (incorrect -> 0x0ada), seq 2242486742:2242487300, ack 60199484, win 502, options [nop,nop,TS val 203022225 ecr 3679049798], length 558: HTTP, length: 558
        GET /vulnerabilities/csrf/ HTTP/1.1
        Host: localhost
        User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
        Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
        Accept-Language: en-US,en;q=0.5
        Accept-Encoding: gzip, deflate, br, zstd
        Connection: keep-alive
        Referer: http://localhost/index.php
        Cookie: PHPSESSID=58pcsoelmlge4kru6b5v7t5hl1; security=low
        Upgrade-Insecure-Requests: 1
        Sec-Fetch-Dest: document
        Sec-Fetch-Mode: navigate
        Sec-Fetch-Site: same-origin
        Sec-Fetch-User: ?1
        Priority: u=0, i

E..b..@.@..............P.......<....Zz.....
.....I.FGET /vulnerabilities/csrf/ HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br, zstd
Connection: keep-alive
Referer: http://localhost/index.php
Cookie: PHPSESSID=58pcsoelmlge4kru6b5v7t5hl1; security=low
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i

2 packets captured
2 packets received by filter
0 packets dropped by kernel

```
