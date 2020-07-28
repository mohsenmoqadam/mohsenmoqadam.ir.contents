---
title: "راست روی کنسول"
date: 2019-07-30T11:18:07-04:00
draft: true
tags: ["Docker", "Linux", "DevOps", "Console", "Programming", "Rust", "Kernel"]
---
آماده سازی محیط توسعه همیشه کار زمانبریه و اگه قرار باشه با یه زبان جدید کد بزنیم که این زمان بیشتر هم میشه. البته شرایط میتونه بدترم بشه اگه گیر بدیم که روی کنسول کد بزنیم! من کد زدن روی کنسولو خیلی دوست دارم و یکی از لذتامه. بنابراین تصمیم گرفتم بازم لذتمو ارضا کنم! توی این پست میخوام راهی که من برای رسیدن به این لذت رفتمو با تو به اشتراک بزارم. اگه تو هم مثل من اعتیاد به خط فرمان داری این پست میتونه زمان رسیدن به drug را برات کمینه کنه.

من از linux-mint استفاده میکنم و احتمالا روشی که با هم دنبال میکنیم روی ubuntu و debian هم جواب بده. کاری که انجام میدیم چهارتا بخش داره:

- اول: پکیج هایی که برای توسعه روی Linux لازم داریمو نصب میکنیم.

- دوم: Rust و ملزوماتشو نصب میکنیم.

- سوم: Emacs دوستاشتنیو نصب و کانفیگ میکنیم.

- چهارم: در آخر هم یه hellow_world ساده می نویسم تا باهاش مطمئن بشیم همه چی درسته.

خوب حالا که میدونیم نیت چیه دست بکار میشیم. دستور زیر برامون مرحله اول را انجام میده:

{{< highlight sh >}}
apt install build-essential automake clang libclang-dev
{{< /highlight >}}


با استفاده از دستور زیر Rust را نصب میکنیم. باید بدونیم Rust توی سه تا کانال توزیع میشه و ما از توزیع Nightly استفاده می کنیم. دستوراتی که در ادامه اومده برامون Rust با توزیع Nightly را نصب می کنه. علاوه بر این ملزومات Code Completion که توی Emacs لازم داریم با استفاده از دو تا دستور آخر نصب میشن. اگه حین نصب سوالی شد گزینه پیشفرض جوابه!:

{{< highlight sh >}}
curl https://sh.rustup.rs -sSf | sh
rustup default nightly
rustup update
rustup component add rust-src
cargo install racer
{{< /highlight >}}
  

فعلا کارمون با Rust تموم شده بریم سراغ Emacs. من Emacs را کامپایل و بعد نصبش میکنم. برای همین به سورس کدش نیاز دارم. مشابه دستورات زیر اول سورس کد را دانلود میکنم و توی یه جای موقت بازش میکنم و اونجا دستورات کامپایل و نصب را اجرا میکنم:

{{< highlight sh >}}
cd /tmp
wget https://ftp.gnu.org/gnu/emacs/emacs-26.1.tar.gz
tar -zxvf emacs-26.1.tar.gz
cd emacs-26.1
./configure --without-x --with-gnutls=no
make
sudo make install
{{< /highlight >}}

خوب اگه تو اجرای دستورات بالا به خطایی نخوردید، Emacs بدرستی نصب شده و باید کانفیگش کنیم تا برای کد نوسیی با Rust آماده شه. اول بهتره چنتا کلید ترکیبی یا key-bind که خیلی توی Emacs بکارمون میاد را یادآوری کنیم. توی نگارش وقتی مینویسیم M-x یعنی ترکیب کلید Alt با یه کلید دیگه مثلا x. علاوه بر این وقتی مینویسیم C-x یعنی ترکیب کلید Ctrl با یه کلید دیگه مثلا x. یادمون بمونه که RET هم همون Enter خودمونه. حالا با اجرای دستور زیر فایل کانفیگ Emacs را باز کنید:


{{< highlight sh >}}
emacs ~/.emacs
{{< /highlight >}}


و کد های زیر را توش کپی کنید:

