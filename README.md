# luci-app-frpc

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
### 4、编译