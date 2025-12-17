```
        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/
```
# FFUF – Practical Usage Notes

These notes document **hands-on FFUF usage** during authorized lab testing.
They focus on real workflows: starting from a known request (often captured in
Burp), then using FFUF for **fast, repeatable fuzzing** with effective filtering.

Burp is often easier for exploration and understanding application behavior.
Once the request is understood, FFUF is typically faster - especially for
throttled dictionary attacks, API fuzzing, and large wordlists.

---

## Core Concepts

* **Start from a known request**: Capture a working request (Burp → Save raw request).
* **Fuzz one variable at a time**: Replace the target value with `FUZZ`.
* **Filter first, then match**: Identify the “noise” response (size/words/lines),
  cancel the run, and re-run with filters or matchers.
* **Proxy when needed**: Route traffic through Burp for inspection or modification.

---

## Common & Important Switches

`-u` (URL) and `-w` (wordlist) are required for most runs.

```bash
# ffuf -h

HTTP OPTIONS:
  -H     Header "Name: Value" (multiple allowed)
  -X     HTTP method
  -b     Cookie data "NAME=VALUE; NAME2=VALUE2"
  -d     POST data
  -u     Target URL
  -x     Proxy URL (SOCKS5 or HTTP)

GENERAL OPTIONS:
  -c       Colorize output
  -json    JSON output (newline-delimited)
  -t       Number of concurrent threads
  -v       Verbose output

MATCHER OPTIONS:
  -mc      Match HTTP status codes (or "all")
  -ml      Match number of lines
  -mr      Match regular expression
  -ms      Match response size
  -mt      Match time to first byte (ms)
  -mw      Match number of words

FILTER OPTIONS:
  -fc      Filter status codes
  -fl      Filter number of lines
  -fr      Filter regexp
  -fs      Filter response size
  -ft      Filter time to first byte
  -fw      Filter number of words

INPUT OPTIONS:
  -mode            clusterbomb | pitchfork | sniper
  -request         Raw HTTP request file
  -request-proto   Protocol to use with raw request (default: https)
  -w               Wordlist path

OUTPUT OPTIONS:
  -o      Output file
  -od     Output directory
  -of     Output format (json, html, md, csv, all)
```

---

## Proxying Through Burp

Route FFUF traffic through Burp for inspection or manual modification.

```bash
ffuf -x http://127.0.0.1:8080 \
     -w wordlist.txt \
     -u https://ffuf.io.fi/FUZZ
```

---

## Fuzzing with Request Bodies

### JSON Body (Content-Type Required)

```bash
ffuf -w sqli_payloads.txt \
     -u https://ffuf.io.fi/api/v1/users/1 \
     -X PUT \
     -H 'Content-Type: application/json' \
     -d '{"uid":"FUZZ"}'
```

---

## Query Parameter Fuzzing

> Note the escaped `?` and `&` characters.

```bash
ffuf -u http://10.0.2.6/dvwa/vulnerabilities/brute/\
?username=admin\&password=FUZZ\&Login=Login \
     -w /usr/share/seclists/Passwords/xato-net-10-million-passwords-100.txt
```

---

## API Login Fuzzing (POST)

```bash
ffuf -u http://<target>/login \
     -X POST \
     -d '{"username":"admin", "password":"FUZZ"}' \
     -w auth_bypass.txt
```

---

## Raw Request-Based Fuzzing

### API Endpoint Testing

```bash
ffuf -request api_req.txt \
     -request-proto http \
     -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt \
     -fw 19,20,170
```

### Discover API Files / Endpoints

```bash
ffuf -u http://localhost/labs/api/FUZZ \
     -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt \
     -e .php,.html,.txt \
     -fc 403
```

---

## Filtering & Matching Strategy

Typical workflow:

1. Run without filters
2. Observe invalid response characteristics (size/words/lines)
3. Cancel
4. Re-run with filters or matchers

```bash
# Filter by response size
ffuf -request request.txt \
     -request-proto http \
     -w passwords.txt \
     -fs 1814

# Match on keyword
ffuf -request idor_req.txt \
     -request-proto http \
     -w numbers.txt \
     -mr "admin"
```

---

## File Extension Enumeration

```bash
ffuf -u http://10.0.2.6/dvwa/FUZZ \
     -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt \
     -e .php,.html,.txt
```

---

## Multi-Wordlist Attacks

### Modes
Modes:
- clusterbomb
- pitchfork
- sniper

```bash
ffuf -request req2.txt \
     -request-proto http \
     -mode clusterbomb \
     -w names.txt:FUZZUSER \
     -w pass.txt:FUZZPASS \
     -mr "Successfully logged in"
```

---

## Enumeration

### URL Endpoints

```bash
ffuf -u http://localhost:3000/FUZZ \
     -w /usr/share/seclists/Discovery/Web-Content/big.txt
```

### Files

```bash
ffuf -u http://10.0.2.6/dvwa/FUZZ \
     -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt \
     -e .php,.html,.txt \
     -mc 200
```

### Subdomains

```bash
ffuf -u http://FUZZ.10.0.2.6/dvwa/ \
     -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

---

## Authentication Attacks

### Username Enumeration (POST)

```bash
ffuf -w /usr/share/seclists/Usernames/top-usernames-shortlist.txt \
     -X POST \
     -d "username=FUZZ&&password=x" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -u http://10.0.2.6/dvwa \
     -mr "username doesn't exist"
```

---

## HTTP GET – No Raw Request (Cookies Inline)

```bash
ffuf -u http://10.0.2.6/dvwa/vulnerabilities/brute/\
?username=FUZZUSER\&password=FUZZPASS\&Login=Login \
     -w Lab_names.txt:FUZZUSER \
     -w passwords.txt:FUZZPASS \
     -fr "Username and/or password incorrect." \
     -fc 302 \
     -b "security=low; PHPSESSID=0b6f44bf97816fc76cad801e9723d6fa"
```

---

## HTTP GET – Raw Request

### Single Wordlist

```bash
ffuf -request DVWA_GET_login_request.txt \
     -request-proto http \
     -w passwords.txt \
     -mr "Welcome to the password protected area"
```

### Two Wordlists

```bash
# Separate -w flags
ffuf -request DVWA_GET_login_request.txt \
     -request-proto http \
     -w passwords.txt:FUZZPASS \
     -w top20_names.txt:FUZZUSER \
     -mr "Welcome to the password protected area"

# Comma-separated
ffuf -request DVWA_GET_login_request.txt \
     -request-proto http \
     -w passwords.txt:FUZZPASS,top20_names.txt:FUZZUSER \
     -mr "Welcome to the password protected area"
```

---

## HTTP POST – Raw Request

```bash
ffuf -request LABS_POST_login_request.txt \
     -request-proto http \
     -w passwords.txt:FUZZPASS \
     -w top20_names.txt:FUZZUSER \
     -mr "You have successfully logged in"
```

---

## HTTP POST – No Raw Request

### Single Wordlist

```bash
ffuf -w passwords.txt \
     -X POST \
     -d "username=jeremy&&password=FUZZ" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -u http://localhost/labs/a0x01.php \
     -mr "You have successfully logged in"
```

### Two Wordlists

```bash
ffuf -w passwords.txt:FUZZPASS,Lab_names.txt:FUZZUSER \
     -X POST \
     -d "username=FUZZUSER&password=FUZZPASS" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -u http://localhost/labs/a0x01.php \
     -mr "You have successfully logged in"
```