{{< highlight lisp >}}
(require 'package)
(let* ((no-ssl (and (memq system-type '(windows-nt ms-dos))
		    (not (gnutls-available-p))))
       (proto (if no-ssl "http" "https")))
  (add-to-list 'package-archives (cons "melpa" (concat proto "://melpa.org/packages/")) t)
  (when (< emacs-major-version 24)
    (add-to-list 'package-archives '("gnu" . (concat proto "://elpa.gnu.org/packages/")))))
(package-initialize)
(add-hook 'rust-mode-hook )
(add-hook 'racer-mode-hook )
(add-hook 'rust-mode-hook 'cargo-minor-mode)
{{< /highlight >}}


برای اینکه تغییرات ذخیره بشن اول C-x و بعدش C-s را بزنید.

  
الان لازمه چنتا بسته یا Package برای Emacs نصب کنیم. برای اینکه Emacs بتونه به مخزن بسته هاش وصل شه باید یه تغییر کوچیک توی متغییرهاش بدیم. برای این کار بعد زدن M-x کلید واژه customize-variable را مینویسیم و Enter می کنیم. با زدن Enter پرامپت منتظر وارد کردن کلید واژه بعدی میمونه که باید package-archives را بنویسیم و Enter بزنیم. حالا توی ویرایشگر نتیجه را میبینیم. کافیه URLی که مربوط به melpa و از نوع https هست را به http تغییر بدیم و تغییرات را ذخیره کنیم. برای ذخیره شدن هم C-x و بعدش C-s را می زنیم. اگه بخوام همه ی این توضیحاتو با کد بگم اینجوری میشه:

{{< highlight sh >}}
M-x customize-variable RET
package-archives
-> changed https to http in melpa url <-
C-x C-S
{{< /highlight >}}

  
حالا که Emacs میتونه به مخزن بسته هاش وصل شه ما لیست بسته هاشو بروز می کنیم و بسته هایی که دنبالش بودیم را جستجو و نصب می کنیم. من دیگه توضیح فارسی نمی نویسم. فقط یادمون باشه بسته های rust-mode، racer، company، و cargo را باید نصب کنیم. راستی برای جستجو توی لیست بسته ها از C-s و برای جابجایی بین پنجره ها از C-x o استفاده می کنیم:

{{< highlight erlang >}}
M-x package-refresh-contents RET
M-x list-packages RET
C-s rust-mode RET
C-o then enter on install
C-o
C-s racer RET
C-o then enter on install
C-o
C-s company RET
C-o then enter on install
C-o
C-s cargo RET
C-o then enter on install
{{< /highlight >}}


حالا کانفیگ مربوط به بسته های نصب شده را انجام میدیم. فایل کانفیگ Emacs را باز می کنیم:

{{< highlight sh >}}
C-x C-f ~/.emacs
{{< /highlight >}}

و دستورات زیر را توش کپی می کنیم. یادمون باشه با C-x و بعدش C-s تغییرات را ذخیره می کنیم و از C-x و بعدش C-c برای خروج از Emacs استفاده کنیم.

{{< highlight lisp >}}
(add-hook 'rust-mode-hook )
(add-hook 'racer-mode-hook )
(add-hook 'rust-mode-hook 'cargo-minor-mode)
{{< /highlight >}}
  

خوب همه چی برای مصرف Drug آماده است. فقط کافیه اولین پروژه Rust را با استفاده از cargo ایجاد کنیم. اسمشم میزاریم همون همیشگی hello-world. بعدشم فایل main.rs را با Emacs باز میکنیم تا جذب لذت شروع شه!:

{{< highlight sh >}}
cd /tmp
cargo new hello_world
cd hello_world
emacs src/main.rs
{{< /highlight >}}

کد زیر فایل main.rs را نشون میده:

{{< highlight rust >}}
fn main() {
   println!("Hello, world!");
}
{{< /highlight >}}
  

حالا اگه میخوایید پروژه را build بگیرید از C-c C-c C-b و اگه میخوایید اجراش کنید C-c C-c C-r بکارتون میاد:

{{< highlight sh >}}
C-c C-c C-b: Build Project
C-c C-c C-r: Run Project
{{< /highlight >}}


و اما اگه خواستید برنامه هایی که نصب کردیم را حذف کیند دستورات زیر کمکتون میکنه:

{{< highlight sh >}}
rustup self uninstall
cd /tmp/emacs-26.1/
sudo make uninstall
{{< /highlight >}}

پیروز باشید.