---
title: " Common LispにClojure風の名前空間を実装する"
emoji: "🎄"
type: "tech"
topics: ["lisp"]
published: true
---

Common Lisp (CLtL2) には名前空間を管理する機構が定義されています．
この機構は`in-pacakge`や`import`, `export`を用いてシンボルの名前空間を管理するテーブルを操作します．
普段Common Lispを書いているときには`defpacakge`マクロを使って名前空間を管理していると思いますが，
これは上記の関数のラッパーとして動作しています．

さて，Common Lispには名前空間の管理がすでに存在しますが，
ここで敢えて教育目的として名前空間の管理システムを実装してみようと思います．

一見すると名前空間というのはプログラミング言語の根幹を成す概念であり，そのような
低水準の操作をユーザー側で再実装することができるのか疑問に思うかもしれません．

そもそも名前空間を用いる理由はなんでしょう．
使用しているシンボル間でのシンボル名の衝突を避けるのが目的ですね．
つまり，シンボルに異なるプレフィックスをつけてしまえば，その衝突を避けることができます．
次のコードは関数を上書きしてしまうため名前空間の衝突が起きています．
```lisp
(defun hello () (print 'nihao))
(defun hello () (print 'konnichiwa))
```
そこで次のようにプレフィックスを付けることで回避しましょう。
```lisp
(defun chinese/hello () (print 'nihao))
(defun japanese/hello () (print 'konnichiwa))
```

では毎回シンボルを定義するとき使用するときプレフィックスをつければ解決しますが，
これはいかにも冗長なルールですね．しかもプレフィックスつけるのはソースコードを書く方の責任となります．

そこで名前空間を解決する，つまり直近で定義された名前空間用のプレフィックスをシンボルの付けてくれる，関数を用意しましょう．

```lisp
(defun resolve (stream char)
  (declare (ignore char))
  (read-char stream t nil t)
  ;; シンボルをストリームから取得する
  (let ((sym (read stream t nil t))) 
    ;; シンボルに`*package*`をプレフィックスとしてつける。
    (intern (format nil "~a/~a" *package* sym))))
```

これをリードマクロとしてリーダーに登録すると `~/hello` が `*package*/hello`として扱われます．
ちなみに，これはリード時に解決されるのでソースコードとして実行されたときには既にプレフィックスが付いた
シンボルになっている仕掛けです．

つまり先程のコードは次のようになります．

```lisp
(defparameter *package* 'chinese)
(defun ~/hello () (print 'nihao))
(~/hello) ;; => nihao

(defparameter *package* 'japanese)
(defun ~/hello () (print 'konnichiwa))
(~/hello) ;; => konnichiwa
```

これだけでも、随分とパッケージ管理らしい仕組みになってきました．
しかし，このシステムでは名前空間外の関数を呼ぶときはプレフィックスを付ける必要があります．

```lisp
(japanese/hello) ;; => konnichiwa
(chinsese/hello) ;; => nihao
```

そこで，名前空間外の関数をImportする仕組みを考えます．
幸いCommon Lispには`(setf (fdefinition 'f) #'g)`という関数`g`の定義をシンボル`f`
に代入するという特殊形式が存在します．そこでマクロを用いて

```lisp
(defparameter *package* 'japanese)
(import-functions chinese hello)
```

を

```lisp
(setf (fdefinition 'japanese/hello) #'chinese/hello)
```
に展開することを考えます．これは以下のようになります．

```lisp
(defmacro import (package &rest symbols)
  (when symbols
    `(progn
       (setf (fdefinition ',(intern (format nil "~a/~a" spack/*package* (car symbols))))
             #',(intern (format nil "~a/~a" package (car symbols))))
       (import ,package ,@(cdr symbols)))))
```

これらの準備をすることで簡易的な名前空間を操作をするパッケージが完成しました．

```lisp
(deparameter spack/*package* 'spack)

(defmacro spack/defpackage (package)
  `(defparameter spack/*package* ',package))

(defun spack/resolve (stream char)
  (declare (ignore char))
  (read-char stream t nil t)
  (let ((sym (read stream t nil t)))
    (intern (format nil "~a/~a" spack/*package* sym))))

(set-macro-character #\~ #'spack/resolve)

(defmacro spack/import (package &rest symbols)
  (when symbols
    `(progn
       (setf (fdefinition ',(intern (format nil "~a/~a" spack/*package* (car symbols))))
             #',(intern (format nil "~a/~a" package (car symbols))))
       (spack/import ,package ,@(cdr symbols)))))
```

これを用いれば以下のように名前空間の分離がされたソフトウェア開発ができます．

```lisp
(spack/defpackage example-lib)
(defvar ~/pi 3.141592)
(defun ~/sayhello ()
  (print ~/pi))

(spack/defpackage example-main)
(spack/import example-lib sayhello)
(~/sayhello)
```

名前空間という低水準な操作もリーダーマクロというより低水準なAPIを使えば
簡単に実装できるというデモンストレーションでした．