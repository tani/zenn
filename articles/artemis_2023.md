---
title: "Artemis: Bridging the gap between Vim and Neovim"
emoji: "🌔"
type: "tech"
topics: ["vim", "neovim"]
published: true
---

# Artemis: Bridging the gap between Vim and Neovim

[vim-artemis](https://github.com/tani/vim-artemis)
is a lua module offering a unified interface for
interaction with both Vim and Neovim, geared for use in the configuration file.

In recent years, Neovim has emerged as a notable modern alternative to
Vim, boasting numerous new features and enhancements over Vim.
A key attribute is the ability to compose a configuration in Lua.
Keep in mind that the Lua API is not an innovation. It has already existed in Vim.
Nevertheless, it isn't compatible with Neovim's Lua API. More precisely, Neovim's
Lua API isn't a superset of Vim's Lua API.

Although I prefer Neovim as my primary editor, I also want to extend
support for Vim configuration. To avoid the hassle of maintaining two distinct configuration files,
my aim is to write a singular configuration file compatible with both Vim and Neovim.

This is where `vim-artemis` steps in. It establishes a common interaction realm for
both Vim and Neovim. I developed wrapper functions grounded on
Neovim's Lua API, permitting the usage of a Neovim-like API in Vim.

Initially, `vim-artemis` needs to be loaded into your configuration file.

```lua
local vimx = require("artemis")
```

Once done, `vimx` can be used to interact with Vim and Neovim.

```lua
vimx.cmd("colorscheme gruvbox")
```

Prefer to use `vimx.cmd.colorscheme` instead of the code above? -- No issues.

```lua
vimx.cmd.colorscheme("gruvbox")
```

Moreover, all means of calling `vim.cmd` can be utilized.
We can employ `keymap`, `fn`, `opt` among others. For instance,

```lua
vimx.opt_local.number = true

vimx.keymap.set("n", "<leader>q", function()
  local buf = vimx.fn.bufnr("%")
  vimx.cmd.echo("'Closing " .. vimx.fn.bufname(buf) .. "'")
  vimx.cmd.bdelete(buf)
end)
```

Next, I'd like to highlight a deeper issue between Vim and Neovim.
It's not just the interface compatibility, rather the semantic compatibility also.
Neovim converts a Lua object to a VimL object implicitly.
Vim, however, requires an explicit conversion between Lua and VimL.
This represents a balance between usability and safety.
For instance, `{ 1 = 'a' }` is a lua table. Whether it's a dictionary or a list
depends on context. Neovim converts it to a VimL list implicitly.
We adopt a Neovim-like approach, converting a Lua object to a VimL object implicitly.
Check the source code of the `cast()` function for implementation details.

Even if Vim isn't your preference, `vim-artemis` has utility for Neovim users.
It offers a superior `fn` interface compared to Neovim's Lua API.
In VimL, `namespace#function_name` is used to call a function defined in an autoload script.
In contrast, Neovim uses `vim.fn['namespace#function_name']`, which is rather verbose.
`vim-artemis` provides an improved interface.

```lua
vimx.fn['namespace#function_name'] -- Neovim's Lua API
vimx.fn.namespace.function_name     -- vim-artemis
```

This interface is supported through the use of the `__index` metamethod.
It also facilitates chaining of deeper namespaces, such as `namespace.subnamespace.function_name`.

```lua
vimx.fn.namespace.subnamespace.function_name
```

To conclude, `vim-artemis` establishes a common interface for integrating with Vim and Neovim.
It is intended for use in the cofiguration file and is beneficial for Neovim users as well.

Cheers!
