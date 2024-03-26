# djot チートシート

## 基本

| マークアップ | 結果 |
| --- | --- |
| `_italic_` 、 `{_italic_}` | *italic* |
| `*bold*` 、 `{*bold*}` | **bold** |
| `` `verbatim/code` `` | `verbatim/code` |
| `H~2~O` | H<sub>2</sub>O |
| `20^th^` | 20<sup>th</sup> |
| `{=highlighted=}` | <mark>highlighted</mark> |
| `{+insert+}` | <ins>insert</ins> |
| `{-delete-}` | <del>delete</del> |

段落内では、1行の改行は空白として解釈されます。
段落間は空行を残します。

**引用ブロック**: 

~~~
> これは
> 引用です
~~~

**コメント**: 

 `{% こんな感じ`\
`で複数行対応です %}`。

**主題区切り**（水平線）: 

`***` 、 `---` を1行に書く。
3文字以上、空白やタブでインデントしてもよい。

## そのまま（Verbatim）ブロック

フェンスで囲まれたVerbatimブロックには3つのバックティックを使用します。

~~~~~~
```
$ tree
.
├── aa
│   └── foo.txt
├── bb
│   └── bar.txt
└── c.png
```
~~~~~~

Verbatimブロック自体にトリプルバックティックが含まれている場合は、より多くのバックティックを使用します。バックティックの代わりにチルダを使用することもできます。

### コード ブロック

Varbatimブロックがコードブロックの場合は、始めのバックティック直後に言語名を追加して、構文を強調表示できます: 

~~~~~~
```mylang
func say-hello(nm) {
    print("hello ${nm}!");
}
```
~~~~~~

## 数式

LaTeX形式を使います `` $`p = mv` `` インライン数式と

`` $$`E = K + U` ``

数式表示

## 特殊文字

  * `...` → … (省略記号)
  * `--` → – (エン ダッシュ)
  * `---` → — (全角ダッシュ)
  * 直線のクォート`'` (`"foo"`, `bar's`) は曲線スタイル `’` ("foo"、bar`’`s) に変換されます（日本語ではあまり使わない）。
  * 句読点文字をバックスラッシュでエスケープして、その特別な意味を削除します。
  * バックスラッシュ空白 = 非改行空白。
  * 行末のバックスラッシュ = 強制改行。

## リンク

~~~
<https://example.com>
[read more](https://example.com)
[read this too][foo bar]
[one more link][]
~~~

はこうなります。

~~~
[foo bar]: https://example.com
[one more link]: https://example2.com
~~~

## Images

~~~
![beautiful skyline](clouds.jpg)
![coastal shores][shore]
![lush forests][]

[shore]: the-beach.jpg
[lush forests]: pines.jpg
~~~

## 見出し

~~~
# H1
## H2
### H3
~~~

etc.

## リスト

~~~
項目リスト:

 * lions
 * tigers
 * bears
~~~

同じレベルの項目に同じインデントを使用して、リストマーカーを0個以上の空白でインデントできます。後続の段落をリスト項目の一部にしたい場合は、少なくともリストマーカーで用いた以上の空白でインデントします。

後続の段落と同様に、サブリストの前に空行を置く必要があります (ブロックレベルの要素の間には、ほとんどの場合空行が必要です)。サブリストマーカーも、少なくとも前のリスト項目のマーカーで用いた以上の空白でインデントする必要があります。

~~~
その他のリスト:

 * 1項目目 この項目はかなり長い行があり、見
   栄えを良くするために改行の後に
   インデントされている。

 * 2項目目 この項目もかなり長い行があるが、
きれいなインデントはない
 
   2項目目の2段落目

    - サブリスト
    - まだ
    - 2項目目です。

 * 3項目目

数字リスト:

 1. item one
 2. item two
 3. item three

定義リスト:

: ライオン

  Cプログラムのようにこれがメイン
  
: トラ

  シベリアン、ベンガルそしてトニー・ザ・タイガー
  
: クマ

  ティディもヨギも仲間

~~~

## テーブル

~~~
| 名前 | サイズ |色 |
| --- | --- | --- |
| lime | small | green |
| orange | medium | orange |
| grapefruit | large | yellow or pink |
~~~

区切り行にコロンを使ってセルの並びを変更したり、オプションで空白を追加してセルを整列させることもできる、

例: 

~~~
| Material | Quantity | Catch-phrase  |
| -------- | -------: | :-----------: |
| cotton   |       42 |   Practical!  |
| wool     |       17 |     Warm!     |
| silk     |        4 |    Smooth!    |
~~~


## 脚注

~~~
The proof is elementary.[^proof]

...

[^proof]: TODO. Haven't figured this out yet.
~~~


## スパンとDiv

~~~
これは [スパン]{.some-class #some-id some-key="some val"}.
~~~

これは div:

~~~
{.some-class #some-other-id some-key="some val"}
:::
Div content
goes here.
:::
~~~

これらのショートハンドは:

~~~
::: warning
Watch out!
:::
~~~

以下と等価です。

~~~
{.warning}
:::
Watch out!
:::
~~~

属性 `_onto any inline_{.greenish}` を付加することができ、任意のブロック（通常の段落を含む）の前に属性を付けることができます：

~~~
{.classy}
This paragraph is wearing
a top hat and cuff links.

{source="personal-experience"}
> More than three people on one
> bicycle is *not* recommended.
~~~


## Rawコンテンツ

djotはHTML中心ではありません。どんなフォーマットでもインラインでもブロックでもRaw コンテンツを追加することができますが、明示的にマークされなければなりません：

~~~~~~
We had `<sometag>foos</sometag>`{=html} for dinner.
Then wrapped the leftovers in `\LaTeX`{=latex}.

```{=html}
<video width="320" height="240" controls>
  <source src="movie.mp4" type="video/mp4">
  <source src="movie.ogg" type="video/ogg">
  Your browser does not support the video tag.
</video>
```
~~~~~~

Raw コンテンツは指定されたフォーマットをレンダリングするときにそのまま渡され、そうでない場合は無視される。
