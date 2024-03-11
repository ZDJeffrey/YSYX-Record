# 配置Chisel开发环境
## 安装Scala CLI
### 使用`apt`安装
```shell
curl -sS "https://virtuslab.github.io/scala-cli-packages/KEY.gpg" | sudo gpg --dearmor  -o /etc/apt/trusted.gpg.d/scala-cli.gpg 2>/dev/null
sudo curl -s --compressed -o /etc/apt/sources.list.d/scala_cli_packages.list "https://virtuslab.github.io/scala-cli-packages/debian/scala_cli_packages.list"
sudo apt update
sudo apt install scala-cli
```
> Note:在安装`scala-cli`后，需要为其设置代理
> ```shell
> # 开启power模式
> scala-cli config power true
> # 设置代理
> scala-cli config httpProxy.address 'http://127.0.0.1:7890'
> ```
> 若设置代理错误，会导致`scala-cli`命令无法运行，此时需要直接去修改`~/.local/share/scalacli/secrets/config.json`（此路径与官方文档所给路径不同，可以使用`find`进行寻找）

### 测试
```shell
echo 'println("Hello")' | scala-cli -
```
输出Hello
## 安装JDK
```shell
# Ensure the necessary packages are present:
apt install -y wget gpg apt-transport-https

# Download the Eclipse Adoptium GPG key:
wget -qO - https://packages.adoptium.net/artifactory/api/gpg/key/public | gpg --dearmor | tee /etc/apt/trusted.gpg.d/adoptium.gpg > /dev/null

# Configure the Eclipse Adoptium apt repository
echo "deb https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list

# Update the apt packages
apt update

# Install
apt install temurin-17-jdk
```

## 安装构建工具`Mill`
```shell
# 下载`millw`脚本 添加执行权限
curl -L https://raw.githubusercontent.com/lefou/millw/0.4.11/millw > mill && chmod +x mill
# 移动到系统环境变量目录
sudo mv mill /usr/local/bin/
```


