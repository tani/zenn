---
title: "Quickfix is all you need"
emoji: "🚀"
type: "tech"
topics: ["vim", "neovim"]
published: false
---

# Quickfix is all you need

In July 2023, the author of null-ls.nvim, jose-elias-alvarez,
submitted a critical issue to the repository titled "IMPORTANT: Archiving null-ls".
Within this thread, he asserted his intent to archive the repository,
officially halting further maintenance and development of it.
In effect, this meant the repository would be archived.

null-ls.nvim is a versatile plugin that empowers users to leverage external
programs for code actions, diagnostics, and formatting. Essentially,
it forms an integrated bridge connecting Neovim's built-in LSP client
and these external programs.

Being a user of null-ls.nvim myself, primarily for displaying diagnostics
for non-LSP languages, this news prompted my decision to create a replacement
for the null-ls.nvim tool. Earlier projects related to this endeavor include
but are not limited to, ALE, syntastic, and nvim-lint.
Though these plugins are indeed valuable. However, I just simply want to display
diagnostics for non-LSP languages on-the-fly. Nothing more, nothing less.

Presently, my coding work involves the use of Zig with Vim.
Conveniently, zig.vim offers a LSP-free diagnostic feature for Zig.
Fascinated by this, I am motivated to replicate this functionality for other languages.

Delving deeper, zig.vim owes its efficiency to Vim's quickfix feature.
The workflow is structured below:

- Run external program defind as `'makeprg'` option in Vim.
- Parse the output of the external program and convert it to a quickfix list.
- Display the quickfix list by `:copen` command.

This is a very simple and easy-to-understand flow.
We just to need the parsing logic for the output of the external program.
No plugin is required because it uses the built-in quickfix feature of Vim.

The following is an example of the output of the external program.

```
/path/to/file:line:col: error: message
/path/to/file:line:col: warning: message
```

Then, we can parse the output as follows:

```viml
setlocal errorformat=%f:%l:%c: %trror: %m,%f:%l:%c: %tarning: %m
```

The `'errorformat'` option is used to parse the output of the external program.
`%f` is the file name, `%l` is the line number, `%c` is the column number,
`%t` is the type of the error, which is either `e` or `w`, and `%m` is the
error message.

Then, we set the `'makeprg'` option to the external program and run the `:make`.
The output of the external program is parsed and converted to a quickfix list.
Finally, we display the quickfix list by `:copen` command.

The following is the example settings to check on-the-fly for POD.
This configuration triggers the 'make' command when the file is saved.

```lua
local function run_make()
  vim.cmd[[silent make]]
  if #(vim.fn.getqflist()) > 0 then
	vim.cmd.copen()
  else
	vim.cmd.cclose()
  end
end

vim.api.nvim_create_autocmd("FileType", {
  pattern = "pod",
  callback = function()
	vim.opt_local.makeprg = "podchecker %"
	vim.opt_local.errorformat = {
	  "*** %tRROR: %m at line %l in file %f",
	  "*** %tARNING: %m at line %l in file %f",
	  "%-G%.%#",
	}
	vim.api.nvim_create_autocmd("BufWritePost", {
	  group = "Vimrc",
	  buffer = 0,
	  callback = run_make,
	})
  end
})
```

This is a very simple and easy-to-understand flow.
This is stable and has been used for a long time.
I think this is the best way to display diagnostics for non-LSP languages.
Shall we go back to the basics? Quickfix is all you need.

Cheers.