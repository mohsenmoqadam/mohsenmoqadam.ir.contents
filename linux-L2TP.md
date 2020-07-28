---
title: "پیکربندی L2TP در Linux"
date: 2019-08-08T12:19:52-04:00
draft: true
tags: ["Linux", "DEBIAN", "DevOps", "Console", "MINT", "UBUNTU", "VPN", "L2TP", "Network", "Kernel"]
---

یکی از مشکلاتی که معمولا توی لینوکس توزیع های mint و ubuntu بهش ممکنه برخورد کنیم راه اندازی کلاینت L2TP هستش. من خیلی از کاربرای معمولی این توزیع ها را دیدم که با این مشکل درگیر میشن و نهایتا نمی تونن مشکلشونو حل کنند. واقعیت امر اینه که L2TP-Client روی این توزیع ها بد قلق هست. بهمین خاطر توی این پست تصمیم گرفتم درباره این موضوع صحبت کنم.

  

دقت داشته باشید ممکنه این مراحل روی نسخه توزیع شما کار نکنه اما میتونه راه حل کلی را نشون بده. من مراحل زیر را روی توزیع های Linux Mint 19.1 و Linux Ubuntu 18.04 تست گرفتم.

  

۱- نصب ابزارهای مورد نیاز

{{< highlight sh >}}
sudo apt install network-manager-l2tp network-manager-l2tp-gnome
sudo apt install libreswan
sudo apt install resolvconf
{{< /highlight >}}

۲- اطمینان از ورژن IPSec

  

دستور زیر را روی کنسول لینوکس خود وارد کنید:

{{< highlight sh >}}
ipsec --version
{{< /highlight >}}

خروجی دستور بالا روی سیستم من:

{{< highlight sh >}}
Linux strongSwan U5.6.2/K4.15.0-20-generic
Institute for Internet Technologies and Applications
University of Applied Sciences Rapperswil, Switzerland
See 'ipsec --copyright' for copyright information.
{{< /highlight >}}

  
  

اگه ورژن IPSec شما libreswan هست، دستور زیر را اجرا کنید و مطمئن بشید که از ورژن strongSwan استفاده می کنید:

{{< highlight sh >}}
apt install strongswan
{{< /highlight >}}

  

۳- اطمینان از ورژن xl2tp

  

ابتدا دستور زیر را روی کنسول لینوکس خود اجرا کنید:

{{< highlight sh >}}
sudo xl2tpd -v
{{< /highlight >}}

خروجی دستور بالا روی سیستم من:

{{< highlight sh >}}
xl2tpd version: xl2tpd-1.3.14
{{< /highlight >}}

اینجا باید به یه نکته خیلی مهم دقت کنیم. اگه ورژن کرنل لینوکس شما ۴.۱۵+ هست باید ورژن xl2tp شما ۱.۳.۱۲+ باشه. اگه ورژن پایینتری داشتید میتونید با استفاده از سورس نصبش کنید:

{{< highlight sh >}}
git clone https://github.com/xelerance/xl2tpd.git
cd xl2tpd
make
sudo make install
{{< /highlight >}}

اگه بعد نصب متوجه شدید نسخه xl2tpd تغییر نکرد، کنسول جدیدی باز کنید و مجددا ورژن را چک کنید. اگه باز هم ورژن تغییر نکرده بود بهتره با استفاده از دستور:

{{< highlight sh >}}
which xl2tpd
{{< /highlight >}}

  

مسیرهای xl2tpd را پیدا و حذف کنید. بعد از حذف مجددا xl2tpd را نصب کنید.

  

۴- حالا میتونید توی Network Settings یک اینترفیس از نوع L2TP ایجاد کنید. دقت کنید اگه نتونستد گذرواژه یا Password اکانت خود را وارد کنید، نگران نباشید و بقیه پارامتر ها را تنظیم کنید. وقتی اینترفیس L2TP را روشن می کنید از شما Password اکانت سوال میشه. اگه این موضوع ناراحتتون می کرد باید یکم بیشتر زحمت بکشید و توی مسیر /etc/NetworkManager/system-connections دنبال فایلی به نام اینترفیسی که ساختید بگردید و دو تا تنظیم انجام بدید. اولیش اینه که توی تنظیمات سگمنت [vpn] دنبال کلیدواژه password-flags بگردید و مقدارش را به صفر (0) تنظیم کنید. دومیش هم اینه که یه سگمنت به نام [vpn-secrets] ایجاد کنید و گذرواژه اکانتتونو توش وارد کنید. مشابه زیر:

{{< highlight sh >}}
[vpn-secrets]
password=1234
{{< /highlight >}}



حالا تغییرات را ذخیره کنید و دستور زیر را اجرا کنید تا تغییرات جدید اعمال بشن:

{{< highlight sh >}}
sudo service network-manager restart
{{< /highlight >}}

  

۴- قطع و وصل کردن اکانت روی کنسول

  

اگه دوست داشتید اینترفیسی که ساختید را روی کنسول وصل کنید، اول لیست همه اینترفیس ها را مشاهده کنید تا اسم اینترفیس ساخته شده را کامل ببینید. بعد میتونید با استفاده از اسم روشنش کنید. بعوان مثال روی سیستم من:

{{< highlight sh >}}
sudo nmcli connection
{{< /highlight >}}

  

خروجی دستور بالا:

{{< highlight sh >}}
NAME UUID TYPE DEVICE
Wired connection 1 6c784811-353b-380b-abca-f934caf06a73 ethernet enp2s0
BP1 1947ddcf-999d-485d-ae6a-9dec025ef578 vpn --
BP2 670e3e71-025f-484a-982d-4331510a09c9 vpn --
{{< /highlight >}}

  

حالا با استفاده از دستور زیر اینترفیس BP2 را میتونم وصل کنم:

{{< highlight sh >}}
sudo nmcli connection up BP2
{{< /highlight >}}

  

برای قطع کردن اینترفیس هم میتونید از دستور زیر استفاده کنید:

{{< highlight sh >}}
sudo nmcli connection down BP2
{{< /highlight >}}

  

پیروز باشید.