---
title: "Neovim と Vlime と Mondo (令和3年冬最新版)"
emoji: "🎄"
type: "tech"
topics: ["vim", "neovim", "lisp"]
published: true
---

普段のCommon Lispの開発にはEmacs/[SLIME](https://github.com/slime/slime) (or [SLY](https://github.com/joaotavora/sly))をつかって開発しています。
そして、この記事のように設定してNeovimでCommon Lispを開発できるようにしても、
自分の中での至高の開発環境はEmacs/SLIMEでした。次点では [cxxxr/lem](https://github.com/cxxxr/lem) です。

さて普段のソフトウェア開発ではNeovimをつかっていますがLispのソースコードを
書くときだけはEmacsを立ち上げていました。たとえEmacsのキーバインドをEvilで揃えてつかっていても二つのエディタの設定をメンテナンスするのは中々大変です。
そこで本稿ではNeovimだけで開発が閉じることができるようにNeovimでSLIME相当の開発環境を設定する方法を示します。

## 使用したもの

- [neovim/neovim v0.6.0](https://github.com/neovim/neovim) - Vimからforkしたテキストエディタ
  - たぶんVim 8でも動くはず
- [roswell/roswell](https://github.com/roswell/roswell) - LISP開発環境を管理するCLI
- [fukamachi/mondo](https://github.com/fukamachi/mondo) - fukamachi氏が開発しているSWANKプロトコルを仲介するREPL環境
  - GNU readline が必要です。使用環境に応じてインストールしておく必要があります。
- [fukamachi/vlime](https://github.com/fukamachi/vlime) - vlimeのfukamachi氏によるfork
  - **注意**: [vlime/vlime](https://github.com/vlime/vlime) や 他のForkでは動きませんでした。

## mondo

mondoはroswellを使ってインストールする。
roswellはコマンドラインツールを`$HOME/.roswell/bin`配下にインストールするので
そこに予めPATHを通しておく。

```sh
$ export PATH+=:$HOME/.roswell/bin
$ ros install fukamachi/mondo
```

mondoはSWANKプロトコルをラッピングしたREPLを提供しているので、
単に`mondo`とコマンドを呼び出すだけでSLIME相当のREPLが立ちあがる。

```sh
$ mondo
CL-USER> (+ 1 1)
1
```

## vlime

vlimeの設定は以下のとおりである。

```vim
"Fukamachi's config: https://github.com/fukamachi/neovim-config/blob/7a8f609eae96f18e2775f0c945a1e554aa6e44ce/conf/20-common-lisp.vim
plug#begin('~/.vim/plugged')
Plug 'fukamachi/vlime', { 'rtp': 'vim/' }
plug#end()

nnoremap <silent> <LocalLeader>rr :call VlimeStart()<CR>

let g:vlime_cl_impl = "mondo"
let g:vlime_cl_use_terminal = v:true
let g:vlime_window_settings = {
    \ 'server': {'pos': 'botright', 'size': v:null, 'vertical': v:true}
    \ }

function! VlimeBuildServerCommandFor_mondo(vlime_loader, vlime_eval)
    return ["mondo", "--server", "vlime"]
endfunction

function! VlimeStart()
    call vlime#server#New(v:true, get(g:, "vlime_cl_use_terminal", v:false))
endfunction
```

主な開発フローは以下のようになる。

- Step 1. Mondoに接続する (`<LocalLeader>rr`)
- Step 2a. 現在開いているファイルを読み込む (`<LocalLeader>l`)
- Step 2b. カーソルにあるS式をディスアセンブルする (`<LocalLeader>a`)
- Step 2c. カーソルにあるS式をブレークポイントに設定する (`<LocalLeader>b`)
- Step 2d. カーソルにあるS式を送信する (`<LocalLeader>ss`)
- Step 2e. カーソルにあるS式のマクロを展開する (`<LocalLeader>mm`)
- Step 2f. カースルにあるS式をトレース対象にする (`<LocalLeader>TT`)
- Step 3. Mondoから切断する (`<LocalLeader>cd`)
