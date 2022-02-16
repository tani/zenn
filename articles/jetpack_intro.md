---
title: "とても速いVimプラグインマネージャvim-jetpack"
emoji: "🚀"
type: "tech"
topics: ["vim", "neovim"]
published: true
---

https://github.com/tani/vim-jetpack

vim-jetpackはvim-plugの実装をvim8/neovim向けにモダン化させdein.vimで用いられている最適化手法を取り入れた，Packer.nvim風のコマンド郡を備えたとても高速なプラグインマネージャです。どのくらい高速なのかといえばプラグインマネージャを用いずにビルトインのプラグイン管理機能を使ったVimよりも高速です。つまり、プラグインマネージャを読み込むオーバーヘッドよりもプラグインマネージャが行う最適化によって削減される時間のほうが大きいということです。

Vimのプラグインマネージャを使っている人はもちろん、Vimのビルトイン機能でプラグインを管理しているミニマリストにも自信をもってお勧めできます。

# 既存のプラグインマネージャの中でも安定して高速である

速さは正義です。起動時間が速ければ速いほどユーザー体験は向上します。Vimの起動は多くのテキストエディタの中でもかなり速い部類ですが、それでも頻繁にVimの起動と終了を繰り返していると僅かな遅延が気になってきます。vim-jetpackは最速です。遅延読み込みを設定しなくても十分に高速です。なぜならvim-jetpackではdein.vimのようにプラグインインストール時に事前にプラグインのディレクトリ構造を最適化して起動時に読み込むディレクトリの数を減らし起動時間を短縮するからです。

packer.nvim, dein.vim, vim-jetpack, minpac, packer.nvim, paq.nvim,
vim-plugの6つのプラグインマネージャに対して、適当なvimrcを設定して10回起動したときの統計結果が以下の表とグラフです。単位はミリ秒です。

![](/images/jetpack_benchmark.png)

|          |  dein | jetpack | minpac | packer |   paq |  plug |
| :------: | ----: | ------: | -----: | -----: | ----: | ----: |
| 最小値   | 80.61 |   69.93 |  64.97 |  75.38 | 73.92 | 77.63 |
| 最大値   | 96.02 |   74.48 |  81.30 |  89.40 | 84.95 | 82.82 |
| 中央値   | 85.26 |   71.92 |  72.38 |  78.38 | 78.16 | 80.36 |
| 平均値   | 86.24 |   71.97 |  72.48 |  80.07 | 78.21 | 80.12 |
| 分散     | 27.09 |    2.07 |  23.99 |  24.56 | 10.83 |  3.57 |

minpacとjetpakが他の4つよりも明らかに高速に起動していることがグラフからすぐに分りますね。そこでjetpackとminpacについて表を用いて詳しく観察してみましょう。

minpacは最小値こそjetpackよりも小さく高速に起動した記録がありますが、 最大値も大きく上にブレています。minpacの最大値はpacker,paq, vim-plugの3つの どの平均起動よりも遅くなっており最大値と最小値の幅が大きく、その分散はdeinに継いで2番目に大きい値です。したがって起動が速くなることもありますが、そのは起動速度は安定しておらず 常に高速に起動しているとは言い難いです。

jetpackは最小値こそminpacには劣っていますが、中央値、平均値ともにどのプラグインマネージャよりも小さい値を記録しています。さらに特筆すべきはjetpackの記録の分散がとても小さいことです。vim-plugの分散も十分に小さいですが、jetpackの分散は圧倒的に小さいです。**jetpackは分散、平均値、中央値が6つのプラグインマネージャのなかで最小であることから、 安定して常に高速に起動していることが読みとれます。**

なお使用したベンチマークのスクリプトと結果は https://github.com/tani/vim-jetpack/tree/main/benchmark にて公開しています。

# グラフィカルなインストールの進捗状況

![](/images/jetpack_progress.gif)

プラグインマネージャが高速であると評されるとき、それは単に実行速度だけ表わすものではありません。ユーザーにストレスを与えず体感速度が高速であることが重要です。その点、最近の軽量プラグインマネージャは僕にとって不満でした。パッケージをインストールするコマンドを叩いたあと標示されるのは、ぶっきらぼうなメッセージのみで、いつインストールが完了するのかが予測しづらく、 実際の実行速度にかかわらず体感速度として遅いと感じていました。

