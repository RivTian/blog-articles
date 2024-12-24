## keybinds.json

```json
// Place your key bindings in this file to override the defaultsauto[]
[
  // 以前配置的上下左右移动按键
  {
    "key": "alt+j",
    "command": "cursorLeft",
    "when": "textInputFocus"
  },
  {
    "key": "alt+k",
    "command": "cursorDown",
    "when": "textInputFocus"
  },
  {
    "key": "alt+l",
    "command": "cursorRight",
    "when": "textInputFocus"
  },
  {
    "key": "alt+i",
    "command": "cursorUp",
    "when": "textInputFocus"
  },
  // 切换到文件浏览器,可以在任何位置
  {
    "key": "ctrl+;",
    "command": "workbench.view.explorer",
    "when": "viewContainer.workbench.view.explorer.enabled"
  },
  // 切换到代码编辑区,不论在任何位置
  {
    "key": "ctrl+'",
    "command": "workbench.action.focusFirstEditorGroup"
  },
  // 切换到terminal终端
  {
    "key": "ctrl+,",
    "command": "workbench.action.terminal.toggleTerminal",
    "when": "terminal.active"
  },
  // 打开一个新的terminal
  {
    "key": "ctrl+shift+,",
    "command": "workbench.action.terminal.new",
    "when": "terminalProcessSupported || terminalWebExtensionContributedProfile"
  },
  // 在文件夹资源管理器中新建一个文件
  {
    "key": "a",
    "command": "explorer.newFile",
    "when": "filesExplorerFocus && !inputFocus"
  },
  // 在文件资源管理器里面创建一个文件夹
  {
    "key": "shift+a",
    "command": "explorer.newFolder",
    "when": "filesExplorerFocus && !inputFocus"
  },
  // 在文件资源管理器里面重应名当前文件或文件夹
  {
    "key": "r",
    "command": "renameFile",
    "when": "explorerViewletVisible && filesExplorerFocus && !explorerResourceIsRoot && !explorerResourceReadonly && !inputFocus"
  },
  // 在文件资源管理器中删除文件
  {
    "key": "d",
    "command": "deleteFile",
    "when": "explorerViewletVisible && filesExplorerFocus && !explorerResourceReadonly && !inputFocus"
  },
  // 复制当前文件
  {
    "key": "y",
    "command": "filesExplorer.copy",
    "when": "explorerViewletVisible && filesExplorerFocus && !explorerResourceIsRoot && !inputFocus"
  },
  // 黏贴当前文件
  {
    "key": "p",
    "command": "filesExplorer.paste",
    "when": "explorerViewletVisible && filesExplorerFocus && !explorerResourceReadonly && !inputFocus"
  },
  // 放大editor区域
  {
    "key": "ctrl+m",
    "command": "workbench.action.maximizeEditor"
  },
  // 使用code runner运行代码
  {
    "key": "ctrl+r",
    "command": "code-runner.run"
  }
]
```

## settings.json

