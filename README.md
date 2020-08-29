# [vue](http://weiyunbyte.com)
###vue移动端UI框架
* [Vant UI](https://github.com/youzan/vant.git) 有赞开发的轻量、可靠的小程序 UI 组件库
* [Nut UI](https://github.com/jdf2e/nutui) 京东开发的一套移动端轻量级组件库
* [Cube UI](https://github.com/didi/cube-ui) 滴滴开发的Vue提供的出色的移动ui lib工具
* [Mint UI](https://mint-ui.github.io/#!/zh-cn) 饿了么开源的移动端UI组件库（很久没更新了）
###letsencrypt利用acme.sh生成通配符的https证书
* 安装:  
```
curl  https://get.acme.sh | sh
```
* 并创建 一个 bash 的 alias, 方便你的使用: 
```
alias acme.sh=~/.acme.sh/acme.sh
```
* 生成证书:（阿里云服务器）
```
acme.sh  --issue -d yoururl.com  -d '*.yoururl.com'  --dns dns_ali
```
* copy/安装 证书：  
```
acme.sh --install-cert -d yoururl.com -d *.yoururl.com \
--key-file       /path/to/keyfile/in/nginx/key.pem  \
--fullchain-file /path/to/fullchain/nginx/cert.pem \
--reloadcmd     "service nginx force-reload"
```
* 自动升级acme.sh:  
```
acme.sh  --upgrade  --auto-upgrade
```
* 修改nginx 443配置