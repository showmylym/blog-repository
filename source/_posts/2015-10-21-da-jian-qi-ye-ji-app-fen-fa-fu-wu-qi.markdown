---
layout: post
title: "搭建企业级app分发服务器"
date: 2015-10-21 22:14:12 +0800
comments: true
categories: web
---
###一、配置证书: 
####1、生成key  
　　在apache2目录下，新建ssl目录  

    sudo mkdir /private/etc/apache2/ssl
    cd /private/etc/apache2/ssl
    sudo ssh-keygen -f server.key
　　这里不设置RSA口令  
####2、生成请求文件
    sudo openssl req -new -key server.key -out request.csr

####3、生成SSL证书
    sudo openssl x509 -req -days 365 -in request.csr -signkey server.key -out server.crt
　　至此，生成/private/etc/apache2/ssl/目录下"request.csr" "server.crt" "server.key" "server.key.pub"四个文件  
###二、配置apache让其支持https
####1、修改/private/etc/apache2/httpd.conf文件，去掉下面三行前面的注释符号#
    LoadModule ssl_module libexec/apache2/mod_ssl.so
    LoadModule socache_shmcb_module libexec/apache2/mod_socache_shmcb.so
    Include /private/etc/apache2/extra/httpd-ssl.conf
    Include /private/etc/apache2/extra/httpd-vhosts.conf
####2、修改/private/etc/apache2/extra/httpd-ssl.conf ，去掉下面两行前面的 '#'
    SSLCertificateFile "/private/etc/apache2/ssl/server.crt"
    SSLCertificateKeyFile "/private/etc/apache2/ssl/server.key"
####3、修改/private/etc/apache2/extra/httpd-vhosts.conf，在 'NameVirtualHost * :80' 后面添加：
    NameVirtualHost *:443
　　在文件末尾添加：  

    <VirtualHost *:443>
    SSLEngine on
    SSLCipherSuite ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP:+eNULL
    SSLCertificateFile /private/etc/apache2/ssl/server.crt
    SSLCertificateKeyFile /private/etc/apache2/ssl/server.key
    ServerName localhost
    DocumentRoot "（此为Apache工作根目录）"
    </VirtualHost>
###三、架设企业级应用服务。  
####1、生成install.html
　　代码如下所示  
    
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <style type="text/css">
    	* {
    		font-size: 52px;
    	}

    	a {
    		line-height: 2.5;
    		padding: 20px;
    	}
    </style>
    安装步骤：<br/>
    1、首次安装需要点击<a href="https://192.168.1.239/server.crt" id="text">“安装证书”</a><br/>
    2、否则直接点击 <br/>
    <a href="itms-services://?action=download-manifest&url=https://192.168.1.239/g0/mlf.plist" id="text">“安装平台版app”</a> <br/>
####2、手机需先安装server.crt。
　　因此拷贝/private/etc/apache2/ssl/server.crt到Apache根目录下，install.html中编写超链接指向该证书文件，供设备下载安装。
####3、将mlf.plist mlf.ipa mlf.png存放在自定义目录下，并在install.html中指向它们。
　　mlf.plist代码如下所示

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
	<key>items</key>
	<array>
		<dict>
			<key>assets</key>
			<array>
				<dict>
					<key>kind</key>
					<string>software-package</string>
					<key>url</key>
					<string>https://192.168.1.239/g0/mlf.ipa</string>
				</dict>
				<dict>
					<key>kind</key>
					<string>full-size-image</string>
					<key>needs-shine</key>
					<true/>
					<key>url</key>
					<string>https://192.168.1.239/g0/mlf.png</string>
				</dict>
				<dict>
					<key>kind</key>
					<string>display-image</string>
					<key>needs-shine</key>
					<true/>
					<key>url</key>
					<string>https://192.168.1.239/g0/mlf.png</string>
				</dict>
			</array>
			<key>metadata</key>
			<dict>
				<key>bundle-identifier</key>
				<string>cn.meilif.mlfinhouse</string>
				<key>bundle-version</key>
				<string>2.0</string>
				<key>kind</key>
				<string>software</string>
				<key>title</key>
				<string>美丽范</string>
			</dict>
		</dict>
	</array>
    </dict>
    </plist>

