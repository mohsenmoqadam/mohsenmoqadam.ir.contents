---
title: "ارتباط بین دو process با استفاده از pipe در Linux"
date: 2019-08-11T09:19:01-04:00
draft: true
tags: ["Mint", "Fork", "Linux", "Child", "DevOps", "Console", "Parent", "Process", "Pipe", "FileSystem", "Kernel"]
---
وقتی بخواهیم خروجی یک دستور را به ورودی دستور دیگه ای وصل کنیم از Pipe استفاده می کنیم. Pipe را می تونیم روی کنسول همه سیستم عامل های مبتنی بر POSIX استفاده کنیم. علامت Pipe روی کنسول | هستش و خروجی دستوری که سمت چپش نوشته میشه را به ورودی دستور سمت راستش متصل می کنه. واقعیت امر اینه که نمی خوام زیاد درباره Pipe روی کنسول صحبت کنم، بلکه نیتم اینه که از این قابلیت برای اتصال دو پراسس استفاده کنیم و یمقدار درباره چگونگی انجام این هدف عمق بگیریم. حواسمون باشه که نمونه کد ها صرفا برای روشن شدن موضوع ارائه شده و توصیه نمیشه مستقیم توی محصول نهایی استفاده بشن. علاوه بر این برای ساده شدن بحث مجبورم یکم انتزاعی تر صحبت کنم و بنویسم. 


خوب اول ببینیم دقیقا Pipe روی کنسول چطوری کار میکنه. برای این کار اول توی یه فایل یه استرینگ ساده می نویسیم و بعد محتوای فایل را با استفاده از cat  می گیریم و به wc می فرستیم. دستور cat  معمولا برای نمایش محتوای فایل هم استفاده میشه و محتوی فایلی که بهش میدیم را روی کنسول نمایش میده. ما این خروجی cat  را با استفاده از Pipe به دستور wc میفرستیم. دستور wc هم تعداد خطوط، واژه ها و حروف فایلی که بعنوان ورودی گرفته را نمایش میده. همه ی این داستان توی دستورات زیر نمایش داده شده:

{{< highlight sh >}}
echo "A sample string!" > /tmp/s.txt
cat /tmp/s.txt
cat /tmp/s.txt | wc
{{< /highlight >}}

خروجی دستور آخر هم باید مشابه زیر باشه:

{{< highlight sh >}}
1 3 17
{{< /highlight >}}

و درواقع داره بترتیب از چپ به راست: تعداد حروف، تعداد واژه ها و تعداد خطوط را نمایش میده.


خوب همینقدر راجع به Pipe روی کنسول کفایت میکنه. حالا میخوام به این فکر کنیم که چطور می تونیم همین قابلیت را بین دو پراسس پیاده سازی کنیم؟! 
خوب برای اینکار اول باید دو تا پراسس داشته باشیم و بعد با استفاده از فراخوانی های سیستمی بین این دو Pipe ایجاد کنیم. برای داشتن دو تا پراسس میتونیم یه دونه شو، ایجاد کنیم و با استفاده از فراخوانی تابع سیستمی ()fork ازش یه فورک بگیریم. اینجوری میتونیم دو تا پراسس داشته باشیم. به کد زیر دقت کنید:

{{< highlight c "linenos=table,hl_lines=6 9 14,linenostart=1" >}}
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>

int main(void) {
  pid_t   childPid;

  if((childPid = fork()) == -1) {
    perror("fork");
    exit(EXIT_FAILURE);
  }

  if(childPid == 0) {
    // Child Context                                                                                                                                                                                                                          
    printf("Child process: %d\n", (int)getpid());
    exit(EXIT_SUCCESS);
  }
  else {
    // Parent Context                                                                                                                                                                                                                         
    printf("Parent process: %d\n", (int)getpid());
    exit(EXIT_SUCCESS);
  }
  return(0);
}
{{< /highlight >}}

میتونیم کد بالا را توی یه فایل به نام fork.c ذخیره و با استفاده از دستور زیر کامپایلش کنیم:

{{< highlight sh >}}
gcc -O -o fork fork.c
{{< /highlight >}}

حالا توی همین مسیری که هستیم به فایل به نام fork ایجاد شده که میتونیم اجراش کنیم و خروجی شو ببنیم:

