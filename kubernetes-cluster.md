---
title: "راهنمای سریع نصب کلاستر Kubernetes با کاربرد Development"
date: 2019-08-24T05:53:40-04:00
draft: true
tags: ["Kubernetes", "CentOS", "Docker", "Install", "Cluster with 4 Nodes", "نصب کوبرنتیز"]
---
توی این پست میخوام درباره نصب و پیکربندی کلاستر `Kubernetes` بنویسم. توی پست های قبلی مفهوم `Container` ها را تشریح کردم و به اندازه کافی عمق گرفتیم که `Container` ها چین و چطور توی لینوکس ایجاد و  نگهداری میشن. برای ایجاد `Container` ها به روش حرفه ای ابزار های متعددی در اختیارمون قرار داره که بنظرم بهتریناشون `Docker` و `LXC` هستش. توی اینترنت و مخصوصا سایت `Docker` هم به اندازه کافی راهنما برای ساختن `Container` ها با استفاده از `Docker` وجود داره، بهمین خاطر تصمیم گرفتم فعلا درباره `Container` ساختن با `Docker` پستی نزارم و بریم سراغ `Kubernetes`. 

به زبان ساده `Kubernetes` ابزاری برای استقرار، مقیاس پذیری و مدیریت خودکار برنامه های مبتنی بر تکنولوژی `Containerization` هستش. میتونیم به جای "استقرار، مقیاس پذیری و مدیریت خودکار" از واژه "ارکستراسیون" هم استفاده کنیم. وقتی سرورهای محصولی توی لایه `Production` قرار میگیرن نیازمندی ها متعددی پیدا میکنن، بعنوان مثال:

-   با چه مکانیزمی بار یا `Work Load` روی سرور های مختلف توزیع و تنظیم شه.
-   چطور سرورهای مختلف توی `Zone` های ایزوله و مخصوص خودشون استقرار پیدا کنن.
-   چطور فیچرهای جدید توی لایه های مختلف ارتقا پیدا کنن.
-   چطور خطاهای سخت افزاری مدیریت شن.
-   چطور با تغییرات بار تعداد سروها تغییر کنند.    
-   و...

اگه محصولمون کاربر کمی داشته باشه یا معماری ساده ای داشته باشه خوب براحتی میشه با ابزارهایی که `Linux` در اختیارمون قرار میده و ابزارهای مدیریت تنظیمات مثل `SALT` نیازهامونو برطرف کنیم. اما هر چه محصول بالغ تر شه و توسعه پیدا کنه، ارکستراسیون سرویس ها پیچیده و پیچیده تر میشه. `Kubernetes` به ما کمک میکنه تا با این چالش ها به سادگی برخورد کنیم و تمرکزمون روی راه حل ها باقی بمونه و کمتر به چگونگی و چطور ها فکر کنیم. البته باید یه نکته مهم را درنظر داشته باشیم و اون اینه که سرویس های ما باید مبتنی بر تکنوژی `Containerization` توسعه داده باشن و کمینه از نیازمندی های سیستم های توزیع شده را ارضا کنن.

خوب همینقدر مقدمه کفایت میکنه و شروع کنیم به نصب کلاستر خودمون. حواسمون باشه تنظیمات و روش نصبی که توی این پست میزارم خیلی ساده است و مناسب محیط توسعه هستش. در واقع هدف اصلی این پست هم همینه که کمکمون کنه برای تست سرویس هامون یک کلاستر دم دستی و کوچیک آماده کنیم. بهمین خاطر اکیدا توصیه می کنم از این کلاستر توی لایه های `Stage` و `Production` هرگز استفاده نکنید. علاوه بر این لازمه یه سری مفاهیم مثل `Pod`، `KubeAdmin` و … را بدونیم تا درک روشنی درباره این پست پیدا کنیم.


ما نیاز به ۴ تا سیستم داریم و من با استفاده از `VMWare` روی یک لبتاب چهار تا `CentOS` نصب کردم و شکل زیر تنظیمات اولیه هر کدومشو نشون میده:

<center>![Kubenetes](/images/kubernetes-cluster.png)</center>

