## Postman Field Guide
## Subgroup 1
#### Parse Returned JSON
Example response
```json
{
  "user": { "id": 12, "name": "Bubba" },
  "token": "abc123"
}
```
Postman post response script
```js
// Parse JSON
let json = pm.response.json();
console.log("Parsed JSON:", json);

// Access fields
let userId = json.user.id;
let token = json.token;

// Add variables
pm.environment.set("userId", userId);
pm.environment.set("token", token);

// Basic test
pm.test("Token exists", () => {
    pm.expect(token).to.be.a("string");
});
```
---
#### Parse returned JSON, select spcecific object
Example response
```json
{
  "user": { 
	  "id": 12, 
	  "name": "Jason" 
	},
  "user": { 
	  "id": 13, 
	  "name": "Bob" 
	},
  "token": "abc123"
}
```
Postman post response script
```js
// Parse JSON
let json = pm.response.json();
console.log("Parsed JSON:", json);

let targetId = 13;
const foundItem = json.find(item => item.id === targetId);

if (foundItem) {
	pm.test("Found item with targetId of ${targetId}", () => {
		pm.expect(foundItem).to.be.an.object;
	});	
	pm.test("Message is a string", () => {
		pm.expect(foundItem.message).to.be.a("string");
	});	
} else {
	pm.test("Item with targetId of ${targetId} not found", () => {
		pm.expect(foundItem).to.be.undefined;
	});	
}
```
---
#### Parse returned XML
Example response
```xml
<response>
  <status>success</status>
  <data>
    <id>88</id>
    <name>Chatter App</name>
  </data>
</response>
```
Postman post response script
```js
// Convert XML -> JavaScript object
let xmlData = xml2Json(pm.response.text());
console.log("Parsed XML:", xmlData);

// Access values
let status = xmlData.response.status;
let id = xmlData.response.data.id;

// Save variables
pm.environment.set("xmlStatus", status);
pm.environment.set("xmlID", id);

// Test
pm.test("Status is success", () => {
    pm.expect(status).to.eql("success");
});
```
---
#### Get cookies, headers, session info
**Cookies:**
```js
let cookies = pm.cookies.toObject();
console.log("Cookies:", cookies);

// Get a specific cookie
pm.environment.set("sessionId", pm.cookies.get("sessionid"));
```
**Headers:**
```js
let headers = pm.response.headers.toObject();
console.log("Headers:", headers);

let contentType = pm.response.headers.get("Content-Type");
pm.environment.set("contentType", contentType);
```
**Extract Bearer Token from Authorization header:**
```js
let auth = pm.response.headers.get("Authorization");    // e.g. "Bearer xyz"
let token = auth ? auth.replace("Bearer ", "") : null;

pm.environment.set("authToken", token);
console.log("Extracted Token:", token);
```

Chatter example
The following sets the global variable "Chatter_bearertoken" to the token recieved in this request. It can then be used in future requests (for example in the auth tab)
with the following `{{Chatter_bearertoken}}`
```
let json = pm.response.json();
let token = json.token;
pm.globals.set("Chatter_bearertoken", token)
```
---
#### Write Values to Variables (Global, Environment, Local)
**Environment variable:**
```js
pm.environment.set("apiKey", "12345");
```
**Global variable:**
```js
pm.globals.set("globalUserId", 99);
```
**Local variable (only during this request run):**
```js
pm.variables.set("tempVar", "temporary");
```

**Delete variables:**
```js
pm.environment.unset("apiKey");
pm.globals.unset("globalUserId");
```
---
#### Logging to Postman Console
```js
console.log("User ID:", pm.environment.get("userId"));
console.warn("Warning message");
console.error("Error message");
```
---
#### Write Output to a File (txt, json, csv)
 Postman itself cannot write directly to disk, but Newman (Postman CLI) can.
**Postman post response script:**
```js
let data = {
    time: new Date().toISOString(),
    user: pm.environment.get("userId"),
    token: pm.environment.get("token")
};

pm.environment.set("exportData", JSON.stringify(data));
```
**Run with Newman:**
```sh
# newman run collection.json --reporters cli,json \
  --reporter-json-export output.json
```
Append output to CSV (via Newman + custom reporters)
Inside Tests:
```js
pm.environment.set("csvLine",`${pm.environment.get("userId")},${pm.environment.get("token")}`);
```
Then Newman reporter writes to CSV.

