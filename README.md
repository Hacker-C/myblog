## About

这几天突然想换一个简洁的博客主题，发现了 `Jekyll`。

2022.3.5 更新：目前正在迁移部署服务，因为使用第三方的自动化部署服务还是有点卡顿，所以我决定将博客部署到国内服务器上去。

- 暂时：http://124.222.44.115:8081  （域名服务商转移中，暂未绑定域名）
- 备用：https://myblog-gamma-jet.vercel.app

## Built With

### Environment 

WSL（ubuntu）：

- OS：WSL
- 博客框架：[Jekyll ](https://jekyllrb.com)
- Ruby2.7
- RubyGems 3.1.2
- G++ / GCC
- Python 3.8.5

CentOS：

- Ruby 
- RubyGems（gem）
- GCC and Make

## Getting Started

我完成一篇创作并部署的的模式：在 Windows 下的 `WSL` 中写文档、预览，然后 copy 相应的文件到 `CentOS` 下，重启服务。在 WSL 中或者其他终端下的 jekyll 安装，请自行搜索。

CentOS（centos 8） 下：

新建文件夹

```bash
mkdir myblog
cd myblog
```

下载我的主题：

```bash
git clone git@github.com:Hacker-C/myblog.git .
```

下载安装 `ruby` 以及其需要的依赖：

```bash
yum -y install ruby ruby-devel rubygems rpm-build
```

修改 `gem` 源为国内镜像：

```bash
gem sources --remove https://rubygems.org
gem sources -a https://gems.ruby-china.com
```

安装 G++/GCC：

```bash
sudo yum -y install gcc gcc-c++ kernel-devel
```

安装 `jekyll`：

```bash
gem install jekyll bundler
```

下载依赖：

```
gem install
```

启动服务：

```bash
bundle exec jekyll serve
```

启动成功：

```
Configuration file: /var/www/myblog/_config.yml
            Source: /var/www/myblog
       Destination: /var/www/myblog/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
                    done in 0.664 seconds.
 Auto-regeneration: enabled for '/var/www/myblog'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
  
```

## Deploy

这里我使用 `Nginx` 反向代理部署在了云服务器（CentOS 8）上。

这里需要准备云服务器，以及 SSH 连接好云服务器，这部分自行搜索完成。

### 打包编译 jekyll 文件

未编译的 jekyll 文件无法直接通过 nginx 部署运行，需要先将其编译打包为 html/css/js/images 的形式，使用 `jekyll build` 即可打包编译，生成的文件默认在 `myblog/-site` 下。

### 配置 nginx 

下载 `nginx`：

```bash
sudo yum -y install nginx
```

修改配置文件（`/etc/nginx`）：

```
cd /etc/nginx
```

可以在 `/etc/nginx` 目录下看到 `nginx.conf`，其中有一行：

```nginx
include /etc/nginx/conf.d/*.conf;
```

意思是将 `/etc/nginx/conf.d/*.conf` 文件的内容引入进去，相当于 C/C++ 的 `include`。

所以可以直接在 `conf.d` 下新建 `servers.conf`：

```bash
cd conf.d
touch servers.conf
```

使用 `vim servers.conf`，输入文件内容如下：

```nginx
# 博客
server {
  listen 8081;
  server_name localhost;
  location / {
    root /var/www/myblog/_site; # 指向刚刚编译生成的文件
    index index.html; # 入口
  }
}
```

测试 nginx 配置文件是否正确：

```
sudo nginx -t
```

显示如下说明无误：

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

启动 nginx 服务：

```bash
sudo service nginx start
```

访问 `http://[服务器主机IP]:8081`，大功告成。

> 需要注意，相应的端口需要在云服务器上开启安全组。

### 绑定域名

域名跨服务商转移中，暂未完成。