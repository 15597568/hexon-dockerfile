# hexon-dockerfile
自己创建hexon的docker镜像
## 使用方法：
### 1: 创建文件夹
``` sh
mkdir ~/hexonImage
```
### 2:  进入该文件夹并创建新文件DockerFile，并写入以下内容
```dockerfile
FROM node:lts-slim

# 设置workdir,指向 hexo 的目录
WORKDIR /hexon

# 安装环境
RUN \
  mkdir /blog && \
  apt-get update && \
  apt-get install git -y && \
  git clone https://github.com/gethexon/hexon /hexon && \
  npm install -g hexo pnpm && \
  apt-get clean

# 公开服务器端口
EXPOSE 5777 

# 挂载hexo博客目录的路径
VOLUME /blog

# 构建基础服务器和配置（如果不存在），然后启动
CMD \
  if [ ! -d "/hexon/server/data" ]; then \
    pnpm install; \
    pnpm run setup; \
  fi; \
  pnpm start;
```

~~新手福利~~ 如果不知道怎么操作就直接运行下面的命令吧。
```sh
echo "FROM node:lts-slim

# 设置workdir,指向 hexo 的目录
WORKDIR /hexon

# 安装环境
RUN \
  mkdir /blog && \
  apt-get update && \
  apt-get install git -y && \
  git clone https://github.com/gethexon/hexon /hexon && \
  npm install -g hexo pnpm && \
  apt-get clean

# 公开服务器端口
EXPOSE 5777 

# 挂载hexo博客目录的路径
VOLUME /blog

# 构建基础服务器和配置（如果不存在），然后启动
CMD \
  if [ ! -d "/hexon/server/data" ]; then \
    pnpm install; \
    pnpm run setup; \
  fi; \
  pnpm start;" > Dockerfile
```

### 3: 开始构建镜像
```sh
docker build -t [镜像名称] .
```
例子
```sh
docker build -t hexon .
```

> 提醒：在中国大陆，由于网络原因，镜像构建可能会失败。最简单的方法就是换个时间再试一次，但是成功率贼低QAQ。

### 4: 建立docker-compose.yml文件
```yml
services:
  hexo_hexon:
    image: [镜像名称]
    container_name: [容器名称]
    volumes:
    # 将本机的hexo的文件夹重定向到容器中的/blog
      - [hexo博客的位置]:/blog
    ports:
    # 选择对应的端口
      5777:5777
    restart: no
```
例子
```yml
echo "
services:
  hexo_hexon:
    image: hexon
    container_name: hexon
    volumes:
    # 将本机的hexo的文件夹重定向到容器中的/blog
      - /root/blog:/blog
    ports:
    # 选择对应的端口
      5777:5777
    restart: no" > docker-compose.yml
```

启动容器
```sh
docker-compose up -d
```

### 5: 进行设置
```sh
docker exec -it [容器名称] /bin/bash
pnpm run setup
```
然后根据提示输入内容。

遇到选择端口选项（第一个输入），则按下回车（即选择默认5777）

选择hexo博客路径选项（第二个输入），则输入/blog并按下回车。

第三个输入则是输入用户名。

第四个输入则是输入密码。

设置完成后，输入
```sh
exit
```
退出。

### 5: 重启容器
```sh
docker restart [镜像名称]
```
例子
```sh
dcoker restart hexon
```

### 6: 检查是否启动成功
在浏览器里输入 http://localhsot:5777 检查是否启动成功。

### 7: 清理工作
在构建完镜像并启动容器后，大部分人可能再也用不到这些东西。可以删除~/hexonImage文件夹以节省空间。
```sh
rm -rf ~/hexonImage
```

# 参考/特别感谢
1. [给Hexon升了个级](https://laosu.ml/2022/10/05/%E7%BB%99Hexon%E5%8D%87%E4%BA%86%E4%B8%AA%E7%BA%A7/?highlight=hexon)
2. [Hexon](https://github.com/gethexon/hexon)
