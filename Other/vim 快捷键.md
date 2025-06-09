
https://github.com/chloneda/vim-cheatsheet

## 快速学习推荐

### 1> 分屏快捷键
prefix : `Ctrl + w` 为前缀

1. 窗口分屏, `prefix + s / v` , 水平 /垂直
2. 窗口切换 `prefix + hjkl` , 左下上右
3. 窗口切换 `prefix + HJKL` , 左下上右
4. 窗口关闭 `prefix + c`

### 2> 文本处理 
1. 移动文本
`:[range]m{address}`
2. 粘贴文本 
`:[range]co{address}`
3. 查找修改文本
`:[range]s/{pattern}/{string}/[flags]`

| [range]取值 | 含义       |
| --------- | -------- |
| .         | 当前行      |
| N         | 第N行      |
| 1,10      | 第 1~10 行 |
| %         | 所有行      |
## vim 代码分析工具 ctags cscope

配置了cscope 和 ctags两个功能，cscope比ctags强的功能是可以查找一个函数的调用有哪些并以列表的形式呈现出来！
## ctags
vim -t [funcs_name]