{{< highlight sh >}}
./fork
Parent Pid: 9781
Child Pid: 9782
{{< /highlight >}}

خوب کد بالا به همین راحتی برامون دو تا پراسس ساخت که شناسه اونها 9781 و 9782 هستن.

یه کوچولو ببینیم واقعا چه اتفاقی افتاد. وقتی کد بالا می خواد اجرا بشه هسته سیستم عامل (اینجا Linux/Mint) یه پراسس ایجاد میکنه و اون پراسس تابع main را فراخوانی می کنه. پس اگه ما یه دونه  main داشته باشیم به یه پراسس خواهیم رسید. توی تابع main پراسس ایجاد شده تابع سیستمی ()fork را صدا می زنه. هر پراسسی که این تابع را صدا میزنه، هسته سیستم عامل عین خودشو ایجاد می کنه! در واقع دقیقا بعد از فراخوانی ()fork ما دو تا پراسس جداگانه خواهیم داشت که دقیقا مشابه همند. اما کدهایی که بعد از فراخوانی ()fork مینویسیم مشخص میکنه هر کدوم از این دو میخوان چه کاری انجام بدن.


مقداری که تابع ()fork برمیگردونه برای ما مهمه:

-   اگه "1-" برگردوند: یعنی ایجاد پراسس دوم ممکن نسیت و خطایی رخ داده.
    
-   اگه "0" برگردوند: یعنی پراسس دوم با موفقیت ایجاد شده و شناسه جدید براش درنظر گرفته شده.
    
-   اگه مقدار بجز موارد بالا برگردونه درواقع این همون شناسه تخصیص داده شده به پراسس دوم هستش.

نکته مهم اینکه:
 اگه عملیات ()fork با موفقیت انجام شه مقدار برگشتی برای پراسس اول (پراسسی که ()fork را صدا زده)، شناسه پراسس دوم خواهد بود و مقدرا برگشتی ()fork برای پراسس دوم (که یک نسخه مشابه از پراسس اول هستش) مقدار "0" خواهد بود. اگه متوجه نشدید یبار دیگه این جمله را بخونید و خوب درکش کنید! 

حالا که فهمیدیم چطور دو تا پراسس داشته باشیم بریم سراغ اینکه بین این دو یه دونه Pipe ایجاد کنیم و از پراسس اول به پراسس دوم یه استریگ ارسال کنیم. 
برای ایجاد Pipe بین دو پراسس از تابع سیستمی ()pipe میتونیم استفاده کنیم. یادمون باشه که Pipe ی که به این روش ایجاد میشه مشابه Pipe روی کنسول هستش و جریانی از داده ها را از یک پراسس به پراسس دیگری منتقل می کنه. کد زیر با استفاده از همین تابع سیستمی بین دو پراسس یه دونه پایپ ایجاد میکنه:


{{< highlight sh "linenos=table,hl_lines=10,linenostart=1" >}}
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>

int main(void) {
  pid_t   childPid;
  int     fd[2];

  pipe(fd);

  if((childPid = fork()) == -1) {
    perror("fork");
    exit(EXIT_FAILURE);
  }

  if(childPid == 0) {
    // Child Context                                                                                                                                                                                                                          
    printf("Child process: %d\n", (int)getpid());
    exit(EXIT_SUCCESS);
  }
  else {
    // Parent Context                                                                                                                                                                                                                         
    printf("Parent process: %d\n", (int)getpid());
    exit(EXIT_SUCCESS);
  }
  return(0);
}
{{< /highlight >}}

اگه دقت کنید ما قبل از ()fork تابع ()pipe را صدا زدیم. بنابراین قبل از اینکه پراسس دوم ایجاد شه ما ()pipe را ساختیم. وقتی ()pipe با موفقیت ایجاد شه دو تا File Descriptor میسازه و اونها را توی آرایه ای که بهش پاس دادیم ذخیره میکنه.دومین File Descriptor برای نوشتن (write) روی pipe و اولین File Descriptor برای خوندن (read) از pipe بکارمون میاد. وقتی پراسس دوم ایجاد میشه همونطوری که بالا دیدیم هر دو مشابه همن لذا هر دو ابجکت pipe را با یه عینک می بینن، درواقع محتوای آرایه File Descriptor برای هر دو مشابه هست. برای اینکه منظورمو برسونم ازتون میخوام شکل زیر را ببینید:

