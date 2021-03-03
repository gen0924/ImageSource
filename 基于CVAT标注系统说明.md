## 基于CVAT标注系统流程搭建文档

### 构建CVAT标注镜像
* 打开**终端terminal**界面
* 按照下方命令行指令，安装**dokcer**，更多介绍点解[此处](https://docs.docker.com/install/linux/docker-ce/ubuntu/)  
```
sudo apt-get update
sudo apt-get --no-install-recommends install -y \
		  apt-transport-https \
  		ca-certificates \
  		curl \
  		gnupg-agent \
  		software-properties-common
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   sudo add-apt-repository \
 		 "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  			$(lsb_release -cs) \
  			stable"
sudo apt-get update
sudo apt-get --no-install-recommends install -y docker-ce docker-ce-cli containerd.io
```
* 执行下方指令，便于docker运行无需root权限，更多介绍点击[此处](https://docs.docker.com/install/linux/linux-postinstall/)  
```
sudo groupadd docker
sudo usermod -aG docker $USER
```
* 安装**docker-compose**（1.19.0或更高版本），便于运行对容器  
```
sudo apt-get --no-install-recommends install -y python3-pip python3-setuptools
sudo python3 -m pip install setuptools docker-compose
```
* 克隆[**CVAT**源码](https://github.com/opencv/cvat)
```
sudo apt-get --no-install-recommends install -y git
git clone https://github.com/opencv/cvat
cd cvat
```
* **构建镜像**，该过程会花费较长时间（取决于网速等各式情况）
```
docker-compose build
```
*  **启动容器**，会花费一些时间下载公共镜像，例如postgres:10.3-alpine, redis:4.0.5-alpine，同时，创建容器
```
docker-compose up -d
```
* 上次操作成功完成后，可以创建**超级管理员**，具有所有权限，同时可以在终端控制其他用户权限（**标注者、审查者等**）
```
docker exec -it cvat bash -ic 'python3 ~/manage.py createsuperuser'
```
选择用户名、密码设定为超级管理员，更多详细了解可查看[此处](https://docs.djangoproject.com/en/2.2/ref/django-admin/#createsuperuser)

* Google浏览器是唯一支持CVAT的浏览器，因此需要安装谷歌浏览器，可手动下载Google浏览器软件进行安装，也可在终端用指令安装
```
curl https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
sudo sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list'
sudo apt-get update
sudo apt-get --no-install-recommends install -y google-chrome-stable
```
* 打开Google浏览器，进入链接[localhost:8080](http://localhost:8080/)。然后登录用户、密码，然后就可以创建工程、任务等。

### 额外功能描述
* 在进行任何改动后，都需停止所有容器、再重启容易
```
docker-compose down
docker-compose up -d
```
* **数据挂载功能**，在创建任务时可从三个地方选择数据（**本地、挂载路径、URL链接**），要实现数据挂载，需更改文件**docker-compose.yml**, 具体更改如下：
```
volumes:
      - cvat_data:/home/django/data
      - cvat_keys:/home/django/keys
      - cvat_logs:/home/django/logs
      - cvat_models:/home/django/models
      - sharePath:/home/django/share:ro    # 添加此行
```
**备注：将sharePath更改为自己想要挂载的数据路径位置**

*  **多人网页访问**  
创建 **docker-compose.override.yml** 文件并加入以下设置：
```
version: '3.3'
services:
  cvat_proxy:
    environment:
      CVAT_HOST: 0.0.0.0    # 更改为自己的IP
```
创建完上述文件后，执行如下命令，便可实现多人网页访问
```
docker-compose -f docker-compose.yml -f docker-compose.override.yml build
```

###  创建工程和任务
* 点击界面首页中**Create new project **进行工程创建界面  
![创建工程界面](https://github.com/gen0924/ImageSource/raw/main/images/%E5%88%9B%E5%BB%BA%E5%B7%A5%E7%A8%8B.png)  
* 在工程界面下创建任务**Task**，具体界面如下：  
![工程下创建任务界面](https://github.com/gen0924/ImageSource/raw/main/images/%E5%B7%A5%E7%A8%8B%E4%B8%8B%E5%88%9B%E5%BB%BA%E4%BB%BB%E5%8A%A1.png)  
* 创建任务时高级选项说明  
![创建任务时高级选项](https://github.com/gen0924/ImageSource/raw/main/images/%E5%88%9B%E5%BB%BA%E4%BB%BB%E5%8A%A1%E9%AB%98%E7%BA%A7%E8%AE%BE%E7%BD%AE.png)  
* 任务创建完成后，进行任务指派，若存在本地标注文件，可以点击进入任务，然后点击左上角按钮，选择**Upload annotations**按钮选择要导入的标注文件  
![任务创建完成界面](https://github.com/gen0924/ImageSource/raw/main/images/%E4%BB%BB%E5%8A%A1%E5%88%9B%E5%BB%BA%E5%AE%8C%E6%88%90%E7%95%8C%E9%9D%A2.png)

