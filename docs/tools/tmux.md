### 1> Session

1. 查看/切换session: prefix s
2. 离开Session: prefix d
3. 重命名当前Session: prefix $

## 2> Winodw
分屏快捷键prefix : `Ctrl + b` 为前缀

1. `上下分屏：ctrl + b 再按 _`
2.  水平分屏：ctrl + b 再按 -`
3. `切换屏幕：ctrl + b 再按 Up,Down,Left,Right`
4. `关闭一个终端：ctrl + b 再按 x`

## 3>面板调整
1. 重新排列当前窗口下的所有窗格： ctrl + b 再按空格键
2. z 最大化当前所在面板
3. - ! 将当前面板置于新窗口,即新建一个窗口,其中仅包含当前面板
4.  alt+方向键 以5个单元格为单位移动边缘以调整当前面板大小

## 4> oh-my-tmux

https://github.com/gpakosz/.tmux

Installing in `~` :
```shell
$ cd
$ git clone --single-branch https://github.com/gpakosz/.tmux.git
$ ln -s -f .tmux/.tmux.conf
$ cp .tmux/.tmux.conf.local .
```
- 一个美观的皮肤
-  加入了C-a作为新的`<prefix>
-  `<prefix> m` 鼠标模式，自由可以切换窗口，拖动边框