<center>![Linux Pipes](/images/pipe_fork_example.png)</center>

اما چیزی که ما دنبالشیم این نیست. ما میخواهیم یک پراسس بتونه روی pipe بنویسه و دومی ازش بخونه. یعنی چیزی مشابه شکل زیر:

<center>![Linux Pipes](/images/pipe_final_example.png)</center>

برای اینکار لازمه پراسس ها File Descriptor هایی که لازم ندارن را ببندند. کد زیر برامون چیزی که توی شکل دوم نمایش داده شده را انجام میده:

{{< highlight sh "linenos=table,hl_lines=19 25,linenostart=1" >}}
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>

int main(void) {
  pid_t   childPid;
  int     fd[2];

  pipe(fd);

  if((childPid = fork()) == -1) {
    perror("fork");
    exit(EXIT_FAILURE);
  }

  if(childPid == 0) {
    // Child Context                                                                                                                                                                                                                          
    close(fd[0]);
    printf("Child process: %d\n", (int)getpid());
    exit(EXIT_SUCCESS);
  }
  else {
    // Parent Context                                                                                                                                                                                                                         
    close(fd[1]);
    printf("Parent process: %d\n", (int)getpid());
    exit(EXIT_SUCCESS);
  }
  return(0);
}
{{< /highlight >}}

خوب چیزی که ما میخواستیم این بود به پراسس اول (parent) بتونه با استفاده از pipe به پراسس دوم (child)  یه استرینگ بفرسته. برای این کار پراسس اول فقط لازم داره روی pipe بتونه wite  کنه لذا File Descriptor مربوط به read  را میبنده (close) و عکس این موضوع برای پراسس دوم باید اتفاق بیفته.

خوب خوب! حالا مونده آخرین بخش کارمون. یعنی استفاده از pipe ایجاد شده و ارسال استرینگ از پراسس اول به دوم. کد زیر این کارو برامون انجام میده:

{{< highlight sh "linenos=table,hl_lines=24 25 33 34 35,linenostart=1" >}}
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <string.h>

int main(void) {
  pid_t childPid;
  int   fd[2];
  int   nbytes;
  char  string[] = "A sample string!";
  char  readBuffer[80];

  pipe(fd);

  if((childPid = fork()) == -1) {
    perror("fork");
    exit(EXIT_FAILURE);
  }

  if(childPid == 0) {
    // Child Context                                                                                                                                                                                                                          
    close(fd[1]);
    nbytes = read(fd[0], readBuffer, sizeof(readBuffer));
    close(fd[0]);
    printf("[Child] Received string: %s\n", readBuffer);
    exit(EXIT_SUCCESS);
  }
  else {
    // Parent Context                                                                                                                                                                                                                         
    int status;
    close(fd[0]);
    write(fd[1], string, (strlen(string)+1));
    close(fd[1]);
    wait(&status);
    exit(EXIT_SUCCESS);
  }
  return(0);
}
{{< /highlight >}}


کد بالا را میتونید توی یه فال بنام pipe.c ذخیره کنید و با دستورات زیر کامپایل و اجراش کنید:

{{< highlight sh >}}
gcc -O -o pipe pipe.c
./pipe
[Child] Received string: A sample string!
{{< /highlight >}}

حالا به سه تا نکته دقت کنیم:

-   اول اینکه پراسس دوم (child) فقط File Descriptor مربوط به read را لازم داره و وقتی کارش تموم میشه اونو میبنده و پراسس اول (parent) فقط File Descriptor مربوط به write را لازم داره و وقتی کارش تموم میشه اونو میبنده.
    
-   پراسس اول تا خاتمه پراسس دوم منتظر (wait) می مونه.
    
-   تابع سیستمی read از یک File Descriptor -اینجا همون[0]fd- تعداد مشخصی بایت -اینجا همون(sizeof(readBuffer- را میخونه و توی بافر -اینجا همون readBuffer- ذخیره می کنه. توضیحی مشابه همین برای تابع سیستمی write هم مصداق داره.

پیروز باشید.