vim-jetpackは、この点をvim-plug風のプログレスバー表示を備えることで解決しました。これによりユーザーは現在の処理がどのくらい進行していていつ頃完了するのかを予測できるようにして、 体感速度を向上させました。

# Vim-plugとの互換性が高い

機能を絞れば大抵のソフトウェアは高速化を達成することができます。しかしvim-jetpackは機能を絞ることはせずに、vim-plugと同等のオプションを提供する努力をしています。ほとんどのvim-plugユーザーは`:s/plug/jetpack/g | s/Plug/Jetpack/g`を実行するだけでvim-jetpackへの移行が完了します。単純な文字列置換だけで高速化されるなら、移行しない手はないですね！

|      name       |        type        | description                                                   |
| :-------------: | :----------------: | :------------------------------------------------------------ |
| `branch`/ `tag` |      `sring`       | Branch/ tag of the repository to use                          |
|      `rtp`      |      `string`      | Subdirectory that contains Vim plugin                         |
|      `dir`      |      `string`      | Custom directory for the plugin                               |
|      `as`       |      `string`      | Use different name for plugin                                 |
|      `do`       | `string` or `func` | Post-updat hook                                               |
|      `on`       | `string` or `list` | On-demand loading: Commands, `<Plug>`, and autocommand-events |
|      `for`      | `string` or `list` | On-demand loading: File types                                 |
|      `opt`      |     `boolean`      | On-demand loading: `packadd {name}`                           |
|    `frozen`     |     `boolean`      | Do not update                                                 |

# 複数のDSLが使える

Vimを使う人はこだわりが強い人が多いでしょう。設定ファイルの記述もvim-plugのようなDSLが好きな人も入ればdein.vimのようにVimscriptの関数による最小のDSLが好きな人、はたまたpacker.nvimのようにLuaによる設定やpaq.nvimのような設定が好きな人もいます。vim-jetpackは、これら全ての書き方をサポートしています。

## vim-plug style

```vim
call jetpack#begin()
Jetpack 'junegunn/fzf.vim'
Jetpack 'junegunn/fzf', { 'do': {-> fzf#install()} }
Jetpack 'neoclide/coc.nvim', { 'branch': 'release' }
Jetpack 'neoclide/coc.nvim', { 'branch': 'master', 'do': 'yarn install --frozen-lockfile' }
Jetpack 'vlime/vlime', { 'rtp': 'vim' }
Jetpack 'dracula/vim', { 'as': 'dracula' }
Jetpack 'tpope/vim-fireplace', { 'for': 'clojure' }
call jetpack#end()
```

## dein/ minpac style

```vim
call jetpack#begin()
call jetpack#add('junegunn/fzf.vim')
call jetpack#add('junegunn/fzf', { 'do': {-> fzf#install()} })
call jetpack#add('neoclide/coc.nvim', { 'branch': 'release' })
call jetpack#add('neoclide/coc.nvim', { 'branch': 'master', 'do': 'yarn install --frozen-lockfile' })
call jetpack#add('vlime/vlime', { 'rtp': 'vim' })
call jetpack#add('dracula/vim', { 'as': 'dracula' })
call jetpack#add('tpope/vim-fireplace', { 'for': 'clojure' })
call jetpack#end()
```

## packer style

```lua
require('jetpack').startup(function(use)
  use 'junegunn/fzf.vim'
  use {'junegunn/fzf', do = 'call fzf#install()' }
  use {'neoclide/coc.nvim', branch = 'release'}
  use {'neoclide/coc.nvim', branch = 'master', do = 'yarn install --frozen-lockfile'}
  use {'vlime/vlime', rtp = 'vim' }
  use {'dracula/vim', as = 'dracula' }
  use {'tpope/vim-fireplace', for = 'clojure' }
end)
```

## paq style

