---
title: "Vimmer のための 括弧編集入門"
emoji: "🖊️"
type: "tech"
topics: ["vim", "neovim"]
published: true
---

# Vimmer のための 括弧編集入門

プログラミングにおいて、もっともよくに入力する文字種はなんだろうか。
そう、括弧である。括弧を効率良く編集することは、どんなプログラミング言語をつかう人にとっても有益である。
その信念のもと、僕が使っている括弧編集の種々を紹介したい。

## 括弧の挿入と削除 1

ドア、箱、ノートパソココン、どんなものも開いたら閉じます。括弧だってそうです。
どうせ閉じることがわかっているのであれば、開き括弧が入力された段階で閉じ括弧も入力したいですね。

vim なら `innoremap` を使うことで実現できます。

```vim
inoremap ( ()<Left>
inoremap { {}<Left>
inoremap [ []<Left>
```

上記は挿入モードで開き括弧が入力されたら自動で閉じ括弧も入力してくれる機能です。
一番簡単に閉じ括弧を自動入力する機能です。

## 括弧の挿入と削除 2

- https://github.com/eraserhd/parinfer-rust
- https://github.com/bhurlow/vim-parinfer
- https://github.com/gpanders/nvim-parinfer

前節では括弧を自動で入力することを紹介しましたが、
もっと詳しく編集操作を分析すると、括弧には興味深い性質があることがわかります。
なんと、開き括弧と閉じ括弧の数は同じなのです。
偶然にも日本にある登り坂の数と下り坂の数が同じだという性質を思い出しますね。

上記の方法では括弧を削除してしまえば
簡単に開き括弧の数と閉じ括弧の数をずらすことができてしまいます。
そしてずれた括弧の原因を把握するのは総じて困難です。

そこで僕はparinfer-rustをつかって括弧を簡単に削除できないようにしています。
このプラグインは括弧を削除しようとすると、対応する開き括弧・閉じ括弧も削除してくれます。

```lisp
(print "hello")| ;; |はカーソル文字. <BS> を入力する
(print "hello"|) ;; 括弧は削除されない!
(|print "hello") ;; 開き括弧の前で <BS> を入力する
|print "hello"   ;; 括弧が削除される！ここで開き括弧 ( を打ってみる

(|print "hello") ;; 行末までを括弧で囲う
(print| "hello") ;; 閉じ括弧 ) を入力する
(print)| "hello" ;; 括弧から "hello" が放出される! ここで <BS>! を入力する
(print |"hello") ;; 行末まで括弧を囲う
```

似たような機能は古くは paredit という emacsのプラグインとしても使われていたものです。
先行のプラグインと比較したとき、 parinfer の良いところは、
括弧の編集操作がキー操作に紐付いておらず、
通常の括弧の挿入と削除がカーソルの周辺の文脈に応じて空気を読んでくれる点にあります。

文脈に応じて挙動が変わるというのは好みがわかれる点ですが、
貴重なキー割り当てを消費せず、既存の機能を強化するという戦略は強いと思います。

## 括弧に色を付ける。

ソースコードの構造を決める大事な要因に括弧があります。
関数呼び出し、データ構造の添字アクセス、変数のスコープ宣言、など、ソースコードの構造は
括弧によって決まります。そんな大事な括弧なのに多くのエディタでは括弧にシンタックスハイライトが付いていません。
あまりに括弧が多すぎて、色が付くと煩いというのもあるでしょうが。
しかし、大抵のテキストエディタには拡張機能という形でこの機能が実装されています。

vimの場合は以下のものがあります。

- https://github.com/luochen1990/rainbow
- https://gitlab.com/HiPhish/rainbow-delimiters.nvim

これは、対応する括弧が同色になるようにカラフルに括弧に色を付けてくれます。
これのおかげで、対応する括弧がどこにあるのか数えなくて良くなります。

## 括弧を置換する。括弧で囲む。

上記で括弧を編集する方法を紹介しましたが、
parinfer は LISP 系のプログラミング言語を念頭に設計されています。
そこで、もうすこし緩い括弧の編集方法を紹介します。

- https://github.com/machakann/vim-sandwich
- https://github.com/echasnovski/mini.surround
- https://github.com/tpope/vim-surround

これらは、括弧 **自体** を 文字よりも抽象度の高く、
vim で扱う編集単位の一つとして扱えるようにしてくれます。

```c
int main(int argc, |char** argv) { // sr([ と入力する。
int main[int argc, |char** argv] { // () が [] になる。
/*
 * s (surround モード)
 * r (replace 操作)
 * ( (丸括弧を編集対象にする)
 * [ (角括弧へ置換する)
 */
```

括弧の削除にも使えます。

```c
int main(int argc, |char** argv) { // sd( と入力する。
int mainint argc, |char** argv { // () が削除される
/*
 * s (surround モード)
 * d (delete 操作)
 * ( (丸括弧を編集対象にする)
 */
```

括弧の挿入にも使えます。

```c
int main(int argc, |char** argv) { // saiw( と入力する。
int main[int argc, |(char)** argv] { // 一単語が 括弧で囲われる。
/*
 * s (surround モード)
 * a (add 操作)
 * iw (単語選択)
 * ( (丸括弧を挿入)
 */
```

このように `sa`, `sd`, `sr` といった 括弧専用の操作 (オペレータ) を追加してくれます。
括弧という頻繁に入力する記号にこそ短いコマンドを割り当てたいですね。