نود هایی که به عنوان `Worker` درنظر گرفته میشن درواقع ماشین هایی هستن که قرار سرویس مورد نظر اونجا اجرا شه. من برای هر کدوم: سی پی یو، رم و دیسک را بترتیب `2GB`، `2Core` و `20GB` تنطیم کردم. از طرفی برای نود `Master` منابع بیشتری درنظر گرفتم چرا که `KDE` روش نصب دارم تا بتونم `Dashboard` را روش داشته باشم. برای این نود مقادیر سی پی یو، رم و دیسک را بترتیب `4GB`، `4Core` و `20GB` هستش.

خوب بریم و نصب را شروع کنیم. برای اینکار مراحلی که لازمه را در ادامه به ترتیب تشریح میکنم.

اول از همه فایل `hosts` همه ماشین ها را مشابه زیر تنظیم می کنیم:


{{< highlight c "linenos=table,linenostart=1" >}}
vi /etc/hosts
172.16.245.100 k8s-master
172.16.254.10  k8s-node-01
172.16.254.20  k8s-node-02
172.16.254.30  k8s-node-03
{{< /highlight >}}

برای اینکه از شر فایروال هم راحت بشیم باید ترتیبشو بدیم! بنابراین دستورات زیر را روی همه نودها اجرا می کنیم:


{{< highlight c "linenos=table,linenostart=1" >}}
systemctl disable firewalld 
systemctl stop firewalld
{{< /highlight >}}


خیلی از کارامون قراره روی کنسول باشه بنابراین لازمه روی ماشین ها `bash-completion` هم نصب کنیم:


{{< highlight c "linenos=table,linenostart=1" >}}
yum install bash-completion
{{< /highlight >}}

حالا یه کار که اصلا ربطی به نصب کلاستر `Kubernetes` نداره روی `Master` باید انجام بدیم اونم نصب `Google Chrome` هستش. من باهاش راحتم و میخوام `Dashboard` را باهاش ببینم. اگه شما لازم نداشتیدش میتونید از این مرحله رد بشید. برای این کار اول چنتا پکیج لازم داریم، دستورات زیر را روی کنسول `Master` اجرا می کنیم تا برامون نصبشون کنن:


{{< highlight c "linenos=table,linenostart=1" >}}
yum install libXScrnSaver libappindicator-gtk3
yum install redhat-lsb-core
yum install liberation-fonts
{{< /highlight >}}

حالا میتونید `Chrome` مخصوص `CentOS` را دانلود، نصب و ازش استفاده کنیم:


{{< highlight c "linenos=table,linenostart=1" >}}
rpm -i ~/Downloads/google-chrome-stable_current_x86_64.rpm
google-chrome --no-sandbox
{{< /highlight >}}

خوب از اینجا به بعد کار اصلیمون شروع میشه. اول از همه `Docker` را روی همه نودهای کلاستر نصب میکنیم:


{{< highlight c "linenos=table,linenostart=1" >}}
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io
{{< /highlight >}}

حالا باید سرویس `Docker` را تنظیم کنیم که بعد بوت نودها خودکار بالا بیاد، برای اینکار دستور زیر را روی همه نودها اجرا می کنیم:


{{< highlight c "linenos=table,linenostart=1" >}}
systemctl enable docker
{{< /highlight >}}

حالا باید `Swap File System` را روی همه نودها غیر فعال کنیم. برای این کار لازمه خط مربوط به این مورد را در `fstab` کامنت کنیم و دستور غیر فعال کردنشو صادر کنیم!:


{{< highlight c "linenos=table,linenostart=1" >}}
vi /etc/fstab
	remove any swap entry.
swapoff -a
{{< /highlight >}}

خوب حالا همه چی برای نصب `Kubernetes` آماده شده. چیزی که نیاز داریم اینه که سه تا برنامه `kubelet` ،`kubeadm` و `kubectl` را نصب و تنظیم کنیم. دستورات زیر را روی همه نودهای کلاستر اجرا می کنیم:


{{< highlight c "linenos=table,linenostart=1" >}}
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF

setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system

systemctl enable kubelet
reboot
{{< /highlight >}}