```lua
require('jetpack').setup {
  'junegunn/fzf.vim',
  {'junegunn/fzf', do = 'call fzf#install()' },
  {'neoclide/coc.nvim', branch = 'release'},
  {'neoclide/coc.nvim', branch = 'master', do = 'yarn install --frozen-lockfile'},
  {'vlime/vlime', rtp = 'vim' },
  {'dracula/vim', as = 'dracula' },
  {'tpope/vim-fireplace', for = 'clojure' },
}
```

# 軽量でインストールが簡単

vim-plugの素晴しいところはパッケージマネージャ自体が1つのファイルだけで構成されていることです。これにより、HTTPリクエストが投げれる環境ならどこでもvim-plugを設定することができます。vim-jetpackもこれに倣い1つのファイルに収めています。サイズも小さいためVimの設定ファイルの一つとしてリポジトリに組み込むことも可能です。
余談ですがVimscriptは古典的な逐次インタプリタ処理をしているのでコードが小さいことで単純に速度向上にもつながっています。

## Vim

```
curl -fLo ~/.vim/autoload/jetpack.vim --create-dirs https://raw.githubusercontent.com/tani/vim-jetpack/master/autoload/jetpack.vim
```

## Neovim

```
curl -fLo ~/.config/nvim/autoload/jetpack.vim --create-dirs https://raw.githubusercontent.com/tani/vim-jetpack/master/autoload/jetpack.vim
```

またPlugと同様に1つのファイルで構成されているので，自動でインストールされるようにブートストラップ処理を書くことも可能です．

```vim
let data_dir = has('nvim') ? stdpath('data') . '/site' : '~/.vim'
if empty(glob(data_dir . '/autoload/jetpack.vim'))
  silent execute '!curl -fLo '.data_dir.'/autoload/jetpack.vim --create-dirs  https://raw.githubusercontent.com/tani/vim-jetpack/master/autoload/jetpack.vim'
  autocmd VimEnter * JetpackSync | source $MYVIMRC
endif
```

# 設定や操作が簡単

プラグインを使うだけならば、プラグインのインストールと更新は同時に行われても問題無いことが多いです。そこでvim-jetpackでは単一のコマンド`JetpackSync`のみでインストールと更新、最適化までを一括して行うようにしています。ユーザーは設定ファイルを更新した際に `JetpackSync`(あるいは環境によっては `Jetpack` のみ) で最新の状態を維持できます。

またvim-jetpackによる最適化レベルの設定は3つあり`g:jetpack#optimization`で調整することができます。`let g:jetpack#optimization=0`では一切の最適化を無効にします。これはvim-plugを同じ挙動です。`let g:jetpack#optimization=1`では安全な場合のみ最適化を行います。これがvim-jetpackの既定の挙動です。`let g:jetpack#optimization=2`では全てのプラグインで最適化を行います。これはdein.vimの挙動に近いです。

プラグイン同士の干渉を調べるにはそれなりに処理時間が必要になるので、おすすめは`let g:jetpack#optimization=2`を試してみて設定が壊れてしまうようなら最適化レベルを下げることです。

# dein.vimよりも高速なのか

もし、あなたがVimscriptについて造詣が深くvimrcを注意深く設定することができれば、dein.vimは依然として最速です。

どのタイミングでVimのイベントが発火されるのか、自動読み込みが必要となるのか、プラグイン同士の依存関係と読み込み順序が維持できているのか、これらを把握することができればより高度な遅延処理によって起動時に読み込むプラグインを極限まで遅らせることができます。

一方で高度なカスタマイズ機能を提供するためにdein.vimは色々な処理が起動時に実行されています。もし、ユーザーが単にVimのプラグインを記述して読み込みたいだけなら、このときに実行されていることの大部分は無駄な実行時間です。

vim-jetpackはカスタマイズできる機能をvim-plug相当まで減らすことでその無駄な処理を大幅に削っています。これによりdein.vimよりも高速な起動を実現しています。

# まとめ

vim-jetpackは軽量最速のプラグインマネージャです。大体のケースでは移行するだけで起動時間が大幅に短縮されます。vim-plugのファンシーなインターフェースを気に入っているのではない限り移行しない手はないでしょう。**vim-jetpackはあなたのvimを加速させます。**

今後は、vim-plugとの互換性の向上と不正な記述に対するエラー処理を充実させていく予定です。
