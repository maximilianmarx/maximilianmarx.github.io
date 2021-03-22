---
title: "Exploit: Exploiting CVE-2016-2555 enumerating and dumping the underlying Database"
author: Maximilian Marx
date: 2021-03-21 10:38:00 +0100
categories: [SQL Injection]
tags: [SQLi, Exploit Development]

image: /assets/img/sql-injection.png
---

# Introduction

Over the last few days I was researching an (already existing) vulnerability in ATutor v2.2.1.

According to the official homepage of ATutor:
> *ATutor is an Open Source LMS (Learning Management System), used to develop and manage online courses, and to create and distribute interoperable elearning content.*

So it is an alternative LMS to Moodle.

# The vulnerability

CVE-2016-2555 describes the SQL injection vulnerability in more detail. To be more exact: *include/lib/mysql_connect.inc.php* in ATutor 2.2.1 allows remote attackers to execute arbitrary SQL commands via the searchFriends function to friends.inc.php.

It's noteworthy that this is possible without being authenticated. So everyone can exploit this vulnerability.

[**CVE Details**](https://www.cvedetails.com/cve/CVE-2016-2555/) rated the overall impact of the vulnerability with a CVSS Score of 7.5/10 

I wanted to brush up my scripting skills with Python and simultaniously deepen my knowledge in source code reviews and SQL injections (especially Blind SQL Injection). So I thought to myself, why not starting right now?

> If you want to read about blind SQL injection and what the difference to regular SQL injection is, I can recommend the according articel from [**OSWASP**](https://owasp.org/www-community/attacks/Blind_SQL_Injection)

# The exploit

So over the last few days I was developing (and for a majority debugging) an interactive Python script to enumerate and dump the underlying MySQL database.
This enabled me to extract sensitive information such as usernames and password hashes.

Since blind SQL injections are based on *TRUE/FALSE* queries (asking if a query resolves to true or false), this attacks are often more slow and dumping the complete Database would take a good amount of time, I implemented a binary search algorithm to speed things up.

On average (compared to a regular approach w/o binary search) my implementation was 10 times faster.

The complete code can be found on my [**GitHub**](https://github.com/maximilianmarx/atutor-blind-sqli)

ATutor version 2.2.1 is also vulnerable to a *Remote Code Execution* vulnerability which I'll dive into within the next days.

# Demo

In order to safe you some time, I speeded the coming GIF up twice as fast.
You should now realize how long extracting information through Blind SQL Injection takes.

![img-description](/assets/img/atutor-sqli.gif)_Proof-of-Concept Extracting the underlying Database_

# Setting up ATutor 2.2.1 on Ubuntu Server 16.04

```shell
sudo apt update
sudo apt upgrade
sudo apt-get install openssh-server
sudo apt install aptitude
sudo aptitude purge `dpkg -l | grep php| awk '{print $2}' |tr "\n" " "`
sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo reboot
sudo apt-get install php5.6 libapache2-mod-php5.6 php5.6-mcrypt php5.6-mysql php5.6-gd php5.6-mbstring
sudo apt-get install mysql-server
wget http://downloads.sourceforge.net/project/atutor/ATutor%202/ATutor-2.2.1.tar.gz?r=http%3A%2F%2Fwww.atutor.ca%2Fatutor%2Fdownload.php -O ATutor-2.2.1.tar.gz
tar -zxvf ATutor-2.2.1.tar.gz
sudo mv ATutor /var/www/html/atutor
sudo chown -R www-data: /var/www/html/atutor/
mysql_secure_installation
# -> Set new password for root
# -> Remove anonymous users? Y
# -> Disallow root login remotely? Y
# -> Remove Test DB and access to it? Y
# -> Reload privilege tables now? Y
sudo systemctl restart apache2
sudo nano /etc/mysql/my.cnf
# Add this:
[mysqld]
sql_mode="ONLY_FULL_GROUP_BY,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"
# Save and exit
sudo systemctl restart mysql

# Now visit http://localhost/atutor and start the setup
```


### Sources
- Banner <https://blogs.zeiss.com/digital-innovation/de/wp-content/uploads/sites/2/2020/05/201909_Security_SQL-Injection_1.png>