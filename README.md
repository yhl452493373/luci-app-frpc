# 适用于amd64的luci-app-frpc

从[lede仓库](https://github.com/coolsnowwolf/luci/tree/master/applications/luci-app-frpc)拉取，做了一些调整：

+ 去掉了frpc无用的`vhost_http_port `和`vhost_https_port`
+ 增加了`附加参数`设置，可以通过该设置增加类似`user=xxx`，`meta_token=yyy`的配置
+ 默认不启动定时注册（定时注册默认设为0）
+ 调整部分翻译

如果你是非amd64（x86-64）平台，请自行修改`FRP_URL`、`FRP_HASH`以及[root/etc/init.d/frp](root/etc/init.d/frp)中`local frp_url`中与架构相关的部分，将`sdk_url`改为对应架构的sdk地址

## LEDE SDK下编译（包含[最新的frp](https://github.com/yhl452493373/openwrt-frp.git))
```bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
FRP_URL=$( curl -sL https://api.github.com/repos/fatedier/frp/releases | grep -P 'download/v[\d.]+/frp_[\d.]+_linux_amd64.tar.gz' | awk -F '"' '{print $4}' | awk 'NR==1{print}' )
FRP_VERSION=$( echo $FRP_URL | awk -F '/' '{print $8}' | awk '{gsub(/v/,"");print $1}' )
FRP_HASH=$( curl -sL https://codeload.github.com/fatedier/frp/tar.gz/v0.51.3 | sha256sum | awk -F ' ' '{print $1}' )
sdk_url=https://archive.openwrt.org/releases/21.02.6/targets/x86/64/openwrt-sdk-21.02.6-x86-64_gcc-8.4.0_musl.Linux-x86_64.tar.xz
golang_commit=b6468a6bd5b61d811ae2567a9814aed44354e555
golang_url=https://codeload.github.com/coolsnowwolf/packages/tar.gz/

rm -rf sdk && mkdir sdk && curl "$sdk_url" | tar -xJ -C ./sdk --strip-components=1
cd sdk
./scripts/feeds update -a

rm -rf feeds/packages/lang/golang
curl "$golang_url$golang_commit" | tar -xz -C "feeds/packages/lang" --strip=2 "packages-$golang_commit/lang/golang"

rm -rf feeds/packages/net/frp
git clone https://github.com/yhl452493373/openwrt-frp.git feeds/packages/net/frp

sed -i -e 's/^PKG_VERSION.*/PKG_VERSION:='''$FRP_VERSION'''/' feeds/packages/net/frp/Makefile
sed -i -e 's/^PKG_HASH.*/PKG_HASH:='''$FRP_HASH'''/' feeds/packages/net/frp/Makefile

rm -rf feeds/luci/applications/luci-app-frpc
git clone https://github.com/yhl452493373/luci-app-frpc feeds/luci/applications/luci-app-frpc
sed -i -e 's/\tlocal frp_version=.*/\tlocal frp_version='''$FRP_VERSION'''/' feeds/luci/applications/luci-app-frpc/root/etc/init.d/frp
chmod 755 feeds/luci/applications/luci-app-frpc/root/etc/init.d/frp
chmod 755 feeds/luci/applications/luci-app-frpc/root/etc/uci-defaults/luci-frp

./scripts/feeds install -a
make defconfig
make package/luci-app-frpc/clean
make package/luci-app-frpc/compile V=s
```
编译后，在 `bin/packages/x86_64/luci` 下