حواسمون باشه که دستور آخر داره نود ها را ریبوت می کنه!
بعد از اینکه همه نودها بالا اومدن دستورات زیر را روی `Master` اجرا می کنیم:


{{< highlight c "linenos=table,linenostart=1" >}}
kubeadm config images pull
kubeadm init --apiserver-advertise-address=172.16.245.100 --pod-network-cidr=192.168.0.0/16
{{< /highlight >}}

توی خروجی دستور آخر به توکن نمایش داده میشه. از این توکن قرار ما استفاده کنیم. بنابراین باید یجا نگهش داریم. حالا دستور زیر را روی `Master` اجرا میکنیم، البته حواسمون باشه توکنی که بالا گرفتیم را استفاده کنیم:


{{< highlight c "linenos=table,linenostart=1" >}}
kubeadm join 172.16.245.100:6443 --token zqpog5.iq91kwhab82pz3mr --discovery-token-ca-cert-hash sha256:eef0818dc01669bd19f93df0882ffa271063051cef0f9e4e0a89abed5eb55fd8
{{< /highlight >}}

یه دونه `Environment Variable` هم لازم داریم که روی `Master` ست کنیم:


{{< highlight c "linenos=table,linenostart=1" >}}
vi ~/.bashrc
	export KUBECONFIG=/etc/kubernetes/admin.conf
source ~/.bashrc
{{< /highlight >}}
دستورات زیر را روی `Master` اجرا میکنیم تا فعلا کارمون باهاش تموم شه:


{{< highlight c "linenos=table,linenostart=1" >}}
kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
{{< /highlight >}}

حالا باید چک کنیم `CoreDNS` درست کار میکنه یا نه:


{{< highlight c "linenos=table,linenostart=1" >}}
kubectl get pods --all-namespaces
{{< /highlight >}}

اگه مراحل بالا را درست دنبال کرده باشیم خروجی دستور بالا خوشحالمون می کنه!

حالا وقتشه `Worker` ها را به `Cluster` اضافه کنیم. با استفاده از توکنی که داریم میتونیم روی `Worker` ها دستور ریز را اجرا کنیم:


{{< highlight c "linenos=table,linenostart=1" >}}
kubeadm join 172.16.245.100:6443 --token zqpog5.iq91kwhab82pz3mr --discovery-token-ca-cert-hash sha256:eef0818dc01669bd19f93df0882ffa271063051cef0f9e4e0a89abed5eb55fd8
{{< /highlight >}}
برای اینکه مطمئن شیم، میتونیم روی `Master` نود های `Worker` را چک کنیم:


{{< highlight c "linenos=table,linenostart=1" >}}
kubectl get nodes
{{< /highlight >}}

خوب امیدوارم تا اینجا همه چی خوب پیش رفته باشه. حالا وقتشه که `Dashboard` را روی `Master` نصب کنیم. برای اینکار دستورات زیر را روی `Master` اجرا می کنیم:


{{< highlight c "linenos=table,linenostart=1" >}}
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml
kubectl get pods --all-namespaces
{{< /highlight >}}

برای `Login` کردن هم لازمه توکن مخصوص ایجاد کنیم. دستورات زیر اینکار را انجام میدن:


{{< highlight c "linenos=table,linenostart=1" >}}
kubectl create serviceaccount cluster-admin-dashboard-sa
kubectl create clusterrolebinding cluster-admin-dashboard-sa --clusterrole=cluster-admin --serviceaccount=default:cluster-admin-dashboard-sa
{{< /highlight >}}

حالا روی `Master` کنسول جدیدی باز می کنیم و دستور زیر را وارد میکنیم:


{{< highlight c "linenos=table,linenostart=1" >}}
kubectl proxy
{{< /highlight >}}
و با استفاده از [Open DashBoard](http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/) میتونیم `Dashboard` را باز کنیم و با توکنی که مرحله قبل تولید شد برای `Login` استفاده کنیم. اگه توی نصب `Dashboard` مشکلی داشتید [Reference](https://docs.giantswarm.io/guides/install-kubernetes-dashboard) میتونه کمک کنه.


پیروز باشید.
