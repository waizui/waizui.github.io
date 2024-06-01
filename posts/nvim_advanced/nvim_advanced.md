# Neovim 进阶篇

## 2024-6-1


在[基础篇](https://waizui.github.io/posts/nvim_basic/nvim_basic.html)和[中级篇](https://waizui.github.io/posts/nvim_intermediate/nvim_intermediate.html)里说了如何把neovim配置到进行代码补全。
这篇说一下如何配置neovim的代码调试相关的功能，把它配置成一个IDE。


## 安装LSP服务

### None-ls
想要格式化，代码检查一类的功能的话，先要配置lsp服务，不过在这之前，
推荐要安装[None-ls](https://github.com/nvimtools/none-ls.nvim?tab=readme-ov-file#null-lsnvim)这个插件，
None-ls可以看作是lsp与neovim沟通的中间层，它提供lua实现的lsp相关API。

None-ls提供补全，提示，格式化等功能。在使用之前，需要在setup里面注册sources，
像下面的代码一样。

```lua
return {
   "nvimtools/none-ls.nvim",
   config = function()
     local null_ls = require("null-ls")
     null_ls.setup({
       sources = {
         null_ls.builtins.formatting.stylua,
         null_ls.builtins.completion.spell,
      },
    })
   end,
}
```

### Mason

Mason是一个lsp的管理器，可以用来安装各种语言的lsp服务，非常方便。

## 如何Debug


### Dap

