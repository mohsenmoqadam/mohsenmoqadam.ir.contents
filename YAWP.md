---
title: "باز هم یک Worker Pool دیگه!"
date: 2019-08-25T11:27:52-04:00
draft: true
tags: ["Erlang", "Worker Pool", "YAWP", "MohsenMoqadam", "BisPhone.com", "sapp.ir", "Messenger", "Distributed Systems", "Closed Loop System", "Stochastic Process", "Telegram.me"]
---
توی این پست می خوام با `YAWP` و ایده های پشتش آشناتون کنم. `YAWP` از اول واژه های `Yet Another Worker Pool` گرفته شده و مبتنی بر تئوری احتمالات، تئوری صف و مباحث کنترل سیستم های حلقه بسته طراحی شده. البته من اصلا نمیخوام درگیر مباحث تئوری ماجرا بشم و تلاش میکنم اصول طراحی `YAWP` را خیلی ساده و به زبان برنامه نویسی بگم. طرحی که درباره اش توی پست قرار صحبت کنیم را به زبان `Erlang` نوشتم و توی محصولات مختلفی ازش استفاده کردم و تلاشم اینه که قابلیت های بیشتری بهش اضافه کنم. اگه شما هم تمایل داشتید میتونید ایده های پشت `YAWP` را با زبان برنامه نویسی مطلوبتون پیاده سازی کنید و بهم اطلاع بدید، تا ازش لذت ببریم!


یکی از ملزومات پروژه های مبتنی بر `Actor Oriented Programming` و `Erlang` که همیشه درگیرش میشیم `Worker Pool` هستش. واقعیت امر اینه که تا حالا مدل های مختلفی برای این منظور توسعه داده شده، اما هیچ کدوم واقعا مبتنی بر مدل علمی پیشرفته و قوی نیست. بنظر من یا توسعه دهندگانشون تسلطی به مباحث علمی ماجرا نداشتن یا صرفا یک پروژه دم دستی و بهتره بگم دانشجویی آماده کردن و یا اینکه توی بحران زمان راه حل ساده ای برای رسیدن به هدف انتخاب کردن و توی `github‍‍` به اشتراک گذاشتن. ما به همه ی راهکارهای موجود احترام میزارم و معتقدیم همین اشتراک باعث شده نواقص موضوع در بیاد و ما بتونیم راه حل بهتری برای ماجرا داشته باشیم. شاید راه حل ارائه شده توی این پست و `YAWP` هم در آینده اصلاح بشه یا ایده باشه برای توسعه دهنده ی دیگری تا بتونه روش بهینه تری ارائه کنه.

یک نکته باید اینجا مطرح کنم، و اون اینه که خیلی ها معتقدن `Erlang` خیلی کنده و یا زبان قدیمی و مواردی از این دست! بنظر من `Erlang` زبان متفاوتیه (مثل همه زبان ها!) دلیلشم بخاطر هدفیه که تولید شده. اتفاقا بین همه زبان هایی که باهشون تجربه دارم `Erlang` به شدت راحت تره! `Design Pattern` ها و `Concept` ها توش محدودن و همین باعث میشه روش های که برای حل مسائل باهاش ارائه میشه فاصله زیادی توی `Implementation` از هم نداشته باشن. چیزی که باید دقت داشته باشیم اینه که با همه این خوبی ها اگه به مباحث علمی سیستم های توزیع شده و معماری `EVM` و سایر مباحث علمی مورد نیاز یک هدف تسلط نداشته کافی باشیم، به شدت راه حل ما برای یک مسئله کیفیت پایینی خواهد داشت و متعاقبا کارایی برنامه نوشته شده افت پیدا می کنه و ما انگشو به `Erlang` میچسپونیم! 

خوب بریم سراغ موضوع پست! من با رویکرد `Top-Down` (از بالا به پایین) موضوع را تشریح میکنم و توی مسیر هرجا نیاز باشه توصیف بیشتری از ماجرا خواهیم داشت. خوب اگه بخوام خیلی ساده بگم ما دنبال `Worker Pool`ی هستیم که چنتا ویژگی داشته باشه:


