---
title: "یه مثال برای درک مفهوم Network-Namespace در Linux"
date: 2019-08-13T10:39:00-04:00
draft: true
---
زمانی که داشتم `Docker` میخوندم درک مفاهیم اولیه -بخاطر شناخت ضعیفم از `NameSpace` ها- برام سخت بود. اما برای درک موضوعات مختلف سعی میکردم توی مفاهیم عمق بگیرم و بدونم پشت هر دستور `Docker` چه اتفاقاتی میافته. اون موقع برای آشنا شدن با `Network-Namespace` یه مثال برای خودم تعریف کردم. مثال این بود: چطور میتونم سه تا کارت شبکه مختلف توی سه تا `NameSpace` مختلف داشته باشم که بتونن همو ببین و بهتره بگم همدیگه را `ping‍‍‍‍‍‍‍` کنن. شکل زیر تصویر مناسبی برای این مثال ارائه می کنه.

<center>![Linux Network Namespaces](/images/linux-nw-namespaces-example.png)</center> 

امروز تصمیم گرفتم این مثال را اینجا بزارم شاید بتونه به تو هم توی درک مفاهیم مربوطه کمک کنه.

نکته: اگه درباره `NameSpace` ها توی لینوکس آشنایی نداری، توی پست قبلی من در اینباره توضیح دادم.

خوب کارمونو با ساختن سه تا `Network-Namespace` شروع می کنیم. دستورات زیر برای ما سه تاشو به نامهای `nsb` ،`nsc` و `nsc` ایجاد میکنه:

{{< highlight sh "linenos=table,linenostart=1" >}}
ip netns add nsa
ip netns add nsb
ip netns add nsc
{{< /highlight >}}

حالا باید سه تا کابل شبکه بسازیم (انتزاعی گفتما) که برای هر کابل، یه سرشو برای اتصال به `Bridge` و سر دیگه را برای اتصال به کارت شبکه ای که توی `Network-Namespace` هایی که با دستورات بالا ساختیم، تنظیم کنیم. دستورات زیر برامون این کار را انجام میدن:

{{< highlight sh "linenos=table,linenostart=1" >}}
ip link add cable-a type veth peer name eth0
ip link set eth0 netns nsa
ip add add 0.0.0.0 dev cable-a
ip link set cable-a up

ip link add cable-b type veth peer name eth0
ip link set eth0 netns nsb
ip add add 0.0.0.0 dev cable-b
ip link set cable-b up

ip link add cable-c type veth peer name eth0
ip link set eth0 netns nsc
ip add add 0.0.0.0 dev cable-c
ip link set cable-c up
{{< /highlight >}}

حواسمون باشه که `cable-b` ، `cable-a` و `cable-c` واقعا کابل شبکه نیستن. در واقع اینها اینترفیس مجازی شبکه هستن و دستور اول برای ما یه دونه ایجاد میکنه. اسمشو میزاره `cable-a` و به `eth0` وصلش میکنه. دستور دوم `eth0` را توی `Network-Namespace` مطلوب (`nsa`) قرار میده. و چون یک طرف این واسط مجازی قراره به `Bridge` وصل بشه، لازم نیست `IP` داشته باشه (دستور شماره ۳) و نهایتا هم واسط مجازی را روشن می کنیم (دستور شماره ۴). درواقع الان یک سر `cable-a` به `eth0`ی که داخل `nsa` قرار داره وصله و سر دیگه اش هم آماده اتصال به `Bridge` شده. دستوراتی که توی سگمنت های بعدی اومده همین کار را برای `nsb` و `nsc` انجام میدن.

قبل اینکه `Bridge` را نصب کنیم دو تا کار دیگه لازم داریم برای `eth0`ها انجام بدیم و اونم تنظیم کردن `IP` و روشن کردنشون هستش. دستورات زیر برامون اینکار را انجام میدن:

{{< highlight sh "linenos=table,linenostart=1" >}}
ip netns exec nsa ip addr add 192.168.1.1/24 dev eth0
ip netns exec nsa ip link set eth0 up

ip netns exec nsb ip addr add 192.168.1.2/24 dev eth0
ip netns exec nsb ip link set eth0 up

ip netns exec nsc ip addr add 192.168.1.3/24 dev eth0
ip netns exec nsc ip link set eth0 up
{{< /highlight >}}

حواسمون باشه دستور `ip netns exec` برامون یک دستور را توی یک `Network-Namespace` اجرا میکنه. مثلا دستور اول `ip addr add 192.168.1.1/24 dev eth0` را توی `nsa` اجرا میکنه. فکر می کنم همین توضیح عملکرد باقی دستورات را شفاف کرده باشه و بیشتر توضیح لازم نباشه.

حالا وقتشه `Bridge` را تنظیم کنیم. اول لازمه یه `Package` نصب کنیم. دستور زیر را روی کنسول بزنید:

{{< highlight sh "linenos=table,linenostart=1" >}}
apt install bridge-utils
{{< /highlight >}}

ماشالله اسم `Package` داره میگه که چیکاره است! دستورات زیر اول برامون یه `Bridge` میسازه و  کابل هایی که بالا ساختیم را بهش وصل می کنه و در انتها `Bridge` را روشن میکنه: 

{{< highlight sh "linenos=table,linenostart=1" >}}
brctl addbr nsbr
brctl addif nsbr cable-a
brctl addif nsbr cable-b
brctl addif nsbr cable-c
ip link set nsbr up
{{< /highlight >}}

حالا میتونیم با دستور `ip netns exec` روی هر کدوم از `Network-Namespace`ها دستور `ping` را اجرا کنیم:
{{< highlight sh "linenos=table,linenostart=1" >}}
ip netns exec nsa ping 192.168.1.2
ip netns exec nsa ping 192.168.1.3

ip netns exec nsb ping 192.168.1.1
ip netns exec nsb ping 192.168.1.3

ip netns exec nsc ping 192.168.1.1
ip netns exec nsc ping 192.168.1.2
{{< /highlight >}}

خوب اگه خواستید چیزهایی که بالا ساختیم را پاک کنید دستورات زیر کمکتون میکنه:


{{< highlight sh "linenos=table,linenostart=1" >}}
ip netns delete nsa
ip netns delete nsb
ip netns delete nsc
ip link delete nsbr
{{< /highlight >}}

پیروز باشید.
