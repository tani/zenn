---
title: "生成AI時代だからこそ、Vim as an IME"
emoji: "😎"
type: "tech"
topics: ["vim", "browser"]
published: true
---

# 生成AI時代だからこそ、Vim as an IME

Firenvim を導入しました。

Firenvim とは、Firefox や Chrome のテキストボックスの上に Neovim をオーバーレイし、テキストエリアにある内容を Neovim の一時バッファに送ってくれる機能です。もちろん、そのバッファを保存（`:w`）すると、ブラウザのテキストエリアに即座に反映してくれます。

今回は、このツールを使って「Vim as an IME」を実現する現代的な意義について語ります。

![](https://raw.githubusercontent.com/glacambre/firenvim/refs/heads/master/firenvim.gif)
(引用: Firenvim GitHub README)

## IME をインストールしないという選択肢

Linux ユーザーにとって、日本語 IME は長年の鬼門の一つです。
日本語 IME の厄介さは、以下の2点に集約されます。

- そもそも平仮名を一つ入力するのに、複数回キーボードを叩かなくてはいけない（ローマ字入力など）。
- 入力した後に、変換操作によって入力済みの文字列が変わるという奇妙な振る舞いをする。

つまり、キーボードからの入力を監視しようとすると、**「Keydown しても文字が入力されるとは限らない」** し、**「文字が入力されたとしても、あとで変換によって覆る可能性がある」** という、開発者泣かせの仕様なのです。

Linux 開発者の殆どは非日本語文化圏に住んでいます。こんな悪魔のような仕様に従ってくれるのは、歴史あるソフトウェアか、成熟したフレームワークで書かれたシンプルなアプリ程度です。そのため、Linux 使いは度々「日本語入力ができないソフトウェア」に直面することになります。

そこで生み出されたのが、**Vim as an IME** という手法です。アイディアは簡単です。
Emacs の DDSKK や Vim の skkeleton のような、テキストエディタ内で実装された日本語入力プラグインを使い、一時バッファに日本語を書き溜めます。そして入力が終わった段階でそれを Copy and Paste するのです。
これなら、前述の2つの問題を回避できます。

つまり、OS の IME を使うことを諦め、テキストエディタとコピペだけで戦うという「戦略的撤退」です。

## 生成AI時代の Vim as an IME のメリット

この方法は、IME という先人の日本語圏開発者の努力を捨てる蛮行であり、技術的な退化でもあります。当然、不便な点もあります。しかし、現代においては意外なメリットが生まれています。

### 1. GitHub Copilot など、AI技術の恩恵をフルに受けられる

通常、ウェブサイトのテキストエリアはウェブサイト自身によって管理されています。使い心地はそのサイトの実装次第であり、ほとんどの場合、エディタのような高度な AI 支援は受けられません。

ブラウザのサイドバーに AI Chat サービスを立ち上げ、テキストフォームの内容を貼って推敲したり翻訳させたりしているそこのあなた。Vim as an IME なら、そのコピペ作業すら不要です。

Neovim 上で編集するということは、**GitHub Copilot や 独自の AI プラグインがそのまま使える** ということです。ブラウザの入力欄でありながら、文脈を理解した AI の補完をリアルタイムに受けられるのです。

### 2. 日本語入力辞書を統一できる

日本語入力において、どのような変換を多く用いるかは非常に属人的です。
アニメをよく見る人なら難読名字を頻繁に入力するでしょうし、学術関係者なら `jaist` に対して「北陸先端科学技術大学院大学」を割り当てているかもしれません。

IME の辞書は、その人が十分な時間を掛けて育てるものです。OS ごと、あるいは IME ごとにその辞書を育てるのは時間的損失でしかありません。
Vim as an IME なら、Vim/Neovim の設定の一つとして辞書を統一的に管理することができます。普段から dotfiles を管理している開発者なら、入力辞書もコードとして管理できる利便性は計り知れません。

### 3. Linux の IME の沼に嵌らない

Linux で日本語入力を行うには、Uim, iBus, Fcitx5 などの「IME フレームワーク」と、Anthy, Mozc, SKK, KKC などの「変換エンジン」の組み合わせを選ぶ必要があります。
さらにその相性は、GTK 系なのか Qt 系なのか、GNOME なのか KDE なのか……といった様々な要因による制約を受けます。

最近はインストーラーが自動設定してくれるようになり随分楽になりましたが、それでもふとした瞬間に深淵な面倒臭さが顔を出します。Vim as an IME は、この OS レベルの依存関係から我々を解放してくれます。

## Vim as an IME の実践: Firenvim

ここでは実践方法として `firenvim` を紹介します。Firenvim はブラウザ拡張機能ですが、Native Messaging Protocol を使ってブラウザ外にインストールされている Neovim と通信し、Neovim をフロントエンドとして振る舞わせます。

ワークフローは以下の通りです。

1.  ウェブページのテキストフォームをクリックする。
2.  Neovim がオーバーレイ表示されるので、skkeleton 等で日本語を入力する。
3.  （必要に応じて AI の補完を受けながら執筆する）
4.  バッファを保存（`:w`）し、Neovim を閉じる。
5.  テキストフォームには日本語が入力され終わっている！

もはやネイティブ IME と変わらない手間で Vim as an IME が可能です。コピペすら自動化されています。これなら、従来の作業スタイルのまま移行できますね。

## Firenvim のインストール方法

Firenvim は Neovim の拡張機能ですので、通常のプラグインと同様にインストールします（以下は Packer.nvim の例）。

```lua
use { 'glacambre/firenvim' }
```

インストール後、Neovim 側からブラウザと通信するために必要なスクリプトを生成・配置します。

```bash
$ nvim --headless "+call firenvim#install(0) | q"
```

あとはブラウザの拡張機能をインストールするだけです！

- [Firefox Add-ons: Firenvim](https://addons.mozilla.org/en-US/firefox/addon/firenvim/)
- [Chrome Web Store: Firenvim](https://chromewebstore.google.com/detail/firenvim/egpjdkipkomnmjhjmdamaniclmdlobbo)

## Firenvim を使う上での Tips

Firenvim のおかげで、どこでも AI 支援を受けながら快適に日本語入力ができるようになりました。しかし、デフォルトのままだと、最近のお洒落で線の細いウェブデザインの上に、ゴツい TUI のテキストエディタが乗っかるため、強烈な違和感があります。

そこで、Firenvim 起動時だけウェブページに馴染むよう、見た目を調整しましょう。

```lua
if vim.g.started_by_firenvim == true then
  -- 一般的なウェブ入力欄に近づける設定
  vim.opt.signcolumn = 'no'
  vim.opt.laststatus = 0
  vim.opt.background = 'light'
  vim.opt.cursorline = false
  vim.opt.number = false
  vim.opt.fillchars:append({ eob = " " })
  
  -- 必要に応じてカラースキームやStatuslineも調整
  vim.cmd.colorscheme('quiet') -- 例: シンプルなテーマに変更
  vim.g.ministatusline_disable = true -- 例: mini.nvimのステータスラインを無効化
end
```

`vim.g.started_by_firenvim` 変数によって、Firenvim 経由で Neovim が開かれたかどうかを判定できます。

- `vim.opt.signcolumn = 'no'`: 警告用などに付けていた左側の余白列を消します。
- `vim.opt.laststatus = 0`: ステータスラインも不要なので非表示にします。
- `vim.opt.cursorline = false`: テキストエリアは通常カーソル行を強調しないので無効化します。
- `vim.opt.number = false`: 行番号の表示をやめます。
- `vim.opt.fillchars:append({ eob = " " })`: ファイル末尾の余白に表示される `~` は異質なので消します。
- カラーテーマ: ウェブサイトは白背景が多いため、`light` 設定やモノクロテーマの方が馴染みやすいです。

## Vim as an IME でポータブルな環境を作りましょう

Firenvim は Windows (WSL2含む), macOS, Linux のいずれにも対応しています。
つまり、一度 Vim as an IME をセットアップしてしまえば、**どの OS でも全く同じ操作感と辞書で、AI の支援を受けながら日本語入力をすることができる** ようになります。

一見「日本語 IME をやめる」という選択肢は、退化や諦めといったネガティブな動機に見えます。しかしその実は、**「OS に依存しない、AI 支援付きのポータブルな執筆環境を手に入れる」** という、非常に未来的なアプローチなのです。

さぁ、みなさんも Vim as an IME を実践しましょう。