# 适用于amd64（x86_64）平台lede的luci-app-frpc

从[lede仓库](https://github.com/coolsnowwolf/luci/tree/master/applications/luci-app-frpc)拉取，做了一些调整：

+ 去掉了frpc无用的`vhost_http_port `和`vhost_https_port`
+ 增加了`附加参数`设置，可以通过该设置增加类似`user=xxx`，`meta_token=yyy`的配置
+ 增加了webscoket、quic、wss协议选项
+ 增加了http协议的http2https插件选项
+ 默认不启动定时注册（定时注册默认设为0）
+ 调整部分翻译

## 注意：如果你服务端用了[fp-multiuser](https://github.com/gofrp/fp-multiuser)插件，那么`基础设置`下的`用户名`就相当于该插件的`user`，只需要在`附加参数`中增加`meta_token=xxxx`即可

如果你是非amd64（x86-64）平台，请自行修改`FRP_URL`、[root/etc/init.d/frp](root/etc/init.d/frp)中`local frp_url`中与架构相关的部分，将`sdk_url`改为对应架构的sdk地址.

[/root/etc/init.d/frp](/root/etc/init.d/frp) 参考[kuoruan](https://github.com/kuoruan/luci-app-frpc)的代码做了调整

**如果要在windows下修改文件，请记得将文件的换行改为 `LF`，默认情况下，git拉到windows平台，文件换行符会变成`CR+LF`，这会导致编译完成后，`/etc/init.d/frp`无法执行**

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

./scripts/feeds install -a
make defconfig
make package/luci-app-frpc/clean
make package/luci-app-frpc/compile V=s
```
以上编译时，会拉取最新的frp代码。编译后，在`bin/packages/平台架构/luci`下。amd64（x86_64）在 `bin/packages/x86_64/luci` 下
