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