-   محدودیتی در ایجاد تعداد `Worker Pool` نداشته باشیم.
    
-   کارایی بالایی داشته باشه.
    
-   اینترفیس ساده ای برای `Assign` کردن `Task` به `Worker` ها ارائه بده.
    
-   بتونیم تعداد `Worker` ها را توش تغییر بدیم.
    
-   بتونیم یه سری گزارشات هم از وضعیت `Pool` داشته باشیم.
    
-   ترجیحا `Library` باشه و مفاهیمی که داره توی لایه های دیگه برنامه نفوذ نکنه.
    
-   بتونیم کاراییشو تست و ارزیابی کنیم.
    
-   بتونیم `Worker` ها را مانیتور کنیم و اگه مردن دوباره زنده شون کنیم!

شکل زیر معماری `YAWP` برای رسیدن به این ویژگی ها را نمایش میده.

<center><img src="/images/yawp-arch.png" alt="YAWP-arch" /></center>

خوب حالا ببینیم مولفه هایی که توی شکل بالا نمایش داده شدن چی هستن و قراره چه کار انجام بدن:

`yawp`: اینترفیسی که لازم داریم تا `Worker Pool` را ایجاد و مدیریت کنیم و `Task`ها را به `Worker` ها تحویل بدیم در اختیارمون قرار میده. اگه بخواهیم `Erlang`ی بررسی کنیم، توابعی ماژول `yawp` بهمون میده که عبارتند از:

-   تابع `new`: این تابع برامون یک `Worker Pool` ایجاد میکنه. به همین راحتی و فقط یک ورودی میگیره که اونم تنظیمات مطلوب `Pool` مورد نظر هستش، بعنوان مثال جند تا `Worker` قراره توی `Pool` قرار داشته باشن و …
    
-   تابع `do_task`: یک `Task` را به شایسته ترین `Worker` تحویل میده. درباره معیارهای شایستگی یکم دیگه صحبت می کنیم.
    
-   تابع `do_sync_task`: این تابع هم یک `Task‍` را به شایسته ترین `Worker` تحویل میده و تفاوتش با بالایی اینه که پراسسی که این تابع را فراخوانی می کنه برای دریافت پاسخ منتظر میشه.
    
-   تابع `increment_pool_size`: تعداد `Worker` های یک `Pool` را افزایش میده.
    
-   تابع `decrement_pool_size`: تعداد `Worker` های یک `Pool` را کاهش میده.
    
-   تابع `get_pool_size`: تعداد `Worker` های موجود در `Pool` را اعلام میکنه.
    
-   تابع `get_pool_stats`: گزارشی `Task` های انجام شده ارائه میده.
    

  

`SUP_pool`: این بخش از `YAWP` وظیفه نظارت بر `Worker` های `Pool` را بعهده داره و اگه `Worker` دچار مرگ شد با استراتژی مشخصی دوباره زنده اش می کنه.

  

`CTRL`: این بخش وظیفه کنترل `Pool` را بعهده داره. بعنوان مثال تغییرات اندازه `Pool` توسط این مولفه مدیریت میشه.

  

`MNTR`: این مولفه `YAWP` آمارهای `Worker`ها را جمع آوری و برای ارائه گزارش آماده میکنه.

  

`BCN`: این مولفه وظیفه تولید `Beacon` با نرخ ثابت را به عهده داره. اینکه `Beacon` چیه و چه کاربردی داره را جلوتر متوجه میشیم.

  

`W_i`: این مولفه هم `Worker`های `Pool` را ارائه می کنه و در واقع کمک میکنه محاسبات مورد نیاز `YAWP` توی برنامه ای که ازش استفاده می کنه نفو‌ذ نکنه. اگه بخوام `Erlang`ی بگم `YAWP` برای این منظور `Behaviour`ی بنام `gen_pool_worker` در اختیار ما قرار میده و ما میتونیم با استفاده از این قابلیت `Business` هر `Worker` را مستقل از `YAWP` و قواعدش پیاده سازی کنیم.

