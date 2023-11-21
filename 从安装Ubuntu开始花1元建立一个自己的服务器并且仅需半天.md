# 安装Ubuntu，使用1元近乎免费搭建自己的服务器，放上第一个个人网站
 
---


@[TOC](文章目录)

---

# 前言

   关于安装Ubuntu和自建服务器的文章可能有很多，但是很多里面并没有出现很多问题方面的内容，以及很多相应原理和解释，下面在内容中我会详细介绍我出现的问题，和解决办法，以及部分内容的解释。

# 一、安装Ubuntu（清除Windows）
首先去官网下载想要的Ubuntu的镜像文件[https://cn.ubuntu.com/](https://cn.ubuntu.com/)
![Ubuntu下载](https://img-blog.csdnimg.cn/aacb59e631d44a219e2313e69ed44018.png#pic_center)
## 制作启动U盘
  准备一个不小于8GU盘在[https://rufus.ie/zh/](https://rufus.ie/zh/)下载rufus软件，如下图
  ![启动U盘制作软件](https://img-blog.csdnimg.cn/c2b4594196cb47528e85244fe161b17b.png#pic_center)
如下图步骤，选择U盘设备，选择自己的镜像文件路径，其他设置默认，点击开始。
![启动U盘制作](https://img-blog.csdnimg.cn/7f46381f1ed84bbfb1620585a7ee5246.png#pic_center)

## 安装系统

接下来重启电脑，按你自己品牌进启动设置的选项，选择try install  Ubuntu...... 选项，图片没拍，但是这个可借鉴的还是比较多的，然后进入Ubuntu安装页面：先在旁边选择简体中文，再点击Install Ubuntu
  ![Ubuntu安装首页](https://img-blog.csdnimg.cn/5553daa5a0e94acb9f15b0c981c55a45.png#pic_center)
后面跟着指引进行，安装类型选择“**其他类型**”，下面是默认分配：
![默认分配](https://img-blog.csdnimg.cn/32e3763c23b049e59ae5bf86399f6fed.jpeg#pic_center)
也可以自定义分配：但是其中的**biosgrub**是必须要有的：下图中我一开始忘记设置了，后来安装的时候设置了：boot其实不用设置很大，swap交换空间设置一般为内存的两倍，还有 / 挂载设置五六十G够用，剩下的全给/home，在这里除了swap直接选为交换空间，其余都设置为主分区就可以，只需要更改挂载。
![自定义分配](https://img-blog.csdnimg.cn/c3b73991185f489fa08f46e38f110319.jpeg#pic_center)
接下来选择地区，shanghai，选择键盘布局EnglishUS.后续根据引导进入系统，重启。
我在这一步遇到了点问题，重启之后提示我拔出设备后提示我找不到引导开不了机，但是直接进boot确定就进了。
期间使用了boot-repair但是没怎么解决

```powershell
sudo add-apt-repository ppa:yannubuntu/boot-repair
sudo apt-get update
sudo apt-get install -y boot-repair
boot-repair
```

最后，通过在开机时候进入BIOS设置，把BOOT勾选，解决这个问题。
![硬盘启动](https://img-blog.csdnimg.cn/aa43d65613e74ba2ba2b6e86c3863d11.jpeg#pic_center)
1.最后进入系统打开终端查询是否为root用户，以便得到更高权限：
>查询命令：

```powershell
sudo whoami
```
2.如果配置正确，你将看到输出为root，表示你的用户现在有sudo权限。
>如过不是，执行以下步骤即可：

```powershell
sudo adduser <用户名> sudo
```
3.请将<用户名>替换为你想要添加到sudo组的用户名。
>下面刷新用户组信息

```powershell
sudo su - <用户名>
```
通过这一步，你进入了用户的shell，并加载了新的组成员信息。
执行第1.操作查询你加入root组的用户
# 二、建立一个自己的服务器
在以下的操作中，如果执行了某一步想要退回命令行可以按“q”，或者ctrl+c退出，安装的所有内容和下载的内容都使用默认，不用设置。
## 1.首先实现内网穿透
## 1.1  安装必要软件
安装SSH服务

```powershell
sudo apt-get update
sudo apt-get install openssh-server
```
安装Web服务器（Nginx为例）

```powershell
sudo apt-get install nginx
```
## 1.2 配置SSH访问
通过SSH，你可以在远程终端上管理你的服务器。确保SSH服务已启动：

```powershell
sudo systemctl start ssh
```
## 1.3 配置Web服务器
启动Nginx服务

```powershell
sudo systemctl start nginx
```
## 1.4 检查SSH和nginx状态
使用以下命令检查 SSH 服务的运行状态：查看之后按q退出到命令行

```powershell
sudo systemctl status ssh
```
如果 SSH 服务正在运行，你应该会看到类似下面的输出：

```yaml
● ssh.service - OpenBSD Secure Shell server
   Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2023-11-28 10:30:42 UTC; 10min ago
 Main PID: 1234 (sshd)
    Tasks: 1 (limit: 4653)
   CGroup: /system.slice/ssh.service
           └─1234 /usr/sbin/sshd -D
```
>在输出中，Active: active (running) 表示 SSH 服务正在运行。

使用以下命令检查 Nginx 服务的运行状态：

```powershell
sudo systemctl status nginx
```
如果 Nginx 服务正在运行，你应该会看到类似下面的输出：

```yaml
● nginx.service - A high-performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2023-11-28 10:35:18 UTC; 5min ago
     Docs: man:nginx(8)
  Process: 2345 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
 Main PID: 2346 (nginx)
    Tasks: 2 (limit: 4653)
   CGroup: /system.slice/nginx.service
           ├─2346 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           └─2347 nginx: worker process
```
>在输出中，Active: active (running) 表示 Nginx 服务正在运行。

## 1.5 设置SSH和nginx开机启动和如何关闭

启用 SSH 服务自动启动

```powershell
sudo systemctl enable ssh
```
启用 Nginx 服务自动启动

```powershell
sudo systemctl enable nginx
```
你也可以关闭 SSH 和 Nginx 服务，可以使用 systemctl 命令。以下是关闭这两个服务的方法：
>关闭 SSH 服务

```powershell
sudo systemctl stop ssh
```

>关闭 Nginx 服务

```powershell
sudo systemctl stop nginx
```
这两个命令将立即停止相应的服务。如果你希望服务在系统启动时不再自动启动，你可以使用以下命令：

>禁用 SSH 服务自动启动

```powershell
sudo systemctl disable ssh
```

>禁用 Nginx 服务自动启动

```powershell
sudo systemctl disable nginx
```

## 1.6修改设置后要重启服务

如果要重启 SSH 服务，可以使用：

```powershell
sudo systemctl restart nginx
```

如果要重启 Nginx 服务，可以使用：

```powershell
sudo systemctl restart nginx
```
>上面这些命令的一些介绍：

systemctl 命令的语法是 sudo systemctl [动作] [服务名]，其中 [动作] 是你想要执行的操作，比如 start、stop、restart，而 [服务名] 是服务的名称，比如 nginx。
内网中，SSH服务默认使用的端口号是22。这是默认设置，但实际上，系统管理员可以配置SSH服务以使用不同的端口号。如果端口号已更改，你可以通过以下几种方式查找：

>  查看SSH配置文件：
通过查看SSH服务器的配置文件，你可以找到配置的端口号。在大多数Linux系统上，SSH配置文件是 /etc/ssh/sshd_config。你可以使用文本编辑器（如nano或vim）查看该文件：

```powershell
sudo nano /etc/ssh/sshd_config
```
在配置文件中，找到并查看 Port 行，它会告诉你SSH服务使用的端口号：

```powershell
Port 22
```
如果有其他端口号，将显示在这一行上。
>使用nmap工具：
nmap 是一个网络探测工具，也可以用于扫描端口。如果你的系统中没有安装 nmap，可以使用以下命令安装：

```powershell
sudo apt update
sudo apt install nmap
```
然后使用 nmap 扫描SSH服务所在的主机：

```powershell
nmap -p 1-65535 <SSH_HOST>
```
其中 <SSH_HOST> 是你SSH服务所在的主机的IP地址或主机名。扫描结果将显示SSH服务使用的端口号。

使用以上方法之一，你应该能够找到SSH服务使用的端口号。如果端口号不是默认的22，你可能需要记住这个端口号以便成功连接到SSH服务。

## 1.7内网访问

通过以上步骤你已经建立了一个内网服务器，你可以通过以下命令查询IP地址来在同一个网络环境进行访问，出现nginx默认页面就说明你已经成功了：
连WiFi时看类似wlp3s0的192.168.xxx.xxx

```powershell
ifconfig
```
或者

```powershell
ip link
```
或者

```powershell
ifconfig -a
```

输入上面这个命令之后，会出现以下三个：

```powershell
lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff
3: wlp3s0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff
```
其中类似“ens33”的是以太网接口，“wlp3s0”是WiFi

其他设置可以指定SSH监听的端口：默认监听22

```powershell
server listening on 0.0.0.0 port 22
```

## 1.8 配置防火墙
使用ufw（Uncomplicated Firewall）来配置防火墙规则














```powershell
sudo apt-get install ufw
sudo ufw allow 22  # 允许SSH
sudo ufw allow 80  # 允许HTTP
sudo ufw enable
```

## 1.9 设置静态IP
连接WiFi时每次启动都会随机分配一个内网IP地址，我们需要设置它为固定的IP以便于访问。
在开始设置前计算机本身自带了一个“**01-network-manager-all.yaml**”文件在下面的路径：

```powershell
/etc/netplan/
```
1. 首先需要禁用这个自带的，这个时候会断网，不过不要担心，下面来设置一个自己的：
>停用 Network Manager

```powershell
sudo systemctl stop NetworkManager
sudo systemctl disable NetworkManager
```
2. 创建你自己的 netplan 配置文件

```powershell
sudo vim /etc/netplan/01-netcfg.yaml
```
下面是一个连接WiFi进行修改的例子：

```yaml
network:
  version: 2
  renderer: networkd
  wifis:
    wlp2s0:
      dhcp4: no
      dhcp6: no
      addresses: [192.168.31.223/24]
       routes:
        - to: 0.0.0.0/0
          via: 192.168.31.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
      access-points:
        "xiaomiE617":
          password: "12345678"
```
运行后可能会出现一些警告，但是这是关于权限的警告，你的 netplan 配置文件的权限过于宽松。你可以确保只有管理员（root）有权访问：
>使用以下命令更改文件权限：

```powershell
sudo chmod 600 /etc/netplan/01-netcfg.yaml
sudo chmod 600 /etc/netplan/01-network-manager-all.yaml
```
如果你是以太网连接替换以下内容

```yaml
renderer: networkd
 wifis:
    wlp2s0:
      dhcp4: no
      dhcp6: no
     addresses: [192.168.31.223/24] 
替换为下面
renderer: networkd
ethernets:
    ens33:
      dhcp4: no
      addresses: [192.168.31.223/24]
删除
access-points:
        "xiaomiE617":
          password: "12345678"
```
保存并关闭文件。

应用配置更改：

```yaml
sudo netplan apply
```

下面对上面内容做一个简短的的解释：

 - enp3s0 通常代表以太网接口（Ethernet），以 en 开头。
   
  - wlp2s0 通常代表无线接口（Wireless LAN），以 wl 开头。
   
  - 一般来说，如果你的服务器使用有线连接，你应该选择 enp3s0，如果使用无线连接，你应该选择 wlp2s0。
   -192.168.31.189是你希望设置的静态IP地址
   
  - 192.168.31.1是你的网关地址，8.8.8.8和8.8.4.4是Google的DNS服务器地址，你可以根据需要更改。
   
-  为了确保不与其他设备的IP地址冲突，你需要将后面的189尽可能设置的大一点
   
-   通常来说，局域网内的IP地址是按照子网掩码进行划分的。IPv4地址是32位的，而子网掩码决定了网络部分和主机部分的位数。
   
 -  常见的子网掩码是24位，表示前24位是网络部分，剩下的8位是主机部分。在这种情况下，可以分配256个不同的IP地址（2^8），其中0和255是保留的，因此可用的IP地址范围是1到254。
   
-   如果你使用了24位的子网掩码，那么在设置静态IP地址时，可以在最后一个部分设置的最大值是254，确保不与其他设备的IP地址冲突。例如，可以将你的静态IP地址设置为192.168.31.254。
   
-   当然，实际使用中，你可能不需要使用所有的254个IP地址，而是只使用其中的一部分。
   
-   在网络中，CIDR（Classless Inter-Domain
   Routing）表示法用于指定IP地址和其分配的子网掩码。CIDR标记的格式是IP地址后跟斜杠（/）和一个数字，例如 /24。
   
-   /24 表示一个包含 24
   位网络前缀的子网掩码。这意味着前24位用于网络标识，剩余的8位用于主机标识。在IPv4中，每个IP地址有32位，因此对于 /24
   的子网，将有2^8（256）个可能的IP地址。
   
-   具体到 addresses: [192.168.31.189/24] 这个例子：
   192.168.31.189 是你的设备的IP地址。 /24 意味着前24位用于网络标识，剩余的8位用于主机标识，因此这个设备所在的子网包含256个可能的IP地址，范围是从 192.168.31.0
   到 192.168.31.255。 使用CIDR表示法有助于更灵活地分配IP地址，以适应不同大小的网络。例如，/24
   表示256个IP地址，而 /16 表示65536个IP地址，以此类推。
-    "xiaomiE617":
          password: "12345678"
路由器的名称和密码
>在进行更改后查询IP，如果不是你设置的IP地址，打开WiFi设置，那里有一个含有你配置文件内容的一个WiFi，点击连接它就可以了，后面重启后一直连的就是这个，你的内网IP也固定了。

>使用 journalctl 命令来检查网络相关的日志

```powershell
journalctl -u systemd-networkd
```

# 三、实现公网访问
因为本次实现是低成本实现，所有花费，最多的是注册域名，新用户在阿里云等很多平台有优惠1元可以首年注册.cn等的顶级域名。所以使用的是内网穿透和动态DNS，从免费的服务提供商做一个反代。
## 1.1 免费体验
步骤如下：

1. 使用反向代理服务，如Ngrok、Serveo等。
2. 注册并获取一个公共域名（通常是由反向代理服务提供的）。
3. 将反向代理服务的配置指向你的本地服务器的内部IP地址和端口。
4. 通过提供的公共域名和端口访问你的服务。
>下面我将以使用 Ngrok 为例，说明如何使用反向代理服务进行公网访问。Ngrok 是一个提供反向代理服务的工具，它可以让你将本地服务映射到一个公共域名和端口上，从而在公网上访问你的服务。

步骤 1: 注册 Ngrok 账户
访问 Ngrok 网站：

在浏览器中打开 [Ngrok 官方网站](https://ngrok.com/)，然后注册一个账户。

获取授权令牌：

登录 Ngrok 账户后，你可以在 [https://dashboard.ngrok.com/get-started](https://dashboard.ngrok.com/get-started) 页面获得你的授权令牌。

步骤 2: 下载并安装 Ngrok
下载 Ngrok：

访问 [Ngrok 下载页面](https://ngrok.com/download) 选择合适的版本下载。
>这个页面有各个系统完整的操作步骤


解压下载的 Ngrok 压缩包。

```powershell
sudo tar -xvzf ~/Downloads/ngrok-v3-stable-linux-amd64.tgz -C /usr/local/bin
```
认证 Ngrok 账户：

使用你在步骤1中获得的授权令牌进行认证。

```powershell
./ngrok authtoken your_auth_token
```
步骤 3: 配置并运行 Ngrok
运行 Ngrok：

```powershell
ngrok http 80
```
这会将本地服务的端口80映射到Ngrok提供的公共域名和端口上。
获取公共域名：

在Ngrok运行后，你会看到输出中包含了一个以http://或https://开头的公共域名。例如：

```powershell
Forwarding                    http://your_ngrok_subdomain.ngrok.io -> http://localhost:80
```
**your_ngrok_subdomain.ngrok.io** 就是你的公共域名。

步骤 4: 访问你的服务
现在，你可以通过提供的公共域名和端口来访问你的本地服务。在上面的例子中，你可以通过访问 http://your_ngrok_subdomain.ngrok.io 来访问本地服务。

请注意，Ngrok免费版有一些限制，包括每小时最多连接数和随机生成的子域名。
这个子域名周期只有几个小时，所以并不稳定。
>停止运行按Ctrl+c

如果你在后台运行了 Ngrok，可以使用以下步骤来停止：

打开终端。

运行以下命令查找 Ngrok 进程的 PID（进程ID）：

```powershell
ps aux | grep ngrok
```
这将显示包含 "ngrok" 关键词的进程列表，找到与 Ngrok 相关的行，并记下其 PID。

使用 kill 命令停止 Ngrok 进程。假设 PID 是 12345：

```powershell
kill 12345
```
或者，如果你使用强制终止：

```powershell
kill -9 12345
```
请确保在终止 Ngrok 进程之前，你不再需要其服务，因为停止后将无法再次通过相同的 Ngrok 地址访问你的本地服务。

## 1.2 使用固定域名并实现内网穿透
首先去阿里云注册新用户，在搜索栏搜索域名注册，找到新用户首年一元注册，根据上面的提示一步步进行操作，最后实名认证，通过审核，你就拥有你自己的一个域名了。
这里使用Cloudflare Tunnel来进行内网穿透。
Cloudflare Tunnel是一个服务，旨在帮助用户通过Cloudflare的全球网络将其服务安全、快速地连接到互联网。Cloudflare Tunnel中的一个关键功能是Argo Tunnel，该功能允许你通过Cloudflare的网络将你的服务器与公网连接起来。
Cloudflare Tunnel 是一款隧道软件，可以快速安全地加密应用程序到任何类型基础设施的流量，让您能够隐藏你的 web 服务器 IP 地址，阻止直接攻击，从而专注于提供出色的应用程序。

1. 在官网注册登录后，点击Add a site
![cloudflare首页](https://img-blog.csdnimg.cn/cd0b7d78d4af498aabb367af2a66c992.png#pic_center)
2. 把你注册好的域名放上面，然后根据提示进行下一步，后面会给你提示，在你的注册商那里更换DNS解析，并给你他们的解析地址，以阿里云为例：点击域名解析：
![域名解析第0步](https://img-blog.csdnimg.cn/3837b3d8437b4535a5bdb549b71e85c9.png#pic_center)
3. 点击解析设置：
![域名解析第1步](https://img-blog.csdnimg.cn/dbc5d0752b7b4a1ba413d88e2232ea8a.png#pic_center)
tom.ns.cloudflare.com	
autumn.ns.cloudflare.com
![第二步](https://img-blog.csdnimg.cn/7f87f7b481be4517afbbd12e9566ac98.png#pic_center)
下面开通Cloudflare Zero Trust。在这里：
![第三步](https://img-blog.csdnimg.cn/89be8d0896f94562bd3a4cf2536d06e3.png#pic_center)
点击Zero Trust
![第四步](https://img-blog.csdnimg.cn/6f4c1e0da6174ac4aae50aeddb385470.png#pic_center)
点击Access，点击Tunnels

![第五步](https://img-blog.csdnimg.cn/d32a0277b04b4186bccbcae6c78de68a.png#pic_center)

点击Create a tunnel 
![第六步](https://img-blog.csdnimg.cn/380d7110dc2248db95165b75c377448c.png#pic_center)
后面根据提示操作

这里选择免费的那个，但是需要绑定信用卡或者PayPal。（可以绕过）随便点击下绕过去。

这里需要安装cloudflear,选择合适的版本，有两个选择，一个是未安装执行的，一个是已安装执行的。直接复制到命令行即可。推荐根据网址到GitHub下载好，再执行 下面的命令安装：

```powershell
sudo dpkg -i cloudflared-linux-amd64.deb
```
之后复制官网给的已安装的命令运行。
![第七步](https://img-blog.csdnimg.cn/a17103f8510841eaa4665b4f8ba23ced.png#pic_center)
下面中间填你的域名，前面填别的，用来对应下面的url，设置的127.0.0.1默认80端口，前面已经设置了nginx服务默认是在80端口，所以下面你访问这个域名，就是nginx默认的页面。

![第八步](https://img-blog.csdnimg.cn/946fc100425e44d68ef9892337e5c7c1.png#pic_center)
下面你就可以公网访问你的服务器了。

## 1.3 更改端口的内容
这里我以前面nginx的web服务为例，你也可以自行更改。

将您的前端页面（main.html）复制到Nginx的默认网站根目录。默认情况下，Nginx的网站根目录在/var/www/html。

打开终端，使用以下命令将main.html复制到网站根目录：

```powershell
sudo cp /home/lingfeng/web/main.html /var/www/html/
```
确保Nginx已经正确配置。Nginx的配置文件通常位于**/etc/nginx/nginx.conf**或者**/etc/nginx/sites-available/default**。

打开Nginx的配置文件，找到**server**块，并确保其中包含以下指令：

```powershell
root /var/www/html;
index index.html index.htm;   替换为   index main.html

```
重新启动Nginx服务，使配置生效。使用以下命令重启Nginx：

```powershell
sudo service nginx restart
```
现在，当你访问Nginx服务器的IP地址或域名时，会显示你的自定义前端页面（main.html）了。请注意，如果你的前端页面依赖于其他资源文件（如CSS、JavaScript等），你需要将这些文件一并复制到网站根目录，并在前端页面中正确引用这些文件。如果页面未变，建议清楚之前的缓存再重新进。


关于Nginx中设置允许流量通过的端口，是在Nginx配置文件中的listen指令中设置的。例如，要允许HTTP流量通过默认的端口80，你的Nginx配置文件中可能有以下行：

```powershell
server {
    listen 80;
    # 其他配置...
}
```

如果你使用的是HTTPS，你可能会在配置文件中看到类似以下的行：

```powershell
server {
    listen 443 ssl;
    # 其他配置...
}
```
确保listen指令中的端口号与你配置防火墙时允许的端口相匹配。


以上这些设置在下次开机时会自动启动，所以你要做的就是打开你的电脑，就可以拥有一台能连接公网的服务器。
# 总结
写这篇文章为想要拥有一台服务器，但是并没有充裕的资金的个人用户提供了一个基础的教程，如果有上面问题需要修改欢迎提出意见，我部署好的的个人页面启动时间不固定，因为在学校会有断电。其中的web服务你也可以使用python等。



