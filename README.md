# Archlinx下构建Jekyll环境

安装ruby：

```
# pacman -S ruby
```

安装Jekyll：

```
# gem install jekyll
```

在`~/.bashrc`中添加以下代码：

```
PATH="$(ruby -e 'print Gem.user_dir')/bin:$PATH"
```