تا اینجا به درک اجمالی از معماری `YAWP` رسیدیم و حالا وقتشه با ایده های اصلی `YAWP` آشنا بشیم. بهترین و پرکاربرد ترین مدلی که برای یک `Process` یا `Actor` میشه ازش استفاده کرد مدل `Queue` هست. این مدل با `Actor`های Erlang بیشترین همونایی و دقت را داره اما وقتی با `Proccess` یا `Thread` سروکار داریم یکمقدار دقتش پایین میاد. هر `Actor` در ساده ترین شکلش از یک `Queue` و یک واحد پردازش که میتونیم `Process` یا `Thread` بهش بگیم ساخته میشه. ما توی `Erlang` به این دو بهمراه یه سری موارد دیگه `Actor` می گیم اما توی ادبیات تئوری صف به واحد پردازشی معمولا `Server` اتلاق میشه. شکل زیر تصوری از مدل `Actor` بهمون نشون میده.

<center><img src="/images/yawp-queue.png" alt="yawp-queue"/></center> 

`Task`ها اجزای یک `Load` هست که برای انجام شدن توی صف `Actor` قرار میگیره و `Actor` بترتیب `Task`ها را بر میداره و پردازششون میکنه. جالب اینه که `Actor` هایی از جنس `gen_server` توی `Erlang` دقبقا همبن رفتار را دارند. حالا فرض کنیم بخواهیم با این مدل `Worker Pool` بسازیم. یک روش میتونه این باشه که همه `Task`ها تحویل یک `Actor` بشه و اون مثلا بصورت `Round Robin` به `Worker`ها تحویل بده. پیاده سازی این ایده روی `github` موجود هست. ضعفی که این پیاده سازی داره اینه که سرعت برداشتن و `Assign` کردن `Task`ها محدوده و اگه اندازه `Load` زیاد باشه تعداد `Task`های زیادی توی `Queue` قرار میگیرن و منتظر `Assign` شدن می مونن! نتیجه میشه تاخیر انجام `Task`ها. البته اگه فرض کنیم زمانی که برای پردازش هر `Task` توی `Worker`ها لازمه نسبت به زمان `Assign` کردن خیلی بیشتر باشه این ایده تا حدودی میتونه خوب کار کنه. 
میشه ایده های دیگه ای هم برای رفع چالش توی `github` یافت. بعنوان مثال: پروژه `Toveri` تلاش کرده راه حل متفاوتی ارائه کنه. ما هم توی محصولات مختلفی تا حالا ازش استفاده کردیم. `Toveri` برای ارسال `Task`ها از انتخاب `Round Robin` استفاده میکنه. اطلاعاتی که لازم داره را توی `ETS` نگهداری میکنه و هربار که میخواد `Task`ی را به `Worker`ی تحویل بده نیاز داره یک `Read` و یک `Write` روی `ETS` انجام بده. برای اینکار وقتی اندازه `Load` زیاد میشه `Toveri` هزینه سنگینی باید پرداخت کنه. ضمن اینکه تحویل کار به یک `Worker` بدون اطلاع از شرایط `Worker` انجام میشه و بهتره `Worker`ی که تعداد `Task`های کمتری توی صفش داره انتخاب شه. 

اگه بیشتر بگردیم متوجه ایرادات روش های موجود می شیم. عمده این ایرادات عبارتند از:

-   درنظر نگرفتن مدل `Queue`.
    
-   استفاده نادرست از ابزارهایی مثل `ETS` و … .
    
-   عدم توزیع بار محاسباتی مربوط به انتخاب `Worker` یا `Worker` شایسته.
    
-   تحویل کورکورانه `Task`ها به `Worker`ها بدون در نظر گرفتن بار جاری `Worker`.

