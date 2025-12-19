### cURL examples

```sh
$ curl https://jsonplaceholder.typicode.com/posts              #all posts
$ curl https://jsonplaceholder.typicode.com/posts/3            #single post
$ curl -i https://jsonplaceholder.typicode.com/posts/3         #data & headers
$ curl -I https://jsonplaceholder.typicode.com/posts/3         #just headers
$ curl -o curl.txt https://jsonplaceholder.typicode.com/posts  #output file
$ curl -O http://thesource.com/somefile.txt                    #download file

# Bearer Token
$ curl -X POST https://reqbin.com/get/json -H "Authorization: Bearer {token}"

# Cookies
$ curl -b "cookie_name=cookie_value" https://reqbin.com/echo

# PUT -d data new record
$ curl -d "id=3&body=hello World" https://jsonplaceholder.typicode.com/posts

# PUT -d update record, -X tells what http method to use
$ curl -X PUT -d "body=hello World" https://jsonplaceholder.typicode.com/posts/3

# Delete a record
$ curl -X DELETE https://jsonplaceholder.typicode.com/posts/3

# authenticated user - NOT VERIFIED, may work on FTP etc
# Doesn't seem to work on DVWA, may need auth headers
$ curl -u admin:password http://localhost/index.php

# Follow redirects -L
$ curl -L http://localhost/index.php

# upload file w/FTP with user creds -T upload local file
$ curl -u user@company.com:p@ssword! -T hello.txt ftp://ftp.company.com

# downlaod file w/FTP with user creds -O download remote file to Output
$ curl -u user@company.com:p@ssword! -O ftp://ftp.company.com/hello.txt

```
---

### `-H` - Headers: Bearer Token

To send a Bearer Token to the server using Curl, you can use the -H "Authorization: Bearer {token}" authorization header. The Bearer Token is an encrypted string that provides a user authentication framework to control access to protected resources. To send a Curl POST request, you need to pass the POST data with the -d command line option, and the authorization header and bearer token are passed with the -H command line option
```
curl -X POST https://reqbin.com/echo/get/json
     -H "Authorization: Bearer {token}"
     -d "[post data]"
```

#### Content Type
```
Content-Type: image/png
Content-Type: text/html; charset=UTF-8
Content-Type: multipart/form-data;
boundary=---Q3d4fD"
```
```
curl https://reqbin.com/echo/post/json -H 'Content-Type: application/json'
```


### `-b` - Cookies
```
$ curl -b "cookie_name=cookie_value" https://reqbin.com/echo
```


### `-L` - follow redirects
Without the -L, the following command would only return the 302. 

```
$ curl -L http://localhost/index.php

172.17.0.1 - - [08/Apr/2025:23:26:02 +0000] "GET /index.php HTTP/1.1" 302 423 "-" "curl/8.12.1-DEV"
172.17.0.1 - - [08/Apr/2025:23:26:02 +0000] "GET /login.php HTTP/1.1" 200 1937 "-" "curl/8.12.1-DEV"
```

