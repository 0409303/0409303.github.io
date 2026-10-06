[toc]



## 库

### 安装库

```shell
pip install requests
pip install spotipy

# 🇨🇳 中国用户必看：解决下载慢/超时报错（换国内镜像源）
# 临时使用一次（推荐，最简单）
pip install 库名 -i https://pypi.tuna.tsinghua.edu.cn/simple

# 如果不想每次都加 -i，可以永久设置（一行命令搞定）
# 设置后，以后直接输入 pip install 库名 就会自动走国内源，网速飞起
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

清华大学：https://pypi.tuna.tsinghua.edu.cn/simple
阿里云：https://mirrors.aliyun.com/pypi/simple
腾讯云：https://mirrors.cloud.tencent.com/pypi/simple

# 查看pip配置列表
pip config list
```



### 批量安装

```shell
pip install -r requirements.txt
```

requirements.txt

```
requests==2.31.0
numpy==1.26.0
pysimplegui==4.60.5.1
spotipy==2.25.2
pylast==5.3.0
musicbrainzngs==0.7.1
python-dotenv==1.0.1
pandas==2.2.3
# 安装 pandas-stubs 后，PyCharm 就能像查字典一样，在你写 df. 或 pd. 时，精准地给出 pandas 特有的方法补全和参数提示，让编码效率和准确性大大提升。
pandas-stubs==2.2.3
mutagen==1.47.0
tqdm==4.66.5

# excel
openpyxl==3.1.5

# 压缩
# pyzipper 是 Python 标准库 zipfile 的增强版，最大的特点是支持 AES 加密的 ZIP 文件
pyzipper==0.4.0
# 支持多种压缩算法（如 LZMA2, BZip2, Deflate 等）和加密方法（如 7zAES），并提供命令行工具，方便直接操作 7z 文件
py7zr==0.21.0
# 支持 RAR3 和 RAR5 格式的归档文件，以及多卷归档和密码保护
rarfile==4.2

# 提供在 CPU 或 GPU 上运行的张量（Tensor）计算，并支持自动微分，用于构建和训练神经网络
torch==2.4.0

# 第三方库
# 这是一个网易云音乐的第三方库
git+https://github.com/zixing131/pyncm.git
```



### 查看安装结果

```
pip show python-dotenv
```



## 版本问题

### win7环境

1. python <= 3.8
2. pycharm <= 2019.3

pycharm内置的pip版本过低，会导致安装时出现尝试从**源码编译**依赖包的问题

而且自带的pip 19.0.3 有一个已知问题，它会阻止你通过常规命令升级 pip 本身

所以需要通过以下命令升级pip，再来安装依赖

```shell
# 强制把 pip 升级到最新版
easy_install -U pip

# 更新其他构建工具
python -m pip install --upgrade setuptools wheel 
```

升级完 pip 后，如果安装其他包时再次遇到“Installing build dependencies”卡住或报错，记得用 `--no-build-isolation` 参数，让 pip 直接使用你刚刚升级好的 setuptools，而不是再去临时下载构建依赖。