حالا ببینیم `YAWP` چه راه حل هایی را برای مشکلات موجود استفاده میکنه. یکی از مهمترین نیازمندی ها `YAWP` اینه که از وضعیت `Worker` ها اطلاع جمع آوری کنه تا بتونه یک `Task` را به شایسته ترینشون تحویل بده و اینجوری زمان پردازش `Load` را کاهش بده. برای رسیدن به این هدف `YAWP` از ایده `Beacon` استفاده می کنه. `Beacon` درواقع یک `Task` سبک وزن هستش که وقتی تحویل `Worker` داده میشه فقط کافیه `Worker` تعداد `Task`های موجود در صفشو توش بروز کنه و بصورت `cast` (یعنی منتظر تصدیق دریافت نمی مونه) برای مولفه `CTRL` ارسال کنه. `Beacon` ها با نرخ ثابت مثلا هر ۲۰۰ میلی ثانیه ارسال میشن. شکل زیر کمک میکنه به درک بهتری برسیم:

<center><img src="/images/yawp-beacon.png" alt="yawp-beacon"/></center> 

زمانهایی که اندازه `Load` کمه `Task`ها با سرعت کمتری وارد `Queue` میشن. بنابراین بین دو `Beacon` متوالی `Task`های کمتری قرار میگیره اما وقتی اندازه `Load` افزایش پیدا می کنه بین دو `Beacon` متوالی تعداد بیشتری `Task` در صف قرار می گیره. بنابراین وقتی تعداد `Task`های یک `Worker` افزایش پیدا میکنه سرعت پردازش `Beacon`ها کاهش پیدا میکنه. علاوه بر این اگه بعضی `Task`های تحویل شده به یک `Worker` زمان پردازش بیشتری مطالبه کنن باز هم روی زمان پردازش `Beacon`ها اثر خواهد گذاشت. با این تفاسیر `Beacon`ها معیاری برای اندازه گیری طول صف و شلوغ بودن `Worker` در اختیار `YAWP` قرار بدن. اگه حواسمون جمع باشه `Beacon`ها با نرخ ثابت مثلا ۲۰۰ میلی ثانیه، تولید میشن اما تضمینی وجود نداره توسط `Worker`ها با نرخ ثابت پردازش بشن. مثلا فرض کنیم فاصله دریافت `Beacon`های یک `Worker` توسط `CTRL` حدود 400 میلی ثانیه طول بکشه. این یعنی اینکه بین دو `Beacon` متوالی `Worker` زمانی بیشتر از ۲۰۰ میلی ثانیه مشغول پردازش `Task`ها بوده. هر چه این زمان از بازه زمانی بین دو `Beacon` متوالی بیشتر باشه یعنی `Worker` بیشتر شلوغه و باید بهش کمتر `Task` تحویل بشه تا از شلوغ بودنش کاسته بشه.

از طرفی باید حواسمون هم باشه که سیستم بلاک نشه و نره توی حالتی که همه `Worker`ها شلوغن و دیگه نمیتونیم `Task` تحویل بدیم. برای رفع این مشکل `YAWP` برای هر `Worker` یک شناس پذیرش `Task` محاسبه میکنه و توی زمانهای ثابتی این شانس را توی `ETS` به اشتراک میزاره، بعنوان مثال هر 200 میلی ثانیه! بنابراین توی هر ثانیه تنها پنج `Write` روی `ETS` اتفاق میفته. 
از طرفی هر `Actor`ی که تابع `do_task` یا `do_sync_task` را صدا بزنه محتوای `ETS` را میتونه `Read` کنه. با روشن کردن `read_concurrency` روی `ETS` میتونیم موازی خواندن از `ETS` را روشن کنیم و  کمک کنیم خواندن از `ETS` با بیشترین سرعت ممکن انجام بشه. 

 برگردیم به موضع شانس `Worker`ها برای انتخاب شدن. گفتیم `YAWP` برای هر `Worker` با استفاده از `Beacon` یک شانس محاسبه می کنه. اگه فاصله بین دو `Beacon` متوالی که توسط یک `Worker` ارسال میشه بیشتر از فاصله زمانی ارسال دو `Beacon` متوالی باشه یعنی اون `Worker` شلوغ شده. علاوه بر این `Worker`ها توی `Beacon` طول صفشونم اطلاع میدن. بنابراین مولفه `CTRL` هم میتونه از طول صف و هم از میزان زمانی شلوغ بودن هر `Worker` اطلاع کسب کنه. `CTRL` از این اطلاع برای ساختن `CPD` استفاده میکنه و توی دوره های زمانی مناسب این `CPD` را توی `ETS` بروز میکنه. نکته ای که باید بهش دقت کنیم اینه که شانس هر `Worker` در قبال وضعیت سایر `Worker`ها سنجیده میشه و یک شانس نسبی محاسبه میشه!

