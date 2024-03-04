---
title: "TypeScriptフレンドリーで安全な構造化束縛ができるパターンマッチライブラリ: @core/match"
emoji: "🚀"
type: "tech"
topics: ["javascript", "typescript"]
published: true
---

# TS Match

あたらしいパターンマッチライブラリを作りました。
unknownutilsなどのtypeguardと組み合わせることで、構造化束縛も同時に出来るようになります。
型パズルが裏で動いているので Honoのルータみたいに束縛したところも型情報が残ります。
ほぼREADMEの和訳で恐縮ですがよろしくおねがいします。

https://github.com/tani/ts-match

ECMAScript には構造化束縛があります。これは、複雑な構造から必要な部分だけを抽出するための非常に便利な記法です。ただし、この構造化束縛は、パターンマッチングとして使用するには不完全です。なぜなら、構造化束縛を行うためには、予め割り当てられた値がパターンに一致する必要があり、パターンに一致しない場合、例外がスローされます。

そのため、構造化束縛を行うためには、TypeScriptがコンパイル時に構造が一致することを保証するか、zodやunknownutilsなどのバリデーションライブラリが実行時に構造が一致することを確認します。前者は、JSONデータのように、コンパイル時に構造が決まっていないデータには無力です。後者は、構造化束縛パターンと検証パターンを2つずつ書く必要があります。

このライブラリは、コンパイル時の型情報を保持しながら、構造化束縛とバリデーションを同時に行うことができ、EcmaScriptの構造化束縛を完成させ、真のパターンマッチングを可能にするライブラリです。

また、これはTypeScriptだけでなく、JavaScriptでも実行時構造化束縛の安全性を高めます。

## 使い方

このライブラリはJSRで公開されており、`jsr:@core/match`でDenoで使用できます。
ユーザーが覚える必要があるのは、`$`と`match`の2つの関数だけです。

```ts
import { placeholder as $, match } from 'jsr:@core/match';
```

- `$`は、構造化束縛用のパターンを作成するための関数です。

    ```ts
    const pattern = {
        name: $('name'), // この値はunknown valueとしてキャプチャされる
        address: {
            country: $('country'), // プレースホルダはどこにでも書けます
            state: 'NY', // プレースホルダなしでも、マッチャーは===を使って値を比較する
        },
        age: $('age', isNumber), // 型ガードでプレースホルダの型を指定できます。
        favorites: ['baseball', $('favorite')], // 配列にプレースホルダを入れることができます。
        others: [$(1), $(Symbol.other)], // 数字やシンボルでプレースホルダを宣言できます。
    }
    ```

- `match`は、構造化束縛を行うための関数です。
  上記のパターンに基づいて構造化束縛を実行すると、次の`Result`型に対応する値は、
  - プレースホルダとして宣言された名前がキーのオブジェクトであり、
  - 型ガードが指定されたプレースホルダは、その型になります。
  - 構造が一致しない場合や型ガードが失敗した場合、`undefined`が返されます。

  ```ts
  type Result = {
    [1]: unknown,
    [Symbol.other]: unknown
    name: unknown,
    country: unknown,
    age: number,
    favorite: unknown
  } | undefined;
  const result: Result = match(pattern, value);
  ```

## 型ガードの宣言方法

TypeScriptでは、型ガードは`(v: unknown) => v is T`型の関数であり、以下のように宣言できます。unknownutilsのようなジェネリック型ガードのコレクションもあります。

```ts
function isNumber(v: unknown): v is number {
    return typeof v === "number";
}
```

---

## このライブラリの使用シナリオ

まず、以下のライブラリを読み込んでださい。

```ts
import { placeholder as $, match } from 'jsr:@core/match';
import { assertEquals } from 'jsr:@std/assert';
```

このライブラリは、オブジェクトが特定のパターンに一致するかどうかを確認するために使用できます。
次の例は、単純な文字列値でオブジェクトを一致させる方法を示しています。
オブジェクトがパターンに一致する場合、結果は空のオブジェクトです。
これは、パターンにプレースホルダがまだ含まれていないためです。

```ts
Deno.test('01 match object with primitive string value', () => {
    const pattern = 'hello';
    const value = 'hello';
    const result = match(pattern, value);
    assertEquals(result, {});
});
```

数値をパターンとして使用することもできます。