### `-i` - show response headers
```sh
$ curl -i https://jsonplaceholder.typicode.com/posts/3

HTTP/2 200 
date: Tue, 08 Apr 2025 21:43:18 GMT
content-type: application/json; charset=utf-8
content-length: 283
report-to: {"group":"heroku-nel","max_age":3600,"endpoints":[{"url":"https://nel.heroku.com/reports?ts=1741875343&sid=e11707d5-02a7-43ef-b45e-2cf4d2036f7d&s=uNjSvwJ32d4vsIfl6%2FS%2FtjJAuroJcuc6K3sIsx%2B%2FrDI%3D"}]}
reporting-endpoints: heroku-nel=https://nel.heroku.com/reports?ts=1741875343&sid=e11707d5-02a7-43ef-b45e-2cf4d2036f7d&s=uNjSvwJ32d4vsIfl6%2FS%2FtjJAuroJcuc6K3sIsx%2B%2FrDI%3D
nel: {"report_to":"heroku-nel","max_age":3600,"success_fraction":0.005,"failure_fraction":0.05,"response_headers":["Via"]}
x-powered-by: Express
x-ratelimit-limit: 1000
x-ratelimit-remaining: 999
x-ratelimit-reset: 1741875352
vary: Origin, Accept-Encoding
access-control-allow-credentials: true
cache-control: max-age=43200
pragma: no-cache
expires: -1
x-content-type-options: nosniff
etag: W/"11b-USacuIw5a/iXAGdNKBvqr/TbMTc"
via: 1.1 vegur
cf-cache-status: HIT
age: 26150
accept-ranges: bytes
server: cloudflare
cf-ray: 92d4f0840ae91a48-EWR
alt-svc: h3=":443"; ma=86400
server-timing: cfL4;desc="?proto=TCP&rtt=19212&min_rtt=18003&rtt_var=5064&sent=7&recv=9&lost=0&retrans=0&sent_bytes=3699&recv_bytes=654&delivery_rate=239746&cwnd=252&unsent_bytes=0&cid=b29fda223c95dd9d&ts=76&x=0"

{
  "userId": 1,
  "id": 3,
  "title": "ea molestias quasi exercitationem repellat qui ipsa sit aut",
  "body": "et iusto sed quo iure\nvoluptatem occaecati omnis eligendi aut ad\nvoluptatem doloribus vel accusantium quis pariatur\nmolestiae porro eius odio et labore et velit aut"
}   
```

### Download files with curl
-o save the file to the dir/filename supplied
-O save the file using the original name to the current folder

```sh
$ curl -o myfile.txt http://thesource.com/somefile.txt
$ curl -O http://thesource.com/somefile.txt
```
---

### Proxy cURL to Burp
To have Burp capture requests from curl, use the --proxy tag:
```
curl --proxy http://127.0.0.1:8080 https://swapi.dev/api/people/1/
```
another example from TCM Lab API 0x01
```sh
$ curl -X POST -H "Content-Type: application/json" -d '{"username": "admin", "password": "password123"}' http://localhost/labs/api/login.php --proxy http://127.0.0.1:8080

{"status":"error","message":"Invalid credentials"} 
```
---

## Adding custom data in headers

Adding multiple `-H` tags in cUrl allows you to add multiple headers, the example below, I am adding a disclaimer to the headers, adding a bugHunter ID
```
# curl -X POST \
       -H "Content-Type: application/json" \
       -H "Disclaimer: BugBounty Hunter ID 2834508" \
       -d '{"username": "jeremy", "password": "cheesecake"}' http://localhost/labs/api/login.php \
       --proxy http://127.0.0.1:8080 

{"status":"success","token":"eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0=.eyJ1c2VyIjoiamVyZW15Iiwicm9sZSI6InN0YWZmIn0=."} 
```

SCREENSHOT ---- EDIT
![[Screen Shot 2025-03-18 at 5.52.47 PM.png]]



---
Some curl examples from TCM PWPA -> Autorize

### PUT request with token in Header
```
$ curl -X PUT \
       -H "Content-Type: application/json" \
       -d '{"token": "yJhbGciOiJub25lIiwidHlwIjoiSldUIn0=.eyJ1c2VyIjoiamVyZW15Iiwicm9sZSI6InN0YWZmIn0=.", \
            "username":"jeremy", \
            "bio": "New bio informationXX."}' \
        http://localhost/labs/api/account.php
```
---


### TBFT

