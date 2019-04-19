参考： https://gist.github.com/tanyuan/55bca522bf50363ae4573d4bdcf06e2e

在 Ubuntu 下非常简单，只需要三步

1. 安装 [xcape](https://github.com/alols/xcape)

```bash
sudo apt-get install xcape
```

2. 修改按键映射

```bash
# 修改大小写锁定为 ctrl
setxkbmap -option ctrl:nocaps

# 将短按左 ctrl 改为 ESC
 xcape -e 'Control_L=Escape'
 ```
 
 3. 将上面的命令加入到自动启动中，看个人习惯，如： ~/.bashrc 中
