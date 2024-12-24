#brew 

参考笔记：[https://pjchender.dev](https://pjchender.dev)

```sh
# 安裝套件
brew search <formula>        # 尋找有無某套件
brew install <formula>         # 安裝套件
brew uninstall <formula>       # 移除套件

# update local packages
brew outdated                      # find out what is outdated
brew update                        # 更新 brew 本身和 formula
brew upgrade                       # 更新 brew 內的所有套件
brew upgrade <formula>             # 更新 brew 內的特定套件

# 移除舊版本（Homebrew 預設不會自動移除舊版本）
brew cleanup <formula>            # remove everything or specific formula of old versions
brew cleanup -n                   # see what would be cleaned up

# brew tap：適用於安裝不在 homebrew 的第三方套件（會增加 homebrew 的 formulae）
brew tap                      # list tapped repositories
brew tap <tap-name>           # add tap
brew untap <tap-name>         # remove a tap

# services
brew tap homebrew/services
brew services
brew services list                          # 看有哪些服務
brew services start [service_name]          # 開啟某個服務
```


## Homebrew 安裝與使用

[Homebrew](https://brew.sh/index_zh-tw.html) 是 Mac OSX 上的的套件管理工具，是方便安裝管理 OSX 裡需要用到但預設沒安裝的套件，其內容多是**Command Line Software**，這些 CMD 軟體通常是 Open Source 的，可於 [Formulas](https://github.com/Homebrew/homebrew/tree/master/Library/Formula)中檢視可安裝的清單。

Homebrew 的安裝只需要打開終端機，輸入：

```sh
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

就可以了。接著可以輸入以下指令確認是否有安裝成功：

```sh
brew --version
```

## Homebrew-cask 安裝與使用

brew cask 是 brew 的其中一個套件，它主要是可以透過 homebrew 的 API 來安裝許多 **MAC 的 GUI 軟體**，這些軟體可能是免費或需付費的：

```
# 安裝$ brew install --cask google-chrome$ brew install --cask google-chrome firefox   # 一次安裝多個 app$ brew cask list          # 列出所有透過 cask 安裝的 App# 移除$ brew cask uninstall google-chrome# 搜尋$ brew cask search <軟體名>     # 顯示某 APP 能否透過 cask 安裝$ brew cask info google-chrome      # 顯示關於某 APP 的詳細資料# 更新軟體$ brew cask outdated          # 列出所有並非最新版的軟體$ brew cask upgrade         # 更新所有軟體
```

> [homebrew-cask usage](https://github.com/caskroom/homebrew-cask/blob/master/USAGE.md) @ Github

## 透過 Homebrew-services 啟動資料庫等服務

```sh
brew tap homebrew/services
brew install postgresql
brew services start postgresql
```

> [Homebrew-Services](https://github.com/Homebrew/homebrew-services) @ Github [brew 和 brew cask 指令的差別](https://apple.stackexchange.com/questions/125468/what-is-the-difference-between-brew-and-brew-cask?newreg=135aa4d32b8a44edaa934ccde78cfb31) @ StackExchange

## 透過 home-brew-cast-fonts 安裝字型

所有可透過 homebrew 安裝的字體可以參考[這裡](https://github.com/Homebrew/homebrew-cask-fonts/tree/master/Casks)：

```sh
# you only have to do this once!
brew tap homebrew/cask-fonts                  
brew install --cask font-inconsolata
```

> [brew-cask-fonts](https://github.com/Homebrew/homebrew-cask-fonts) @ homebrew