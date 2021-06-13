# macOS 终端配置


## iterm2 终端配置

**比较好的 `Iterm2` 终端配色**

- `Cobalt2`：适用 `macOS` 本地及远程主机
- `Cobalt Neon` : 适合远程主机

### 安装 brew

*国外源*

```bash
liwangguideMBP:~ liwanggui$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

*国内源*

```bash
# 安装
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
# 卸载
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/HomebrewUninstall.sh)"
```

### 配置 zmodem（lrzsz）

Mac 默认终端是不支持 `sz` `rz` 命令的，`iterm2` 终端可以配置支持 `sz` `rz`  命令，配置方法如下：

*1. 安装 lrzsz*

```bash
brew install lrzsz
```

*2. 安装 iterm2-zmodem 脚本*

```bash
git clone https://github.com/aikuyun/iterm2-zmodem.git
cp iterm2-zmodem/iterm2*.sh /usr/local/bin/
chmod +x /usr/local/bin/iterm2*.sh
```

*3. 配置 iterm2 Triggers*

```bash
Regular expression: \*\*B0100
    Action: Run Silent Coprocess
    Parameters: /usr/local/bin/iterm2-send-zmodem.sh
Regular expression: \*\*B00000000000000
    Action: Run Silent Coprocess
    Parameters: /usr/local/bin/iterm2-recv-zmodem.sh 
```

按 `command` + `,` 打开配置面板，然后点击 “`Profiles`”, “`Advanced`”， “`Triggers -> Edit`”

> Tips: 正常连接服务器就可以使用 `sz` `rz` 命令了
> 通过添加 profile 配置主机列表

### 常用快捷键

- `command + d` 横向分屏
- `command + shift + d` 水平分屏
- `command + enter` 全屏，取消全屏
- `command + ;` 打开输入历史记录
- `command + f` 打开搜索框
- `option + command + i` 分屏时同时操作多个窗口，重复取消

## oh-my-zsh

### 安装

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

> zsh 主题配置项: `ZSH_THEME="ys"`

### 配置 zsh-syntax-highlighting

```bash
brew install zsh-syntax-highlighting
echo 'source /usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
export ZSH_HIGHLIGHT_HIGHLIGHTERS_DIR=/usr/local/share/zsh-syntax-highlighting/highlighters' >> ~/.zshrc
source  ~/.zshrc
```

### 配置从当前位置删除到行首， ctrl + u

```bash
echo 'bindkey \^U backward-kill-line' >> ~/.zshrc
source  ~/.zshrc
```

## 新增命令无法自动补全解决方法

```bash
rehash  # 执行 rehash 后就可以了
```

## 配置 vim

```bash
$ cat ~/.vimrc
" 语法高亮
syntax on
" 自动检测文件外部更改
set autoread
" 显示标尺
set ruler
" 设置编码
set encoding=utf-8
" 共享剪切板
set clipboard=unnamed
" 总是显示状态栏
set laststatus=2
" 配置(软)制表符为宽度为4
set tabstop=4
set softtabstop=4
" 将制表符转为空格
set expandtab
" 设置缩进的空格数为4
set shiftwidth=4
" 设置自动缩进：即每行的缩进值与上一行相等；使用 noautoindent 取消设置
set autoindent
" 使用 C/C++ 语言的自动缩进方式
set cindent
" 设置C/C++语言的具体缩进方式
set cinoptions={0,1s,t0,n-2,p2s,(03s,=.5s,>1s,=1s,:1s
" 高亮搜索内容
set hlsearch
" 可以删除任意值
set backspace=2
"记录上次编辑位置
au BufReadPost * if line("'\"") > 1 && line("'\"") <= line("$") | exe "normal! g'\"" | endif
```
