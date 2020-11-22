---
description: حل legacy box
---

# Legacy - بالعربي

![](../.gitbook/assets/image%20%2824%29.png)

  
  
السلام عليكم و رحمة الله و بركاته  
  
1- تبدأ الاول بعمل Scan على الماشين ب nmap عشان نشوف ال Open Ports الموجودة و ال services/الخدمات و شو منهم فيه ثغرة/vulnerable فتستغله و طبعا بدون عمل استطلاع/recon او enumeration ما ممكن تبدأ اي عملية Pentesting او Hacking  


![](../.gitbook/assets/image%20%2841%29.png)

2- نبدأ بالبحث اذا كان اي خدمة / service مصابة و اول شيء لدينا هو ال 139 & 445   
بعد شوية بحث عن الثغرات الي ممكن يكون ال smb مصاب فيها و خاصة الي بتستهدف win xp لقيت ms08-067 و ممكن تتسغلها بال metasploit  بسهولة جدا  
  


```text
use exploit/windows/smb/ms08_067_netapi
```

بعد ما تزبط ال options و تعمل run المفروض يظهرلك كلاتي : 

![](../.gitbook/assets/image%20%2829%29.png)

ما في اي داعي لل privesc هون لان انت بالاصل بسبب الاستغلال هاد بتدخل ك root 

هلئ نحصل ال root flag 

```text
type Administrator\Desktop\root.txt
```

و الان ال user flag

```text
type john\Desktop\user.txt
```

بهيك انتهت الماشين بس كيف حصل الاستغلال / exploit ؟!!!

{% hint style="info" %}
الامر كله بسبب خطأ بمعالجة / handling طلب / request بال RPC service !

بحصل عن طريق اي طلب خاص بال UUID 

4B324FC8-1670-01D3-1278-5A47BF6EE188

ال RPC فيه function اسمها NetprPathCanonicalize و الي بدورها بتستخدم ال library NETAPI32.DLL  
الكود :

`long NetprPathCanonicalize(  
[in] [string] [unique] wchar_t *ServerName,  
[in] [string] [ref] wchar_t *PathName,  
[in] long OutBufLen;  
[in] [string] [ref] wchar_t *Prefix,  
[in] [out] [ref] long *PathType;  
[in] long PathFlags;  
);`

المشكلة بان المتغير variable الي اسمه PathName ما بتم معالجته بشكل صح فبسبب 

buffer overflow

 هو بياخد  ال path بالشكل الاتي : 

/pathpart1/../../pathpart2

فبياخد منه على قدر ال calculated buffer فالمفروض ان رح ياخد منه حد معين و الباقي يكون كلاتي

/../pathpart2

بمعنى اخر هو اخد هي المقدمة

/pathpart1/..

طيب هون لما رح يرجع ياخد التاني بما ان مافي شي قبل ال path او ال input ف رح يبحث عن اول /  بهي الحال و حسب فهمي رح يكون ال / بمكان بالذاكرة / memory عند شغلة اسمها ال designated buffer 

فلو اجا ال / بال stack على مكان vulnerable  فرح يحصل نسخ لل source string للمكان الي هو واقف عليه.

طيب لحد الان هي تعتبر الثغرة بدون الاستغلال , فكيف بستغلها المهاجم ؟ 

بيقدر المهاجم يتلاعب بال stack عن طريق ان يرسل الطلب / request  مرتين او يعمل call على ال RPC مرتين 

بالمرة التانية رح يحصل overwrite لل stack buffer . 
{% endhint %}

كنت بتمنى اقدر اعمل الامر عملي بشكل يدوي اكتر و احللللكم \(يا اخي اخت الكلمة ما بقى اعرف اكتبا ولا الفظها 😂\) الي بحصل مع توضيح اكتر و اكتب python script يستغل الموضوع , بس للاسف رح ياخد وقت, فبتمنى للوقت الحالي يكون هيك الشرح قرب فكرة ال exploit و شو بصير ورا ال Enter الي بتضغطها.

السلام عليكم.

Resources:

[https://labs.f-secure.com/assets/BlogFiles/hello-ms08-067-my-old-friend.pdf](https://labs.f-secure.com/assets/BlogFiles/hello-ms08-067-my-old-friend.pdf)  
[https://blog.rapid7.com/2014/02/03/new-ms08-067/](https://blog.rapid7.com/2014/02/03/new-ms08-067/)  
[https://www.mysonicwall.com/sonicalert/searchresults.aspx?ev=article&id=74](https://www.mysonicwall.com/sonicalert/searchresults.aspx?ev=article&id=74)  
[https://docs.microsoft.com/en-us/security-updates/securitybulletins/2008/ms08-067](https://docs.microsoft.com/en-us/security-updates/securitybulletins/2008/ms08-067)