```ts
Deno.test('02 match object with primitive number value', () => {
    const pattern = 123;
    const value = 123;
    const result = match(pattern, value);
    assertEquals(result, {});
});
```

論理値もパターンとして使用できます。

```ts
Deno.test('03 match object with primitive boolean value', () => {
    const pattern = true;
    const value = true;
    const result = match(pattern, value);
    assertEquals(result, {});
});
```

Null値をパターンとして使用することもできます。パターンと値がどちらもNullの場合でも、match関数は空のオブジェクトを返すことに注意してください。

```ts
Deno.test('04 match object with primitive null value', () => {
    const pattern = null;
    const value = null;
    const result = match(pattern, value);
    assertEquals(result, {});
});
```

Undefined値もパターンとして使用できます。パターンと値がどちらもUndefinedの場合でも、match関数は空のオブジェクトを返すことに注意してください。

```ts
Deno.test('05 match object with primitive undefined value', () => {
    const pattern = undefined;
    const value = undefined;
    const result = match(pattern, value);
    assertEquals(result, {});
});
```

Symbol値もパターンとして使用できます。

```ts
Deno.test('06 match object with primitive symbol value', () => {
    const symbol = Symbol('hello');
    const pattern = symbol;
    const value = symbol;
    const result = match(pattern, value);
    assertEquals(result, {});
});
```

複合オブジェクトをパターンとして使用することもできます。オブジェクトがパターンに一致する場合、結果はまだ空のオブジェクトです。これは、パターンにプレースホルダが含まれていないためです。

```ts
Deno.test('07 match object with compound object value', () => {
    const pattern = { name: 'hello', age: 123 };
    const value = { name: 'hello', age: 123 };
    const result = match(pattern, value);
    assertEquals(result, {});
});
```

配列もパターンとして使用できます。オブジェクトがパターンに一致する場合、結果はまだ空のオブジェクトです。これは、パターンにプレースホルダが含まれていないためです。

```ts
Deno.test('08 match object with array value', () => {
    const pattern = ['hello', 123];
    const value = ['hello', 123];
    const result = match(pattern, value);
    assertEquals(result, {});
});
```

では、プレースホルダを使ってみましょう。まず、単一のプレースホルダを宣言します。プレースホルダは`$`関数を使って宣言され、プレースホルダの名前が引数として渡されます。プレースホルダは、名前の文字列値と、`test`という型ガード関数を保持します。次のプレースホルダは、型ガードがないため、最も単純な形式のプレースホルダです。

```ts
Deno.test('09 declare single placeholder', () => {
    const pattern = $('a');
    assertEquals(pattern.name, 'a');
    assertEquals(pattern.test, undefined);
});
```

プレースホルダを使用するには、match関数を適用します。match関数は、プレースホルダの名前と、関連するオブジェクトの値のキー・値ペアを含むオブジェクトを返します。オブジェクトがパターンに一致しない場合、match関数はundefinedを返します。結果のオブジェクトには、`a`というキーがあり、オブジェクトの値は`hello`で、プレースホルダが型ガードを持っていないため、TypeScriptでは型は`unknown`です。

```ts
Deno.test('10 match object with single placeholder', () => {
    const pattern = $('a');
    const value = 'hello';
    const result = match(pattern, value);
    assertEquals(result, { a: 'hello' });
});
```

プレースホルダに型ガードを与えるには、`$`関数の第二引数に型ガード関数を宣言します。

```ts
Deno.test('11 match object with single placeholder and type guard', () => {
    const pattern = $('a', (v: unknown): v is string => typeof v === 'string');
    const value = 'hello';
    const result = match(pattern, value);
    assertEquals(result, { a: 'hello' });
});
```

型ガードが失敗した場合、match関数はundefinedを返します。

```ts
Deno.test('12 match object with single placeholder and type guard', () => {
    const pattern = $('a', (v: unknown): v is string => typeof v === 'string');
    const value = 123;
    const result = match(pattern, value);
    assertEquals(result, undefined);
});
```

プレースホルダを複合オブジェクトで使うこともできます。次のパターンでは、`a`と`b`の2つのプレースホルダがあります。結果のオブジェクトには、`a`と`b`のキーがあり、オブジェクトの値はそれぞれ`hello`と`123`です。また、TypeScriptでの`a`の値の型は`unknown`であり、`b`の値の型は`unknown`です。

