---
title: 在VSCode和cursor里增加点击方法名跳转到对应为止
author: 独孤流
date: 2025-05-18 10:14:00 +0800
categories: [rust]
tags: [iOS]     # TAG names should always be lowercase
---


```
# 检查环境，如果没有安装则进行安装
rust-analyzer --version
rustup which rust-analyzer
rustup install stable
rustup component add rust-analyzer
# 安装环境：
rustup component add rust-analyzer
```

打开vscode配置
打开 `settings.json`：
快捷键：`Cmd + Shift + P` / `Ctrl + Shift + P`

输入并选择：`Preferences: Open User Settings (JSON)`
在配置里增加如下配置
```
{
  "rust-analyzer.checkOnSave.command": "clippy",        // 保存时检查代码质量（可选）
  "rust-analyzer.inlayHints.enable": true,               // 类型提示
  "rust-analyzer.cargo.loadOutDirsFromCheck": true,
  "rust-analyzer.procMacro.enable": true,                // 支持宏生成的代码跳转
  "editor.inlineSuggest.enabled": true,                  // inline 补全（AI 辅助体验）
  "editor.hover.enabled": true,
  "editor.definitionLinkOpensInPeek": false              // 跳转时是否用 peek（改为 false 是直接跳转）
}
```

完整配置如下：
```
{
    "security.workspace.trust.untrustedFiles": "open",
    "debug.onTaskErrors": "showErrors",
    "explorer.confirmDelete": false,
    "rust-analyzer.checkOnSave.command": "clippy",
    "rust-analyzer.inlayHints.enable": true,
    "rust-analyzer.procMacro.enable": true,
    "rust-analyzer.cargo.loadOutDirsFromCheck": true,
    "editor.definitionLinkOpensInPeek": false,
    "explorer.confirmDragAndDrop": false
}
```
![image](/assets/img/rust/rust_analyzer.png)

----

进入方法实现：
`cmd + 鼠标点击`

进入实现后想要返回之前的位置
`ctl + -` : 不断返回上一步的操作

修改快捷键操作习惯
按下：`Cmd + Shift + P`\
输入：`Preferences: Open Keyboard Shortcuts`\
搜索：`Go Back`, 选择并双击，然后修改自己喜欢的快捷键，之后回车即可
也可以修改前进： `Go Forward`来修改为自己习惯喜欢的
