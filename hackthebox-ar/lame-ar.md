---
description: >-
  الماشين بتركز على المبتدئين و على مبادئ اساسية باختبار الاختراق متل انك تعمل
  سكان بشكل صح و انك تبحث عن كل اصدار حاص بكل خدمة متواجدة على السيرفر و تشوف
  ازا مصابة باي نوع من الثغرات او الها اي استغلال
---

# Lame - بالعربي

![](../.gitbook/assets/image%20%2847%29.png)

  
السلام عليكم و رحمة الله و بركاته  
  
1- تبدأ الاول بعمل Scan على الماشين ب nmap عشان نشوف ال Open Ports الموجودة و ال services/الخدمات و شو منهم فيه ثغرة/vulnerable فتستغله و طبعا بدون عمل استطلاع/recon او enumeration ما ممكن تبدأ اي عملية Pentesting او Hacking  
  


```text
nmap -sV -sC  10.10.10.3 -oA lame -Pn
```

```text
Starting Nmap 7.80 ( https://nmap.org ) at 2020-11-08 01:53 EST
Nmap scan report for 10.10.10.3
Host is up (0.17s latency).
Not shown: 996 filtered ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.6
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h32m43s, deviation: 3h32m09s, median: 2m42s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2020-11-08T01:56:09-05:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 64.23 seconds

```

متل ما  شايف في عنا اكتر من شي مثير للاهتمام  


{% hint style="info" %}
  
1-   ال  vsftpd 2.3.4 و هاد مصاب ب backdoor command execution \([اضغط هنا](https://www.rapid7.com/db/modules/exploit/unix/ftp/vsftpd_234_backdoor)\)  
انا حاولت استغله سواء بال metasploit او اكتب الاستغلال/ exploit بايدي ومانفع  
و وفقا للمقالة هي \([اضغط هنا](https://www.hackingtutorials.org/metasploit-tutorials/exploiting-vsftpd-metasploitable/)\) ف الي بحصل ان ال backdoor بال vsftp ال function بتعمل check على رقمين hexadecimal الي هنن 0x3a و  0x29  
ازا حولنا الرقمين ل ASCII رح نشوف ان 0x3a = : و 0x29 = \)  
ف لما بتجمعن بيعطي هاد الوجه :\) , فاذا لقيت ال function هدول الاتنين مع اسم المستخدم / username الي رح ادخله بتشتغل هي ال function vsf\_sysutil\_extra  
هي لما بتشتغل بتفتح 6200 port و بنفس الوقت بقبل اي ip يتصل عليه لان هو اصلا فاتح bind socket و بعمل استماع / listen على اي اتصال بحصل على  هاد ال port و بنفس ال function بيعطي shell لاي حدا بتصل عليه.  
ف نحن ممكن نستغلها يدوي بخطوتين كلاتي :   
  


![](../.gitbook/assets/image%20%2832%29.png)

هون عم حط ال :\) كرمال يفتح ال port 6200 و يبدأ يستمع   
ازا كان الاستغلال شغال و ال vsftp هاد مصاب لازم بالخطوة التانية يتصل و يعطيني shell كمان , بس الي عم يحصل ان ما عم يقدر يتصل عليه  


![](../.gitbook/assets/image%20%2828%29.png)

انا كرمال اتاكد من الي بحصل عملت scan بال nmap على ال port 6200   


![](../.gitbook/assets/image%20%2825%29.png)

هاد بعني ان احتمال كبير في جدار حماية / firewall , انا مارح جرب اتخطاه / bypass لانو رح يطول كتير الشرح , ف رح انتقل للبعده
{% endhint %}

2- عندك ال ssh بس انا بعرف ان مافيها شي ف روح للبعدها

3- ال samba ازا عملت بحث عن Samba smbd 3.X - 4.X exploit  
رح تلاقي هيدا الاستغلال \([اضغط هنا](https://www.exploit-db.com/exploits/16320)\)  
طبعا الامر سهل جدا بتفتح ال metasploit و بتعمل بحث عن "Username map script Command Execution"   


![](../.gitbook/assets/image%20%2817%29.png)

حاليا بس بتعمل use لل exploit و بتحط ال options المذكورة و بتعمل run   
  
رح تلاقي نفسك بصلاحيات ال root و بتعمل  


> cat makis/user.txt

بتحصل ال user flag و بتعمل

> cat /root/root.txt

و هيك بتحصل ال root flag 

طيب حاليا مبروك عليك اتحلت ال machine , بس ما خلص المقال في شوية شرح لل samba exploit الي حصل



{% hint style="info" %}
وفقا للمقال هاد \([اضغط هنا](https://amriunix.com/post/cve-2007-2447-samba-usermap-script/)\) بحصل الاتي :   
  
المشكلة الاساسية بانو ال 

> `./source/lib/username.c`

بتاخد اي ادخال / input من المستخدم / user و ما بحصلها filtering  او هي unfiltered و بعدين بنبعت ال input هاد لل function هي 

> `./source/lib/smbrun.c`

و هي الي بتنفذ الاوامر , بالنظر لل exploitation script \([اضغط هنا](https://raw.githubusercontent.com/amriunix/CVE-2007-2447/master/usermap_script.py)\) بتم ايضاح الفكرة اكتر و ايضا مطابقة للشرح.
{% endhint %}

بهيك خلصت و السلام عليكم و رحمة الله و بركاته.