```ts
Deno.test('13 match object with compound object and placeholders', () => {
    const pattern = { name: $('a'), age: $('b') };
    const value = { name: 'hello', age: 123 };
    const result = match(pattern, value);
    assertEquals(result, { a: 'hello', b: 123 });
});
```

同様に、型ガードをプレースホルダに渡すことができます。この場合、`a`の値の型は`string`であり、`b`の値の型は`number`です。

```ts
Deno.test('14 match object with compound object and placeholders (type guard)', () => {
    const pattern = { 
        name: $('a', (v: unknown): v is string => typeof v === 'string'), 
        age: $('b', (v: unknown): v is number => typeof v === 'number') 
    };
    const value = { name: 'hello', age: 123 };
    const result = match(pattern, value);
    assertEquals(result, { a: 'hello', b: 123 });
});
```

オブジェクトがパターンに一致しない場合、match関数はundefinedを返すことが期待されます。次のパターンには、`a`と`b`の2つのプレースホルダがあります。オブジェクトの値は、パターンに一致しないため、`name`キーが欠けています。

```ts
Deno.test('15 match object with compound object and placeholders (missing key)', () => {
    const pattern = { name: $('a'), age: $('b') };
    const value = { age: 123 };
    const result = match(pattern, value);
    assertEquals(result, undefined);
});
```

他の場合では、型ガードが失敗すると、match関数はundefinedを返します。次のパターンには、`a` と `b` の2つのプレースホルダがあります。オブジェクトの値は、`age` 値が数値でないため、パターンに一致しません。

```ts
Deno.test('16 match object with compound object and placeholders (type guard fail)', () => {
    const pattern = { 
        name: $('a', (v: unknown): v is string => typeof v === 'string'), 
        age: $('b', (v: unknown): v is number => typeof v === 'number') 
    };
    const value = { name: 'hello', age: '123' };
    const result = match(pattern, value);
    assertEquals(result, undefined);
});
```

配列でもプレースホルダを使用できます。次のパターンには、`a` と `b` の2つのプレースホルダがあります。結果のオブジェクトには、`a` と `b` のキーがあり、オブジェクトの値はそれぞれ `hello` および `123` です。さらに、`a` の値の型は TypeScript で `unknown`、`b` の値の型は `unknown` です。

```ts
Deno.test('17 match object with array and placeholders', () => {
    const pattern = [$('a'), $('b')];
    const value = ['hello', 123];
    const result = match(pattern, value);
    assertEquals(result, { a: 'hello', b: 123 });
});
```

前と同じように、プレースホルダに型ガードを渡すことができます。これで、`a`の値の型は`string`で、`b`の値の型は`number`になります。

```ts
Deno.test('18 match object with array and placeholders (type guard)', () => {
    const pattern = [
        $('a', (v: unknown): v is string => typeof v === 'string'), 
        $('b', (v: unknown): v is number => typeof v === 'number') 
    ];
    const value = ['hello', 123];
    const result = match(pattern, value);
    assertEquals(result, { a: 'hello', b: 123 });
});
```

オブジェクトがパターンに一致しない場合、match関数はundefinedを返すことが期待されます。次のパターンには、`a`と`b`の2つのプレースホルダがあります。オブジェクトの値は、パターンに一致しないため、`name`キーが欠けています。

```ts
Deno.test('19 match object with array and placeholders (missing key)', () => {
    const pattern = [$('a'), $('b')];
    const value = [123];
    const result = match(pattern, value);
    assertEquals(result, undefined);
});
```

ほかの場合では、型ガードが失敗すると、match関数はundefinedを返します。次のパターンには、`a`と`b`の2つのプレースホルダがあります。オブジェクトの値は、`age`値が数値でないため、パターンに一致しません。

```ts
Deno.test('20 match object with array and placeholders (type guard fail)', () => {
    const pattern = [
        $('a', (v: unknown): v is string => typeof v === 'string'), 
        $('b', (v: unknown): v is number => typeof v === 'number') 
    ];
    const value = ['hello', '123'];
    const result = match(pattern, value);
    assertEquals(result, undefined);
});
```

オブジェクトと配列の両方でプレースホルダを使用できます。次のパターンには、`a`および`b`の2つのプレースホルダがあります。結果のオブジェクトには`a`および`b`キーがあり、オブジェクトの値はそれぞれ`hello`および`123`です。さらに、`a`の値の型はTypeScriptで`unknown`が`b`の値の型は`unknown`です。

