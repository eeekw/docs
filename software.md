## brew

替换镜像源

```sh
git -C "$(brew --repo)" remote set-url origin https://mirrors.ustc.edu.cn/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git
```

重置

```shell
git -C "$(brew --repo)" remote set-url origin https://github.com/Homebrew/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://github.com/Homebrew/homebrew-core
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git
```



## RubyGems

替换镜像源

```shell
gem sources  #列出默认源
gem sources --remove https://rubygems.org/
gem sources -a https://gems.ruby-china.com/
```

## 其它问题

### Pod install 速度慢

速度慢的原因不在pod命令，而是github上的代码库访问速度慢，加速方法，开启代理

`git config --global http.proxy socks5://127.0.0.1:1080`　代理所有git库

`git config --global http.https://github.com.proxy socks5://127.0.0.1:1080`　只代理github库

> `socks5://127.0.0.1:1080` 默认代理端口1080

`git config --global --unset http.proxy`　移除代理

`git config --global --unset http.https://github.com.proxy` 移除github代理

> 关闭代理时　记得移除代理配置