وقتی یک `Actor` بخواد `Task`ی را به `Pool` تحویل بده ابتدا `CPD` را از `ETS` میخونه و یک الگوریتم بهینه را برای انتخاب بهترین `Worker` اجرا می کنه. اینجا باید دقت داشته باشیم که این هزینه را `YAWP` پرداخت نمیکنه بلکه `Actor`ی که یکی از توابع `do_task` یا `do_sync_task` را فراخوانی کرده پرداخت میکنه!

شکل زیر کمک میکنه بحث بالا را بهتر درک کنیم. درواقع این شکل تصویری از یک سیستم کنترلی جلقه بسته را بما نشون میده و میتونیم ثابت کنیم که این سیستم در حالت `steady state` میتونه به نقطه پایداری همگرا بشه. من دیگه وارد این بحثاش نمیشم و اگه فرصت کنم در آینده فرمولاسون `YAWP` و مدل ریاضی شو جداگانه تشریح می کنم. حالا وقتش یکم با `YAWP` کار کنیم.

<center><img src="/images/yawp-closed-loop.png" alt="yawp-closed-loop" width="80%"/></center> 

برای استفاده از `YAWP` لازمه `Erlang 18+` ،`Rebar3` و `git` روی سیستم شما نصب باشه. دستورات زیر `YAWP` را دریافت و اجراش میکنه:

{{< highlight sh "linenos=table,linenostart=1" >}}
cd /tmp
git clone https://github.com/mohsenmoqadam/YAWP.git
cd YAWP
make rel-dev && make console-dev
{{< /highlight >}}

به همین سادگی! 
حالا دستورات زیر را روی `REPL` اجرا کنید تا یک `Worker Pool` با حداقل 16 عدد `Worker` ایجاد بشه.


{{< highlight erl "linenos=table,linenostart=1" >}}
rr(yawp).
Pool = #yawp_pool{name=my_pool, max_size=32, min_size=16, sui=200, bi=200, worker={yawp_gen_worker_test, start_link, [{'Ki', 'Vi'}]}}.
yawp:new(Pool).
observer:start().
{{< /highlight >}}

آخرین دستور `observer` را برای شما باز می کنه و میتونید بصورت تصویری ببینید چه اتفاقی افتاده. شکل زیر عکسی از `observer` روی سیستم من هستش. اگه دقت کنید همه مولفه های `YAWP` ساخته شده. 

<center><img src="/images/yawp-observer.png" alt="yawp-observer" width="100%" height="100%"/></center> 

برای ایجاد یک `Pool` تنظیمات با استفاده از رکورد `{}yawp_pool#` اعلام میشه. این تنظیمات شامل ۶ مورد هست:

-   `name`: نام `Worker Pool`. با استفاده از این نام میتونیم `Task`ها را به `Pool` تحویل بدیم، گزارش بگیریم و یا تنظیمانشو بعدا تغییر بدیم.
    
