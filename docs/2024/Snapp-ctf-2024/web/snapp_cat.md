# Snapp cat

First of all when we browse the given url we're faced with a swagger documentation. 
I tried to call some of them but all seemed to require authentication except the register and login endpoints.
So I tried the register endpoint and found out that the required phone number format is as follows: +123456789012
I registered an account using the required values and logged in to the application and got a authentication token.
Afterwards, I tried to access /api/user/1 but it needed my email to be verified. I called the email verification and got the new email verified token.
Then I called the /api/user/1 endpoint and it returned some information like username and phone number of admin.
After that I called /api/user/login-with-phone endpoint with the Admin's phone number and it returned th sha256 hash of OTP.
I cracked the hash and logged in as admin user.
Now that I had the admin privilege I tried to access /api/cat/random-cat-image endpoint and it generated a uuid for me.
Then I called /api/cat/create with random values and it showed me an error saying that an error has been occured in /app/index.js. I gave the /app/index.js as input to imagePath parameter and then tried to get the file from /api/cat/{catId}.
It returned the base64 encoded index.js file. I noticed an endpoint like /secret and tried to call it but it didn't work. So I took a closer look and noticed that a claim should be set in my JWT that isn't set by default. So I took the JWT secret from index.js file and generated a new token with required property and called the secret endpoint again. 
Finally I was awarded with the flag.

# Solution

1. register
![Pasted image 20240225234831.png](./Pasted image 20240225234831.png)

2. login
![Pasted image 20240225234845.png](./Pasted image 20240225234845.png)


3. login with phone
![Pasted image 20240225234907.png](./Pasted image 20240225234907.png)

4. crack hash
![Pasted image 20240225234933.png](./Pasted image 20240225234933.png)

5. send login code
![Pasted image 20240225234953.png](./Pasted image 20240225234953.png)


6. email verification
![Pasted image 20240225235024.png](./Pasted image 20240225235024.png)


7.  get verification code from session
![Pasted image 20240225235106.png](./Pasted image 20240225235106.png)

8. send email verification code
![Pasted image 20240225235130.png](./Pasted image 20240225235130.png)

9. get admin info
![Pasted image 20240225235225.png](./Pasted image 20240225235225.png)

10. login with admin phone
![Pasted image 20240225235311.png](./Pasted image 20240225235311.png)

11. crack admin login code
![Pasted image 20240225235409.png](./Pasted image 20240225235409.png)

12. send admin login code
![Pasted image 20240225235522.png](./Pasted image 20240225235522.png)

13. become admin
![Pasted image 20240225235625.png](./Pasted image 20240225235625.png)


14. random cat
![Pasted image 20240225235932.png](./Pasted image 20240225235932.png)

15. create cat with path /etc/passwd
![Pasted image 20240226000205.png](./Pasted image 20240226000205.png)

16. get data
![Pasted image 20240226000255.png](./Pasted image 20240226000255.png)

17. /etc/passwd data read
![Pasted image 20240226000336.png](./Pasted image 20240226000336.png)

18. get /app/index.js
![Pasted image 20240226001053.png](./Pasted image 20240226001053.png)
![Pasted image 20240226001102.png](./Pasted image 20240226001102.png)
![Pasted image 20240226001204.png](./Pasted image 20240226001204.png)
![Pasted image 20240226002046.png](./Pasted image 20240226002046.png)


19 get flag with new jwt secret
![Pasted image 20240226001440.png](./Pasted image 20240226001440.png)
![Pasted image 20240226002024.png](Pasted image 20240226002024.png)
![Pasted image 20240226002102.png](Pasted image 20240226002102.png)


```
SNAPP{7dc998269394314896af6378f15c2c12}
```