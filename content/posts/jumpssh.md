---
title: "How to access a network device through a jump server using JumpSSH"
date: 2020-09-15T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["python", "technocal"]
categories: ["Networking"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/<path_to_repo>/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

A jump host is a server inside a secure zone, that you access from a less secure zone. You can then jump from this host to greater security zones. An example would be a high security zone inside a corporation.

in this tutorial we are going to get a backup of a router configuration through a jumpserver using ***[JumpSSH](https://github.com/AmadeusITGroup/JumpSSH)***
***JumpSSH*** is a module for Python 2.7+/3.4+ that can be used to run commands on remote servers through a gateway.
installing the required packages
```BASH
pip install jumpssh 
pip install python-dotenv
```
we will use [python-dotenv](https://github.com/theskumar/python-dotenv) to manage our credentials to environment variable.

creating a .env file in the same path of our script
```BASH
jumpserver_ip = 'your_jump_server_ip' 
jumpserver_username = 'your_jump_server_username' 
jumpserver_password = 'your_jump_server_password' 
remote_ip = 'your_remote_node_ip' 
remote_username = 'your_remote_node_username' 
remote_password = 'your_remote_node_password'
```
then creating our script
```PYTHON
from dotenv import load_dotenv
import jumpssh 
import 

os load_dotenv() 

jumpserver_ip = os.getenv("jumpserver_ip") 
jumpserver_username = os.getenv("jumpserver_username") jumpserver_password = os.getenv("jumpserver_password") 
remote_ip = os.getenv("remote_ip") 
remote_password = os.getenv("remote_password") 

gateway_session = SSHSession(jumpserver_ip, jumpserver_username, password=jumpserver_password).open() 
remote_session = gateway_session.get_remote_session(remote_ip, password=remote_password, allow_agent=False, look_for_keys=False) 

with open('R1.txt', 'w') as fl:
	fl.write(remote_session.get_cmd_output('show running config')) remote_session.close() 


gateway_session.close()
```