-   `max_size`: تعداد حداکثر `Worker` ی که میتونه توی `Pool` وجود داشته باشه را مشخص می کنه. حواسمون باشه این عدد بعد از ساخته شدن `Pool` قابل تغییر نیست.
    
-   `min_size`: تعداد حداقل `Worker` ی که میتونه توی `Pool` وجود داشته باشه را مشخص می کنه. و باز هم حواسمون باشه این عدد بعد از ساخته شدن `Pool` قابل تغییر نیست.
    
-   `sui`: معادل حروف اول `Shaper Update Interval` هستش و بازه های زمانی بر حسب میلی ثانیه برای بروز کردن `CPD` را داخل `ETS` مشخص می کنه.
    
-   `bi`: معادل حروف اول `Beacon Interval` هستش و بازه های زمانی ارسال `Beacon`ها را بر حسب میلی ثانیه مشخص می کنه.
    
-   `Worker`: در واقع یک `(`MFA (`Module, Function, Arity` هستش که به `YAWP` اطلاع میده از چه `Module`ی برای ساختن `Worker` ها استفاده کنه. یک `Module` نمونه به نام `yawp_gen_worker_test` همراه پروژه هستش که ما برای ساختن `Pool` ازش استفاده کردیم.

برای ارسال` Task`ها هم میتونیم مشابه دستورات زیر را روی `REPL` وارد کنیم:


{{< highlight sh "linenos=table,linenostart=1" >}}
yawp:do_task(my_pool, any_task).
yawp:do_sync_task(my_pool, any_task).
{{< /highlight >}}

حواسمون باشه که `any_task` یک `atom` هستش و بجاش میتونه هر ترم `Erlang`ی استفاده بشه. مهم اینه که این ترم برای `Worker` قابل درک باشه. اما برای معرفی یک `Module` بعنوان `Worker` باید از `Behaviour` مخصوصی که `YAWP` ارائه می کنه استفاده کنیم. روش استفاده از این `Behaviour` توی ماژول `yawp_gen_worker_test` که همراه پروژه قرارداره مشخص شده. سورس کد این ماژول در قطعه کد زیر نمایش داده شده:

{{< highlight java "linenos=table,hl_lines=3,linenostart=1" >}}
-module(yawp_gen_worker_test).

-behaviour(yawp_gen_worker).

%% API
-export([start_link/1]).

%% gen_server callbacks
-export([init/2,
         handle_call/3,
         handle_cast/2,
         handle_info/2,
         terminate/2,
         code_change/3,
         format_status/2]).

-include("yawp.hrl").
-define(SERVER, ?MODULE).

-record(state, {pool_name :: string()}).

%%%===================================================================
%%% API
%%%===================================================================

start_link(Confs) ->
    yawp_gen_worker:start_link(?MODULE, Confs, []).

%%%===================================================================
%%% yawp_gen_worker callbacks
%%%===================================================================

init(PoolName, _Args) ->
    process_flag(trap_exit, true),
    {ok, #state{pool_name = PoolName}}.

handle_call(_Task, _From, State) ->
    Reply = ok,
    {reply, Reply, State}.

handle_cast(_Task, State) ->
    %%timer:sleep(20),
    {noreply, State}.

handle_info(_Info, State) ->
    {noreply, State}.

terminate(_Reason, #state{pool_name = PoolName} = _State) ->
    ?YAWP_LOG_INFO("Worker ~p leaves pool: ~p", [self(), PoolName]),
    ok.

code_change(_OldVsn, State, _Extra) ->
    {ok, State}.

format_status(_Opt, Status) ->
    Status.

%%%===================================================================
%%% Internal functions
%%%===================================================================
{{< /highlight >}}
اگه دقت کنیم متوجه خواهیم شد که دقیقا یک `gen_server` استاندارد `OTP` هستش و همه ی قواعد `gen_server` اینجا هم صادقه.

اگه توی استفاده از `YAWP` مشکلی مشاهده کردید خوشحال میشم با من درمیون بزارید.

پیروز باشید.