## 括弧の中身を扱う。

今度は括弧が囲むもの自体を編集することを考えましょう。
括弧の中身を一斉に削除したいということがあります。
この機能は vim に標準で備わっています。

```c
int main(int argc, |char** argv) { // `di(` 丸括弧(`(`)の内側 (`i`) だけを削除 (`d`)
int main() { // 括弧の中身がすべて消える!

int main(int argc, |char** argv) { // `da(` 丸括弧(`(`)も含めた全体 (`a`) を削除 (`d`)
int main { // 括弧ごと消える!
```

ところで、括弧を用いた範囲選択は便利ですが、LaTeXのように括弧のパターンが特殊なものがあります。
そういったものも、同様に扱えるようにするのが、`mini.ai` や `vim-textobj-user` です。

- https://github.com/echasnovski/mini.ai
- https://github.com/kana/vim-textobj-user

ここでは、`mini.ai` 例に説明します。以下のコードはLaTeXで数式を挿入するときに使われる括弧 `\\(` と `\\)` や
`\\[` と `\\]`範囲選択できるようにします。

以下のコード使うと、`"a"` か `"i"` かで操作を振り分けつつ、その中身を `m` あるいは `M` で選択できるようになります。
例えば、`dim` は `\\(\\)` で書こまれた数式表現の中身のみを削除します。
`daM` は `\\[\\]` で書こまれた数式表現を括弧ごと削除します。

```lua
local ai = require("mini.ai")
ai.setup({
  custom_textobjects = {
    m = function(t)
      local od = t == "a" and 0 or 2
      local cd = t == "a" and 1 or -1
      local op = vim.fn.searchpos("\\\\(", "bnW")
      local from = { line = op[1], col = op[2] + od }
      local cp = vim.fn.searchpos("\\\\)", "nW")
      local to = { line = cp[1], col = cp[2] + cd }
      return { from = from, to = to }
    end,
    M = function(t)
      local od = t == "a" and 0 or 2
      local cd = t == "a" and 1 or -1
      local op = vim.fn.searchpos("\\\\[", "bnW")
      local from = { line = op[1], col = op[2] + od }
      local cp = vim.fn.searchpos("\\\\]", "nW")
      local to = { line = cp[1], col = cp[2] + cd }
      return { from = from, to = to }
    end,
  },
})
```

言語ごとに括弧に相当するものが違う場合は上記のようなプラグインを用いて括弧を独自定義することになります。
キーワードは テキストオブジェクトです。

## 括弧間を移動する。

括弧は編集されるだけではありません。
対応する括弧に移動したいこともあります。
あまり良いことではありませんが、長大な関数の末尾から、関数の先頭に移動したいことがあります。
このような移動は `%` というコマンドが割り当てられています。

```c
int main(int argc, char** argv) {
// long long code
return 0;
|} // カーソル位置はココ。ここで % を入力する

int main(int argc, char** argv) |{ // カーソル位置がここに移動する!
// long long code
return 0;
}
```

言語では括弧に相当するものは様々ですから、それに併せて移動を定義する必要があります。
HTMLでは、 `<div>` に対応するものは `</div>` です。
これを実現するが、 `vim-matchup` です。
`vim-matchup` は既存の `%` を拡張してどの言語でも対応する括弧に移動できるようにしてくれます。

- https://github.com/andymass/vim-matchup

## さらに複雑な括弧へ

典型的な括弧の操作について紹介してきました。ここで括弧の概念をさらに拡張しましょう。
たとえば、`\\begin{document}` と入力されたら、 `\\end{document}` と入力して欲しいですね？
`<div>` なら `</div>` と入力して欲しいですね。

このように一文字とは限らない複雑な括弧の対応を取るためのフレームワークが `nvim-insx` や `lexima.vim`  です。
強力な Vimの正規表現を使用しているため、バックスラッシュが多く煩雑なコードになっていて恐縮ですが、
以下のコードは HTML と LaTeXで前述のような閉じ括弧を自動補完するためのコードです。

- https://github.com/hrsh7th/nvim-insx
- https://github.com/cohama/lexima.vim
- https://github.com/windwp/nvim-autopairs

```lua
vim.fn["lexima#add_rule"]({
  filetype = "html",
  char = "<",
  input_after = ">",
})
vim.fn["lexima#add_rule"]({
  filetype = "html",
  char = ">",
  at = "<\\(\\w\\+\\)\\%#>",
  leave = ">",
  input_after = "</\\1>",
  with_submatch = true,
})
vim.fn["lexima#add_rule"]({
  filetype = { "tex", "plaintex" },
  char = "<CR>",
  input = "<CR>",
  input_after = "<CR>\\\\end{\\1}",
  at = "^.*\\\\begin{\\(.\\{-}\\)}" .. "\\({.\\{-}}\\|\\[.\\{-}\\]\\)*" .. "\\%#$",
  with_submatch = true,
})
```

## まとめ

この記事では、括弧にまつわる便利な編集方法を、ビルトインと拡張機能の両面から紹介しました。
コーディング支援というと、自動補完など関数名を対象にすることが多いですが、
実は括弧のほうが入力回数が多いことに気付きます。そして、その課題は先人の知恵によって既に解決されています。
是非、いま一度括弧の編集について振り替えってみてください。
