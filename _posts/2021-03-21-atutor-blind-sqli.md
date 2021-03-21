---
title: "Exploit: Exploiting CVE-2016-2555 enumerating and dumping the underlying Database"
author: Maximilian Marx
date: 2021-03-21 10:38:00 +0100
categories: [Red Teaming]
tags: [Malware, Exploit Development, Evasion]
---math: true
mermaid: true
---image: /assets/img/studiumplus.svg
---

Over the last few days I was researching an (already existing) vulnerability in ATutor v2.2.1.

According to the official homepage of ATutor:
> *ATutor is an Open Source LMS (Learning Management System), used to develop and manage online courses, and to create and distribute interoperable elearning content.*

So it is an alternative LMS to Moodle.

CVE-2016-2555 describes the SQL injection vulnerability in more detail. To be more exact: *include/lib/mysql_connect.inc.php* in ATutor 2.2.1 allows remote attackers to execute arbitrary SQL commands via the searchFriends function to friends.inc.php.

It's noteworthy that this is possible without being authenticated. So everyone can exploit this vulnerability.

[**CVE Details**](https://www.cvedetails.com/cve/CVE-2016-2555/) rated the overall impact of the vulnerability with a CVSS Score of 7.5/10 

I wanted to brush up my scripting skills with Python and simultaniously deepen my knowledge in source code reviews and SQL injections (especially Blind SQL Injection). So I thought to myself, why not starting right now?

> If you want to read about blind SQL injection and what the difference to regular SQL injection is, I can recommend the according articel from [**OSWASP**](https://owasp.org/www-community/attacks/Blind_SQL_Injection)


So over the last few days I was developing (and for a majority debugging) an interactive Python script to enumerate and dump the underlying MySQL database.
This enabled me to extract sensitive information such as usernames and password hashes.

Since blind SQL injections are based on *TRUE/FALSE* queries (asking if a query resolves to true or false), this attacks are often more slow and dumping the complete Database would take a good amount of time, I implemented a binary search algorithm to speed things up.

On average (compared to a regular approach w/o binary search) my implementation was 10 times faster.

The complete code can be found on my [**GitHub**](https://github.com/maximilianmarx/atutor-blind-sqli)

ATutor version 2.2.1 is also vulnerable to a *Remote Code Execution* vulnerability which I'll dive into within the next days.