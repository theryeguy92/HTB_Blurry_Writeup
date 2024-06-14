# Blurry Writeup

# Intro
Blurry is an interesting HTB machine where you will leverage the CVE 2024-24590 exploit to pop a reverse shell in order to escalate your privlages within the local network. This box uses ClearML, an open-source machine learning platform that allows its users to streamline the machine learning lifecycle. ClearML is used by many Data Engineers and Data Scientist.

Below you can find of the tools that I used to complete this challenge
1. Kali Linux: An operating system that specializes in penetration testing.
2. Nmap: An open-source toolf for network exploration, along with security auditing
3. Clearml python libraries: These libraries are used to interact with th open-source ClearML that we plan to hack into. I will go more into these libraries within the writeup.

I will go into detail regarding the steps taken. First we will go over the initial reconnaissance, identifying avenues of exploitation, exploitation foothold, then post exploitation.

### Initial Reconnisance
As with any initial penetration test, we will start with a nmap scan.

```bash
┌──(kali㉿kali)-[~]
└─$ nmap -v -sV -sC -p- 10.10.11.19                               
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey: 
|   3072 3e:21:d5:dc:2e:61:eb:8f:a6:3b:24:2a:b7:1c:05:d3 (RSA)
|   256 39:11:42:3f:0c:25:00:08:d7:2f:1b:51:e0:43:9d:85 (ECDSA)
|_  256 b0:6f:a0:0a:9e:df:b1:7a:49:78:86:b2:35:40:ec:95 (ED25519)
80/tcp open  http    nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-favicon: Unknown favicon MD5: 2CBD65DC962D5BF762BCB815CBD5EFCC
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-title: ClearML
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

}
```

We see that we have ports 22 and 80 open. Also that that it is a Linux machine. To get more info, we will visit the page at blurry.htb or 10.10.11.19 within the web browser.


![appblurryhtb login](https://github.com/theryeguy92/HTB_Blurry_Writeup/assets/103153678/bf5b7bc6-a7f4-4fb8-ba1f-4367812385b5)


Here we are able to make a user name of our choosing and log into ClearML. You will be greated with a page regarding instructions of how to use clear ML along with creating your own credentials.

To do so, we will need to pip install the clearml libraries via the instrictions below.

![credentials](https://github.com/theryeguy92/HTB_Blurry_Writeup/assets/103153678/a2029720-dcc0-4de9-ae8a-5b35b59ee2af)



One thing to note, to be sure that the libaries are installed correctly, I ran a pip install command on the following:

```bash
pip install clearml
pip install clearml-init
pip install clearml-agent

}
```
For some reason the origional pip isntall ClearML did not properly install the clearml-init and clearml-agent for me. If you run into issues, I reccommend retying to pip install these.


### Avenues of Exploitation

After playing around with the website, and researching, I found that ClearML has a vulnerability (CVE2024-24590). This vulnerability can allow a user to run commands via the pickle python library.

The pickle library is used by many python users. The uses range from data transfer, saving a program state, or caching. From my experience as a data analyst/data scientist, the pickle library has many uses.

However in this case, we will use it as an exploit. Below is the python payload that I used.

![python payload CVE 2024 24590](https://github.com/theryeguy92/HTB_Blurry_Writeup/assets/103153678/04e406a0-42c0-40dc-8ff7-d8e9b33adcec)


### Exploitation Foothold

After the payload is created, we will run the the following command. This will start the ClearML agent to upload our malicious code to the ClearML site.

```bash
clearml-agent daemon --queue default
}
```

After this is ran, we will set up a listener (I put my listener on port 4444)

```bash
nc -lvnp 4444
}
```

Then execute the python payload

```bash
python3 <your python file payload>.py
}
```

NOTE: At least in my case, it took a while for me to establish a connection. I recommend getting a cup of coffee and coming back. After, you should see that we that we have a reverse shell.

![reverse shell](https://github.com/theryeguy92/HTB_Blurry_Writeup/assets/103153678/b403c153-9b87-4527-9fc2-e7547d2e6af2)


### Post Exploitation

Now we are in the system and gained a foothold. It is good practice to see who we are, and if the user we are logged into has any root permissions. 

As seen below, we are the user jippity and they are able to run sudo privilages on the file path /user/bin/evaluate_model /models/*.pth. This means that we are able to run sudo level commands in this directory.

Before we do that, we can explore the directory to find the user flag. You should be able to find pretty easily as you navigate through the user files.

![userflag](https://github.com/theryeguy92/HTB_Blurry_Writeup/assets/103153678/721c2932-9a6a-49d3-b8e3-6ba81fed24ee)


After we get the userflag, our next target is the root flag. Since we know that the user we are logged in as (jippity) has sudo privlages specified in the file path above, we can navigate to the /models directory and see what is in there.

As we will see, they have two files, one being a demo_model.pth and a evaluate_model.py. 






