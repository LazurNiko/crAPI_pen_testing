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
> ## TC#01 Broken Object Level Authorization 
>
>> ### 'User should access the restricted page of another user using id parameter'
>
> Scope:
>
```
GET /identity/api/v2/videos/:id?video_id={value}`
GET /community/api/v2/community/posts/{value}`
GET /identity/api/v2/vehicle/{value}/location`
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
> ## TC#01 Broken Function Level Authorization 
>
>> ### 'Another user shouldn't be able to edit the content of another user'
>
> Scope:
>
```
POST /workshop/api/shop/orders/return_order?order_id={id}
POST /community/api/v2/community/posts/{id}/comment
PUT /identity/api/v2/user/videos/id
DELETE /identity/api/v2/user/videos/id
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