```ts
Deno.test('21 match object with object, array, and placeholders', () => {
    const pattern = { name: $('a'), age: [$('b')] };
    const value = { name: 'hello', age: [123] };
    const result = match(pattern, value);
    assertEquals(result, { a: 'hello', b: 123 });
});
```

同じように、型ガードをプレースホルダに渡すことができます。これで、`a`の値の型は`string`で、`b`の値の型は`number`になります。

```ts
Deno.test('22 match object with object, array, and placeholders (type guard)', () => {
    const pattern = { 
        name: $('a', (v: unknown): v is string => typeof v === 'string'), 
        age: [$('b', (v: unknown): v is number => typeof v === 'number')] 
    };
    const value = { name: 'hello', age: [123] };
    const result = match(pattern, value);
    assertEquals(result, { a: 'hello', b: 123 });
});
```

オブジェクトがパターンに一致しない場合、match関数はundefinedを返すことが期待されます。次のパターンには、`a`および`b`の2つのプレースホルダがあります。オブジェクトの値は、`name`キーが欠けているため、パターンに一致しません。

```ts
Deno.test('23 match object with object, array, and placeholders (missing key)', () => {
    const pattern = { name: $('a'), age: [$('b')] };
    const value = { age: [123] };
    const result = match(pattern, value);
    assertEquals(result, undefined);
});
```

他の場合では、型ガードが失敗すると、match関数は undefined を返します。次のパターンには、`a` および `b` の2つのプレースホルダがあります。オブジェクトの値は、`age` 値が数値でないため、パターンに一致しません。

```ts
Deno.test('24 match object with object, array, and placeholders (type guard fail)', () => {
    const pattern = { 
        name: $('a', (v: unknown): v is string => typeof v === 'string'), 
        age: [$('b', (v: unknown): v is number => typeof v === 'number')] 
    };
    const value = { name: 'hello', age: ['123'] };
    const result = match(pattern, value);
    assertEquals(result, undefined);
});
```

ニッチな機能として、プレースホルダとパターンマッチングをユーザー定義のクラスで使用できます。結果のオブジェクトには、`name`キーと`age`キーがあり、オブジェクトの値はそれぞれ`hello`および`123`です。さらに、TypeScriptでの`name`の値の型は`unknown`であり、`age`の値の型は`number`です。

```ts
class User {
    name: string;
    age: number;
    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }
}
Deno.test('25 match object with user-defined class and placeholders', () => {
    const pattern = { name: $('name'), age: $('age', (v: unknown): v is number => typeof v === 'number') };
    const value = new User('hello', 123);
    const result = match(pattern, value);
    assertEquals(result, { name: 'hello', age: 123 });
});
```

同様に、型ガードをプレースホルダに渡すことができます。これで、`name`の値の型は`string`で、`age`の値の型は`number`になります。

```ts
Deno.test('26 match object with user-defined class and placeholders (type guard)', () => {
    const pattern = { 
        name: $('name', (v: unknown): v is string => typeof v === 'string'), 
        age: $('age', (v: unknown): v is number => typeof v === 'number') 
    };
    const value = new User('hello', 123);
    const result = match(pattern, value);
    assertEquals(result, { name: 'hello', age: 123 });
});
```

オブジェクトがパターンに一致しない場合、match関数はundefinedを返すことが期待されます。次のパターンには、`name`および`age`の2つのプレースホルダがあります。オブジェクトの値は、パターンに一致しないため、`name`キーが欠けています。

```ts
Deno.test('27 match object with user-defined class and placeholders (missing key)', () => {
    const pattern = { name: $('name'), age: $('age') };
    const value = new User('hello', 123);
    const result = match(pattern, value);
    assertEquals(result, { name: 'hello', age: 123 });
});
```

さらに、パターンが`Object`または`Array`の直接のインスタンスではない場合、match関数は `===` 演算子を使ってパターンと値の等しさをチェックしようとします。
そのため、次のパターンは値と一致せず、match関数は undefined を返します。

```ts
Deno.test('28 match object with primitive string value (not equal)', () => {
    const pattern = new User('hello', 123);
    const value = new User('hello', 123);
    const result = match(pattern, value);
    assertEquals(result, undefined);
});
```