```json
{
  "[vue]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "editor.formatOnSave": true,
  "editor.lineNumbers": "relative",
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  // -------------------------------------------------
  "vim.leader": "<space>",
  "vim.useSystemClipboard": true,
  "vim.hlsearch": true,
  "vim.highlightedyank.enable": true,
  "vim.foldfix": true,
  "vim.easymotion": true,
  "vim.incsearch": true,
  "vim.useCtrlKeys": true,
  "vim.surround": true,
  "vim.sneak": true,
  "vim.sneakUseIgnorecaseAndSmartcase": true,
  "vim.normalModeKeyBindingsNonRecursive": [
    // Go to start or end of line---------------------------
    {
      "before": ["H"],
      "after": ["^"]
    },
    {
      "before": ["L"],
      "after": ["$"]
    },
    // ----------------------------------------------------
    // Jump to change
    {
      "before": ["[", "c"],
      "commands": ["workbench.action.editor.previousChange"]
    },
    {
      "before": ["]", "c"],
      "commands": ["workbench.action.editor.nextChange"]
    },
    // Code actions
    {
      "before": ["<leader>", "s", "a"],
      "commands": ["editor.action.sourceAction"]
    },
    {
      "before": ["<space>", "r", "r"],
      "commands": [
        // "extension.runScript"
        // "autojspro.run"
        "code-runner.run"
      ]
    },
    // Quick fix
    {
      "before": ["<leader>", "q", "f"],
      "commands": ["editor.action.quickFix"]
    },
    // 重用名变量
    {
      "before": ["<leader>", "r", "n"],
      "commands": ["editor.action.rename"]
    },
    // Format 格式化当前文件
    {
      "before": ["<leader>", "f", "m"],
      "commands": ["editor.action.formatDocument"]
    },
    // go===================================================
    // go to  References
    {
      "before": ["g", "r"],
      "commands": ["editor.action.goToReferences"]
    },
    // ===================================================
    // new===================================================
    // 新建文件夹,在编辑器的区域
    {
      "before": ["<Leader>", "n", "d"],
      "commands": ["explorer.newFolder"]
    },
    // 新建文件,新建文件的位置取决于,文件资源管理器所在的位置
    {
      "before": ["<Leader>", "n", "f"],
      "commands": ["explorer.newFile"]
    },
    // =====================================================
    // open========================================================
    // 打开文件资源管理器,光标会聚焦到文件资源管理器的窗口
    {
      "before": ["<leader>", "o", "e"],
      "commands": ["workbench.view.explorer"]
    },
    // open search bar
    {
      "before": ["<leader>", "o", "s"],
      "commands": ["workbench.action.quickOpen"]
    },
    // 进入到terminal
    {
      "before": ["<leader>", "o", "t"],
      "commands": ["workbench.action.terminal.toggleTerminal"]
    },
    // 新建一个terminal终端
    {
      "before": ["<leader>", "o", "T"],
      "commands": ["workbench.action.terminal.new"]
    },
    // 隐藏和打开terminal
    {
      "before": ["<leader>", "o", "t"],
      "commands": ["workbench.action.togglePanel"]
    },
    // ================================================
    // find char in all files(fing in all files)
    {
      "before": ["<leader>", "f", "a"],
      "commands": ["workbench.action.findInFiles"]
    },
    // ===============================================
    // 移动==============================================================================
    // 左右移动标签页------------------------------------------------
    {
      "before": ["<leader>", "h"],
      "commands": ["workbench.action.navigateLeft"]
    },
    {
      "before": ["<leader>", "l"],
      "commands": ["workbench.action.navigateRight"]
    },
    // 移动下一个编辑器标签
    {
      "before": ["J"],
      "commands": ["workbench.action.nextEditor"]
    },
    // 移动到上一个编辑器标签
    {
      "before": ["K"],
      "commands": ["workbench.action.previousEditor"]
    },
    // 左右移动标签页---------------------------------------------------------------
    {
      "before": ["<space>", "'"],
      "commands": ["workbench.action.maximizeEditor"]
    },
    // 保存当前文件---------------------------------------------------------------
    {
      "before": ["<space>", "s"],
      "commands": ["workbench.action.files.save"]
    },
    {
      "before": ["g", "b"],
      "commands": ["workbench.action.navigateBack"],
      "when": "canNavigateBack"
    },
    // workbench.action.closeActiveEditor
    {
      "before": ["<space>", "w", "c"],
      "commands": ["workbench.action.closeActiveEditor"]
    },
    //workbench.action.closeAllEditors
    {
      "before": ["<space>", "w", "a"],
      "commands": ["workbench.action.closeAllEditors"]
    },
 
    {
      "before": ["<space>", "q", "f"],
      "commands": ["editor.action.quickFix"]
    }
  ],
  "vim.insertModeKeyBindings": [
    // 退出插入模式
    {
      "before": ["j", "k"],
      "after": ["<Esc>"]
    }
  ],
  "vim.visualModeKeyBindingsNonRecursive": [
    // 移动到非空字符的行首
    {
      "before": ["H"],
      "after": ["^"]
    },
    // 移动到非空字符的行尾
    {
      "before": ["L"],
      "after": ["$"]
    }
  ],
  "vim.digraphs": {},
  "vim.commandLineModeKeyBindings": [],
  "vim.commandLineModeKeyBindingsNonRecursive": [],
  "[jsonc]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[html]": {
    "editor.defaultFormatter": "vscode.html-language-features"
  },
  // ====================================================================
  "EnglishChineseDictionary.enableHover": true,
  "security.workspace.trust.untrustedFiles": "open",
  "[json]": {
    "editor.defaultFormatter": "vscode.json-language-features"
  },
  "liveServer.settings.donotShowInfoMsg": true,
  "typescript.updateImportsOnFileMove.enabled": "always",
  "javascript.updateImportsOnFileMove.enabled": "always",
  "workbench.colorTheme": "Monokai",
  "[python]": {
    "editor.defaultFormatter": "ms-python.python"
  },
  // 忽略排除的文件
  "files.exclude": {
    "**/.git": true,
    "**/.svn": true,
    "**/.hg": true,
    "**/CVS": true,
    "**/.DS_Store": true,
    "**/Thumbs.db": true,
    "**/__pycache__": true, // python 生产的cache文件
    "**/.idea": true
  }
}
```