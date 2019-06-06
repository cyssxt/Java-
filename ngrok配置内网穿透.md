# ngrok配置内网穿透
目前ngrok1.x开源 2.x尚未开源
## 安装git golang ubuntu环境
<pre><code>
apt-get install golang git
</code></pre>
## clone服务器端源码 
<pre><code>
git clone https://github.com/inconshreveable/ngrok.git ngrok
cd ngrok
</code></pre>

## 申请签名 （可以去腾讯云申请）,然后替换证书 位于assets/client/tls目录下

## 编译（生成文件在bin目录下）
<pre><code>
//window
GOOS=windows GOARCH=amd64 make release-client  
//Mac
GOOS=darwin GOARCH=amd64 make release-client
//linux
make release-server release-client
</code></pre>

## 服务端运行ngrokd
<pre><code>
//服务器后台运行 并且将证书分别放置bin目录下
// tlsKey tlsCrt 证书信息 domain：跟域名信息 httpAddr http请求端口 https请求端口
nohup ./ngrokd -tlsKey=cyssxt.key -tlsCrt=cyssxt.crt -domain="cyssxt.com" -httpAddr=":80" -httpsAddr=":443" &
</code></pre>

## 客户启动 ngrok同级目录新建ngrok.yml和start.bat（ngrok不能直接启动需要通过命令行启动）
<pre><code>
//ngrok.yml内容如下
server_addr: "cyssxt.com:4443"//4443为ngrok默认端口
trust_host_root_certs: false
tunnels:
 oracle://配置oracle服务器
  remote_port: 1523
  proto:
   tcp: 1521
 ftp:
  remote_port: 234//ftp端口映射
  proto:
   tcp: 21
 mstsc:
  remote_port: 3389//window远程桌面端口
  proto:
   tcp: "127.0.0.1:3389"
 http:
  subdomain: ngrok//80 http web
  proto:
   http: 80
   https: 443
//start.bat内容如下
start /b ngrok -log=./ngrok.log -log-level=INFO -config=./ngrok.yml start http mstsc oracle ftp
</code></pre>