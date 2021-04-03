# 商用版H5棋牌游戏! 支持千万级用户同时在线!!! 

 **发布此版本的初衷:**

朋友聚会打个牌玩个麻将什么的，现在的游戏平台都要充值才能玩，而且还需要下载app、安装和注册，很麻烦......不能尽兴娱乐。

所以，整合了一套稳定的商用 **H5网页版** 游戏分享出来，让大家摆脱平台的束缚，无监控、无控制、公平公正，支持手机、电脑、Pad，只要有浏览器就可以玩。游戏服务端使用golang开发，采用微服务架构，支持高并发场景需求，使用容器进行封装，简化了复杂的配置过程，小白按照下面的搭建教程也能轻松在几分钟内搭建好平台。

![](https://github.com/openinggame/qp/blob/master/GameScreenshot/1.jpg)
![](https://github.com/openinggame/qp/blob/master/GameScreenshot/5.jpg)
![](https://github.com/openinggame/qp/blob/master/GameScreenshot/8.jpg)
![](https://github.com/openinggame/qp/blob/master/GameScreenshot/9.jpg)
![](https://github.com/openinggame/qp/blob/master/GameScreenshot/7.jpg)

**免责声明:**

   此版本仅限测试(试玩)使用，因平台搭建使用人(开发者)原因导致的任何纠纷、责任等需平台搭建使用人(开发者)自行承担全部责任和赔偿一切损失。

​    欢迎加入我们的Telegram群组，获取最新进度和反馈问题。

#### Telegram群组:

![](https://github.com/openinggame/qp/blob/master/GameScreenshot/tg.jpg) 

​       入群链接: https://t.me/joinchat/Ptl66CwP5WxkODZl

由于大家都知道的原因，国内可能无法打开并使用Telegram，所以打不开是因为你不会科学上网。

## 搭建教程

​    游戏服务集群运行环境：Centos7.x + docker + docker-compose

### 1. 环境安装

#### 1.1 安装docker(centos7.x)

​    已经安装docker的忽略本步骤，yum安装方法自行查询，windows系统安装方法自行查询。

- 安装docker

  ```shell
  [xxx@docker ~]# curl -fsSL get.docker.com -o get-docker.sh
  [xxx@docker ~]# sudo sh get-docker.sh --mirror Aliyun
  ```

- 创建docker用户组，将当前用户加入docker组

  ```bash
  [xxx@docker ~]# sudo groupadd docker
  [xxx@docker ~]# sudo usermod -aG docker $USER
  ```

- docker 使用方法

  ```bash
  [xxx@docker ~]# sudo systemctl enable docker
  [xxx@docker ~]# sudo systemctl start docker
  ```

- docker配置阿里云镜像加速

  ```shell
  [xxx@docker ~]# sudo mkdir -p /etc/docker
  [xxx@docker ~]# sudo tee /etc/docker/daemon.json <<-'EOF'
  {
    "registry-mirrors": ["https://lz2nib3q.mirror.aliyuncs.com"]
  }
  EOF
  [xxx@docker ~]# sudo systemctl daemon-reload
  [xxx@docker ~]# sudo systemctl restart docker
  ```

#### 1.2 安装docker-compose

##### 1.2.1 linux系统安装方法

- 在 Linux 上的也安装十分简单，从 官方 GitHub Release 处直接下载编译好的二进制文件即可。例如，在 Linux 64 位系统上直接下载对应的二进制包。

```bash
[xxx@docker ~]# sudo curl -L https://github.com/docker/compose/releases/download/1.28.6/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
[xxx@docker ~]# sudo chmod +x /usr/local/bin/docker-compose
```

##### 1.2.2 macos、window系统安装方法

- Compose 可以通过 Python 的包管理工具 pip 进行安装，也可以直接下载编译好的二进制文件使用，甚至能够直接在 Docker 容器中运行。`Docker Desktop for Mac/Windows 自带 docker-compose 二进制文件，安装 Docker 之后可以直接使用`。

- 使用`pip`以下命令下载

  ```shell
  [xxx@docker ~]# pip install docker-compose
  ```

- 官方release下载地址：https://github.com/docker/compose/releases



### 2. 创建并启动游戏服务集群

#### 2.1 下载游戏服务集群需要的依赖

##### 2.1.1 创建工作目录

```powershell
[xxx@docker ~]# mkdir -p /data
[xxx@docker ~]# mkdir -p /data/etcd-data
```

##### 2.1.2 安装git服务 (已安装的可直接忽略)

```powershell
[xxx@docker ~]# yum install git -y
```

##### 2.1.3 克隆依赖到本地并将文件解压到工作目录

```powershell
[xxx@docker ~]# cd /data
[xxx@docker ~]# git clone https://github.com/openinggame/qp.git
[xxx@docker ~]# cd qp
[xxx@docker ~]# tar zxf mongodb.tar.gz -C /data
[xxx@docker ~]# tar zxf mysqldb.tar.gz -C /data
```

- 第一遍下载很可能是不成功的，甚至第二遍、第三遍都会不成功，但是不要慌，这也许只是网络的问题，下载过程有一个时间限制，超过了这个限制就会下载失败，多重复几次，总会成功的；当最后一行结尾出现 “done” 这个词时，就表示下载成功了。
- /data 工作目录结构 

```powershell
[xxx@docker ~]# tree /data
data
├── etcd-data     # etcd data-dir
├── mongo_data    # mongodb 数据卷
├── mysql         # mysql 数据卷
└── qp            # docker compose
    └── docker-compose.yml
```



#### 2.2 下载镜像

这步可以略过，执行到 步骤2.3 启动集群时会先检测镜像，若没有会自动下载，但镜像下载失败会启动失败。

推荐不要略过本步骤，先把镜像pull到本地。

```shell
[xxx@docker ~]# docker pull mysql:8.0.23
[xxx@docker ~]# docker pull mongo:4.4.4
[xxx@docker ~]# docker pull quay.io/coreos/etcd:v3.2.32
[xxx@docker ~]# docker pull wurstmeister/zookeeper
[xxx@docker ~]# docker pull wurstmeister/kafka:2.12-2.3.0
[xxx@docker ~]# docker pull redis:latest
[xxx@docker ~]# docker pull openinggame/web:v1
[xxx@docker ~]# docker pull openinggame/server:v1
```

#### 2.3 创建集群网络

```shell
[xxx@docker ~]# docker network create -d bridge game
```

#### 2.4 启动集群

如果没有执行 2.1 的步骤，这里消耗的时间比较久，速度取决你的网络质量。

##### 2.4.1 修改docker-compose.yml文件

```powershell
修改第10行 web服务的IP地址 <ip地址> 为服务器的IP地址：
#    第10行   - API_HOST=<ip地址>

#例如IP地址为：192.168.1.6 ,修改docker-compose.yml中web服务的API_HOST的值。（第 10 行）
  web:
    container_name: web0
    image: openinggame/web:v1
    ports:
      - "80:80"
    environment:
      - API_HOST=192.168.1.6    #修改这行的 IP 地址为你的服务器IP地址
    networks:
      - game
    depends_on:
      - server
# ...
```

- ##### Cento7.x 查询ip地址方法（推荐使用固定IP地址）

```powershell
[xxx@docker ~]# ifconfig eth0 | grep 'inet ' | tr -s ' ' | cut -d ' ' -f3
[xxx@docker ~]# 192.168.1.6
```

##### 2.4.2 通过 docker-compose 启动游戏服务集群

```shell
[xxx@docker ~]# cd /data/qp
[xxx@docker ~]# docker-compose up -d
```



#### 3. 开始游戏

##### 3.1 打开浏览器（谷歌浏览器）输入游戏服务器的IP地址

- ###### 游戏的地址就是上面查询到的服务器IP地址 : http://192.168.1.6

##### 3.2 首次登陆，使用游客登陆，点击 游客登陆 按钮进入游戏。

- ###### 执行docker-compose启动集群后，要等待所有服务器启动起来才可以进入游戏。

- ![](https://github.com/openinggame/qp/blob/master/GameScreenshot/0.jpg)

##### 3.3 进入游戏后，点击 立即注册，绑定手机号码（号码随意输入11位数字）。

- ###### 手机号码可以随意输入 11 位数字，自己记牢就可以了。

- ![](https://github.com/openinggame/qp/blob/master/GameScreenshot/2.jpg)

##### 3.3.1 输入手机号码，然后点击获取验证码。

- ![](https://github.com/openinggame/qp/blob/master/GameScreenshot/3.jpg)

##### 3.3.2 输入密码，然后点击 绑定 按钮，绑定成功后，下次登陆可以使用 手机号码+密码 的方式登陆。

- ###### 默认每个游戏账户有100万游戏币，不够了，重新注册新账户即可。

- ![](https://github.com/openinggame/qp/blob/master/GameScreenshot/10.jpg)

### 最后，祝大家玩的愉快！如果遇到问题，可以加入Telegram群组，获取最新进度和反馈问题。

#### Telegram群组:

​   入群链接: https://t.me/joinchat/Ptl66CwP5WxkODZl