---
#### Full Example
Parse JSON, Extract Token, Save to Var, Log to console, test
Postman post response script:
```js
let body = pm.response.json();

pm.test("Status is 200", () => {
    pm.response.to.have.status(200);
});

// Extract values
pm.environment.set("token", body.token);
pm.environment.set("userName", body.user.name);

console.log("Token:", body.token);
console.log("User:", body.user.name);

// Verify token exists
pm.test("Token present", () => {
    pm.expect(body.token).to.be.a("string");
});
```
#### XML Example
Parse XML, Extract, Save to Var, Log to console, test
Postman post response script:
```js
let xmlObj = xml2Json(pm.response.text());

let name = xmlObj.response.data.name;

pm.environment.set("xmlName", name);

console.log("Name:", name);

pm.test("Name exists", () => {
    pm.expect(name).to.not.be.undefined;
});
```
---

#### Use a Local File as a Wordlist
Postman can read an uploaded file before sending the request.

Prerequisites:
	- Go to Body → form-data
	- Create a key called wordlist
	- Set type to File
	- Attach your text file.

File example (wordlist.txt):
```txt
admin
root
test
guest
```
**Pre-request script:**
Read file, split into lines, store variable
```js
let file = pm.request.body.formdata.find(i => i.key === 'wordlist');

if (!file) {
    throw new Error("No file uploaded under form-data key 'wordlist'");
}

// Convert to string
let content = file.src.toString();
let lines = content.split(/\r?\n/).filter(l => l.trim().length > 0);

// Pull index
let index = parseInt(pm.environment.get("wordIndex") || "0");

// Get current word
let current = lines[index];

// Set environment variable
pm.environment.set("currentWord", current);

// Log it
console.log("Using word:", current);

// Increment index
if (index + 1 < lines.length) {
    pm.environment.set("wordIndex", index + 1);
} else {
    pm.environment.set("wordIndex", 0); // loop around
}
```
Use in request URL or body:
```
{{currentWord}}
```
---
#### Pre-Request Script that goes through a List (wordlist without external file)
Good for testing endpoints with multiple IDs, usernames, tokens, etc.
Create in environment variable:
```js
wordlist = ["admin","root","guest","test"]
```
Pre-request script:
```js
let list = JSON.parse(pm.environment.get("wordlist"));
let idx = Number(pm.environment.get("wordIndex") || 0);

pm.environment.set("currentItem", list[idx]);

console.log("Using item:", list[idx]);

idx++;
if (idx >= list.length) idx = 0;
pm.environment.set("wordIndex", idx);
```
---
#### Pre-Request Script to Generate Dynamic Tokens
Used in chained authentication sequences.
```js
let timestamp = Date.now();
let random = Math.random().toString(36).substring(2, 8);

let token = `${timestamp}-${random}`;

pm.environment.set("dynamicToken", token);
console.log("Generated token:", token);
```
---
#### Pre-Request Script that Depends on Previous Response
Classic “chaining requests” pattern
```js
let lastToken = pm.environment.get("authToken");
if (!lastToken) {
    throw new Error("authToken not found — run login request first.");
}

pm.request.headers.add({
    key: "Authorization",
    value: `Bearer ${lastToken}`
});
```
---
#### Pre-Request Script for Hashing (MD5, SHA, HMAC)
Useful for APIs requiring signatures.
```js
const message = pm.environment.get("currentWord") || "default";

let hash = CryptoJS.SHA256(message).toString();

pm.environment.set("sha256Hash", hash);
console.log("Hash:", hash);
```
---
#### Pre-Request Script to Call Another Request (manual)
You can’t auto-trigger another tab,
but you can perform internal API calls using pm.sendRequest.
Example: Fetch a token before the main request executes.
```js
pm.sendRequest({
    url: "https://example.com/api/auth",
    method: "POST",
    body: {
        mode: "raw",
        raw: JSON.stringify({ user: "test", pass: "123" })
    }
}, (err, res) => {
    if (err) throw err;

    let json = res.json();
    pm.environment.set("apiToken", json.token);

    console.log("Fetched token:", json.token);
});
```
---
#### Pre-Request Script to Delay (rate limiting / brute force pacing)
Blocking sleep is the only way inside Postman
```js
function sleep(ms) {
    const start = Date.now();
    while (Date.now() - start < ms) {}
}

sleep(1500); // 1.5 seconds
console.log("Paused for 1500ms");
```
---
8











 










