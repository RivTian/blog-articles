![](https://wusyong.github.io/static/nvchad.png)

Vim 一直是我最常用到的編輯器，因爲在熟悉 Key Mapping 後，整體的寫作非常的流暢，這一直是我覺得主流 GUI IDE 無法比擬的，而且幾乎所以系統預設都會有 Vim（或至少有 Vi），很容易就可以開起來馬上進行編輯。但 Vim 最大的缺點我覺得大概就是設定檔得「Wrtie once, configure everytime」。就算有備份或 repo 儲存，每次想在新的環境調整好自己熟悉的完整體驗的話，多少都還是得花些時間。而且有些插件時間一久可能就不適用或者更新時壞掉了，常常得再把腳本重新寫好。就算我已經轉到 NeoVim 了，很多我常用的 LSP 插件也三不五時會秀逗，有時甚至懶了就當某些功能像 code action 就不存在了。

[NvChad](https://github.com/NvChad/NvChad) 就是希望能提供 Vim 開箱即用的體驗，我們不必自己寫一大堆的 Vim script 或 lua。NvChad 會給予最合理的預設設置，讓 Vim 馬上就有像 IDE 一樣有各種開發者常常需要的功能。下面我們就來看怎麼安裝，還有常用到的快捷鍵會有哪些。

## 前置準備

-   [Neovim 0.5](https://neovim.io/) 以上（因爲 0.5 開始才有內建的 LSP）
-   終端機使用的字體請用 [Nerd Font](https://www.nerdfonts.com/font-downloads)。我自己是因爲本身就有在用 [Starship](https://starship.rs/) shell prompt，所以就已經是 FiraCode Nerd Font 了。
-   [Material Design Icon Font](https://materialdesignicons.com/)：下載然後安裝到系統的字體目錄。像 Linux 的話就是 `/usr/share/fonts/TTF`。
-   `git`

其他可能需要安裝到還有：

-   `node`：許多 LSP 插件會使用 node，當然你不喜歡的話也可以選擇沒有使用 Node.js 的插件或是乾脆自己寫 Lua。
-   `ripgrep`：Telescope 插件在 grep search 時需要的工具，而且 ripgrep 本身超快，這也是我現在常用的搜尋工具了。

## 安裝流程

安裝 NvChad 其實滿簡單的，因爲基本上就是把設定檔放到 `~/.config/nvim` 目錄底下而已。如果你已經有些設定的話，可以先備份起來：

```console
$ mv ~/.config/nvim ~/.config/nvim.backup
```

接著就可以下載 NvChad 的 repository 到 `~/.config/nvim` 了。

### Linux/Mac

```console
$ git clone https://github.com/NvChad/NvChad ~/.config/nvim
$ nvim +'hi NormalFloat guibg=#1e222a' +PackerSync
```

### Windows

Windows 會比較麻煩一些，因爲得安裝 [`mingw`](http://mingw-w64.org/doku.php)。你可以自己下載安裝，或是用 chocolatey：

```console
choco install mingw
```

裝好之後再來安裝 NvChad：

```console
$ git clone https://github.com/wbthomason/packer.nvim "$env:LOCALAPPDATA\nvim-data\site\pack\packer\start\packer.nvim"
$ git clone https://github.com/siduck76/NvChad
```

然後到 `~/AppData/Local/nvim` 將 NvChad 的 `init.lua` 和 `lua` 資料夾移到 `nvim` 資料夾底下。最後用以下命令進行初始化，遇到錯誤就直接按 Enter 讓 neovim 開始安裝插件：

```console
$ nvim +PackerSync
```

## 更新

因爲整個 NvChad 基本上就是個 git project，所以是可以 pull 最新的內容來更新的，在 NvChad 的設定中你可以按 `<leader> + uu` 來更新。注意 `<leader>` 預設會被 NvChad 改成空白鍵，而且這會執行 `git reset --hard`，所以記得做好備份。

## 解除安裝

如果想要解除安裝的話也很簡單，刪除掉以下目錄就好：

```console
rm -rf ~/.config/cache/nvim
rm -rf ~/.config/nvim
rm -rf ~/.local/share/nvim
```

## 安裝 LSP Server

Nvim 有內建的 LSP Client，但你還是得要安裝你所使用的語言的 LSP Server 才有辦法使用。NvChad 使用[`nvim-lspconfig`](https://github.com/neovim/nvim-lspconfig) 插件，要是以前就用這個插件的話，使用 LSP 的方式與快捷鍵應該就一模一樣。要安裝各語言的 LSP Server 的話，NvChad 是用 [nvim-lspinstall](https://github.com/kabouzeid/nvim-lspinstall)，你可以查這個連結看有哪些支援的語言。舉例來說，你要安裝 rust 的話，就是：

```console
:LspInstall rust
```

## 安裝 Treesitter

[`nvim-treesitter`](https://github.com/nvim-treesitter/nvim-treesitter) 能提供各種語言的語法凸顯，一樣你可以搜尋這個連結看看這個插件支援的語言有哪些。要安裝的話，就是執行這個指令：

```console
:TSInstall rust
```

## 其他設定

如果想要自己自訂設置或是增減插件的話，可以看看 NvChad 的 [Config 頁面](https://nvchad.netlify.app/config)，這邊有陳列出所以腳本大致上的對應位置。舉例來說，你想要自訂 key map 的話，就是編輯 `/lua/chadrc.lua`。

## 快捷鍵（Key Mapping）

如同上面講的，要自訂 mapping 的話就是編輯 `/lua/chadrc.lua`。需要查看各個快捷鍵和 mapping 的話可以透過 `cheatsheet` 插件搜尋。`<leader> + dk` 會顯示預設的 mapping，而 `<leader> + uk` 會顯示自訂的。以下是一些常見會用到的 mapping，我也順便列出其他像是 LSP 插件會用到的快捷鍵：

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20230524172420.png)