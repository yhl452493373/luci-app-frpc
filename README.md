# luci-app-frpc

从[lede仓库](https://github.com/coolsnowwolf/luci/tree/master/applications/luci-app-frpc)拉取，做了一些调整：

+ 去掉了frpc无用的`vhost_http_port `和`vhost_https_port`
+ 增加了`附加参数`设置，可以通过该设置增加类似`user=xxx`，`meta_token=yyy`的配置
+ 默认不启动定时注册（定时注册默认设为0）
+ 调整部分翻译

## 以下为在LEDE中编译的方法
 
### 1、进入到克隆下来的lede目录
```bash
cd lede
```
### 1、删除LEDE原有包
```bash
rm -rf feeds/luci/applications/luci-app-frpc
```
### 2、下载新包
```bash
git clone https://github.com/yhl452493373/luci-app-frpc feeds/luci/applications/luci-app-frpc
```
### 3、赋予文件执行权限
```bash
chmod 755 feeds/luci/applications/luci-app-frpc/root/etc/init.d/frp
chmod 755 feeds/luci/applications/luci-app-frpc/root/etc/uci-defaults/luci-frp
```
### 4、选中luci-app-frpc并编译