```
# curl -X PUT https://swapi.dev/api/people -k -d '{"name":"Bob Walker"}' 
```
```
# curl https://swapi.dev/api/people/1/ 

{
   "name":"Luke Skywalker",
   "height":"172",
   "mass":"77",
   "hair_color":"blond",
   "skin_color":"fair",
   "eye_color":"blue",
   "birth_year":"19BBY",
   "gender":"male",
   "homeworld":"https://swapi.dev/api/planets/1/",
   "films":[
      "https://swapi.dev/api/films/1/",
      "https://swapi.dev/api/films/2/",
      "https://swapi.dev/api/films/3/",
      "https://swapi.dev/api/films/6/"
   ],
   "species":[],
   "vehicles":[
      "https://swapi.dev/api/vehicles/14/",
      "https://swapi.dev/api/vehicles/30/"
   ],
   "starships":[
      "https://swapi.dev/api/starships/12/",
      "https://swapi.dev/api/starships/22/"
   ],
   "created":"2014-12-09T13:50:51.644000Z",
   "edited":"2014-12-20T21:17:56.891000Z",
   "url":"https://swapi.dev/api/people/1/"
}    
```

```
$ curl --proxy http://localhost:8080 https://swapi.dev/api/people/1/ -k
{"name":"Luke Skywalker","height":"172","mass":"77","hair_color":"blond","skin_color":"fair","eye_color":"blue","birth_year":"19BBY","gender":"male","homeworld":"https://swapi.dev/api/planets/1/","films":["https://swapi.dev/api/films/1/","https://swapi.dev/api/films/2/","https://swapi.dev/api/films/3/","https://swapi.dev/api/films/6/"],"species":[],"vehicles":["https://swapi.dev/api/vehicles/14/","https://swapi.dev/api/vehicles/30/"],"starships":["https://swapi.dev/api/starships/12/","https://swapi.dev/api/starships/22/"],"created":"2014-12-09T13:50:51.644000Z","edited":"2014-12-20T21:17:56.891000Z","url":"https://swapi.dev/api/people/1/"}  
```

---

Get JWT Web tokens from accounts
```
$ curl -X POST -H "Content-Type: application/json" -d '{"username": "jeremy", "password": "cheesecake"}' http://localhost/labs/api/login.php

{   "status":"success",
    "token":"eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0=.eyJ1c2VyIjoiamVyZW15Iiwicm9sZSI6InN0YWZmIn0=."
}                                          
```
```
$ curl -X POST -H "Content-Type: application/json" -d '{"username": "jessamy", "password": "tiramisu"}' http://localhost/labs/api/login.php

{  "status":"success",
   "token":"eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0=.eyJ1c2VyIjoiamVzc2FteSIsInJvbGUiOiJhZG1pbiJ9."
}  
```

**Jeremy JWT**
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0=.eyJ1c2VyIjoiamVyZW15Iiwicm9sZSI6InN0YWZmIn0=.
**Jessamy JWT**
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0=.eyJ1c2VyIjoiamVzc2FteSIsInJvbGUiOiJhZG1pbiJ9.

```
$ curl -X POST -H "Content-Type: application/json" -d '{"username": "jeremy", "password": "cheesecake"}' http://localhost/labs/api/login.php

{
    "status":"success",
    "token":"eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0=.eyJ1c2VyIjoiamVyZW15Iiwicm9sZSI6InN0YWZmIn0=."
}
```
```
$ curl -X GET "http://localhost/labs/api/account.php?token=eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0=.eyJ1c2VyIjoiamVyZW15Iiwicm9sZSI6InN0YWZmIn0=."

{
	"username":"jeremy",
	"role":"staff",
	"bio":"Java programmer."
}     
```
```
$ curl -X PUT -H "Content-Type: application/json" \
    -d '{"token": "yJhbGciOiJub25lIiwidHlwIjoiSldUIn0=.eyJ1c2VyIjoiamVyZW15Iiwicm9sZSI6InN0YWZmIn0=.", \
           "username":"jeremy",
           "bio": "New bio informationXX."
         }'
    http://localhost/labs/api/account.php

{
   "status":"success",
   "message":"Bio updated successfully"
}

```               
```

$ curl -X GET "http://localhost/labs/api/account.php?token=eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0=.eyJ1c2VyIjoiamVyZW15Iiwicm9sZSI6InN0YWZmIn0=."                                                                                                   
{
	"username":"jeremy",
	"role":"staff",
	"bio":"New bio informationXX."
} 

```
