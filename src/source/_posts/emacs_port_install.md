---
title: 安装 Emacs (Emacs Port 版本)
date: 2022-08-17 22:28:33
categories: 
- Emacs
---

这是 Mac 系统下的一个分发版本，具有适配Mac的功能。  
安装步骤
``` bash
brew tap railwaycat/emacsmacport
brew install emacs-mac 
brew install emacs-mac --with-natural-title-bar # 状态栏透明（Emacs 29版本不需要）

# disable tap
brew untap railwaycat/emacsmacport
```

透明设置
``` bash
#透明效果默认不开启，若要开启：
defaults write org.gnu.Emacs TransparentTitleBar DARK
#或
defaults write org.gnu.Emacs TransparentTitleBar LIGHT

#若要禁用透明效果：
defaults write org.gnu.Emacs TransparentTitleBar NO

#若要隐藏标题栏上的文档图标：
defaults write org.gnu.Emacs HideDocumentIcon YES
```

参考链接
1. [Emacs Port Homebrew 安装](https://github.com/railwaycat/homebrew-emacsmacport)
2. [状态栏透明](https://emacs-china.org/t/topic/2596)
