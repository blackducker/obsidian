工欲善其事必先利其器，在使用linux平台进行开发时，无论ssh远程登陆还是图形窗口，只要掌握 [**shell**](shell) , [**vim**](vim) ,[ **tmux**](tmux) , 就可以覆盖开发调试测试等几乎所有场景。我不喜欢玩转某某工具箱，只要掌握上述三个工具，以及工具自带的核心功能，就可以花费很少的精力实现高手80%的操作，性价比很高。专注于玩花工具，不如专注于用做的事。

## bash

- 命令检索功能: Ctrl + r
- 光标跳转行头: Ctrl + a
- 光标跳转行尾: Ctrl + e
ps: 必装 `sudo apt install sl`
## zsh

具体这个bash的变种比bash有啥特性不知道，单纯就是为了 oh-my-zsh 这个工具装的。
oh-my-zsh 自带主题，命令补全，通过 plugin 提供了很多工具的缩写 alias，必须 enable 的
`plugins=(docker git tmux extract gitignore cp command-not-found sudo vi-mode z command-not-found emoji)`

- 以git为例，熟悉git的缩写之后，很多常用命令可以变的很简洁
	gst
	glo

- tmux的话，就是
	ta
	tl
## [miniforge](https://github.com/conda-forge/miniforge)跨平台的包管理工具

由于我的开发环境跨越 window linux mac，每个平台上都有一款主流的包管理工具( scoop apt brew )，但是大多都首先网络环境安装非常缓慢。那么有没有一个操作统一的包管理工具，覆盖基础的软件包呢，miniforge 就可以做到这一点，包含的 mamba 工具安装速度非常快，又适应国内环境，近似 conda 所以对 python 的支持很好，但是很多基础包也做了集成，诸如 vim tmux wget。值得注意的是 window 平台上很多包容易找不到。因此，我一般优先使用 mamba ，其次再用主流的包管理器.

windows:
vim python git 