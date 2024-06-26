# WIP: Djot 構文リファレンス

## インライン構文

### 優先順位

> この文書は 2024-02-27 現在、作業中です。

ほとんどのインライン構文は、インライン コンテンツを囲む前後の区切り文字によって定義され、強調表示またはリンク テキストなどとして定義されます。インライン コンテナの「優先順位」を決定する基本原則は、最初に閉じられたオープナーが優先されるということです。コンテナは重複できないため、オープナーが閉じられると、オープナーとクローザーの間の潜在的なオープナーは通常のテキストとしてマークされ、インライン構文を開くことができなくなります。たとえば、

```
_This is *regular_ not strong* emphasis
```

<p><em>This is *regular</em> not strong* emphasis</p>

最初の強調 `_` によって開かれた通常の強調は2番目の `_` によって閉じられ、その時点で強い強調`*`は強い強調を開始する資格を失います。

```
*This is _strong* not regular_ emphasis
```

<p><strong>This is _strong</strong> not regular_ emphasis</p>

逆もまた同じです。

同様に、

```
[Link *](url)*
```

<p><a href="url">Link *</a>*</p>

はリンクの処理だけ行われます。

```
*Emphasis [*](url)
```

<p><strong>Emphasis [</strong>](url)</p>

リンクは無効です（強い強調が区切り文字 `[` を超えて終了するため）。

重複するコンテナは除外されますが、ネスト（入れ子に）されたコンテナは問題ありません。

```
_This is *strong within* regular emphasis_
```

<p><em>This is <strong>strong within</strong> regular emphasis</em></p>

曖昧な場合は、区切り文字をオープナー `{` またはクローザー `}` としてマークに使用される場合があります。したがって、`{_` は `_` のように動作しますが、強調を開くことしかできません。一方 `_}` は `_` のように動作しますが強調を閉じることしかできません。

```
{_Emphasized_}
_}not emphasized{_
```

<p><em>Emphasized</em>
_}not emphasized{_</p>

明示的にマークされたクローザーは明示的にマークされたオープナーとのみ一致し、マークされていないクローザーはマークされていないオープナーのみと一致します（したがって、たとえば `{_hi_)` は強調を生成しません）。

特定のクローザーと一致する可能性のあるオープナーが複数ある場合は、最も近いものが使用されます。例えば：

```
*not strong *strong*
```

は

<p>*not strong <strong>strong</strong></p>

### 通常のテキスト

特別な意味が与えられていないものはすべてリテラル テキストとして解析されます。

すべての ASCII 句読点文字 (djot で特別な意味を持たないものであっても) はバックスラッシュでエスケープできます。したがって、`\*` はリテラル文字 `*` が含まれます。ASCII 句読点文字以外の文字の前のバックスラッシュは、次の例外を除き、リテラルのバックスラッシュとして扱われます。

改行の前（または改行に続くスペースまたはタブの前）のバックスラッシュは、ハード改行として解析されます。この場合、バックスラッシュの前のスペースとタブ文字は無視されます。

スペースの前のバックスラッシュは非改行スペースとして解析されます。

### リンク

リンクには、インラインリンクと参照リンクの2種類があります。

どちらの種類も `[ … ]` の内側でリンク テキスト（任意のインライン形式が含まれる場合があります）から始まります。

インライン リンクには、括弧内にリンク先 (URL) が含まれます。リンクテキストの末尾の `]` とリンク先を定義するオープナー `(` の間にスペースを入れてはいけません。

```
[My link text](http://example.com)
```

URLは複数行に分かれている場合があります。この場合、改行が行われ先頭と末尾のスペースは無視され行が連結されます。

```
[My link text](http://example.com?product_number=23423423423
4234234234234)
```

参照リンクでは、括弧内の宛先ではなく、角括弧内の参照ラベルを使用します。これはリンク テキストの直後に置く必要があります。

```
[My link text][foo bar]

[foo bar]: http://example.com
```

参照ラベルはドキュメント内のどこかで定義する必要があります。以下の [参照リンクの定義] を参照してください。ただし、リンクの解析は「ローカル」でありラベルが定義されているかどうかには依存しません。

```
[foo][bar]
```

ラベルが空の場合、リンク テキストはリンク テキストだけでなく参照ラベルとしても解釈されます。

```
[My link text][]

[My link text]: /url
```

### 画像

画像はリンクと同じように機能しますが、接頭辞 `!` が付いています。リンクと同様にインライン バリアントと参照バリアントの両方が可能です。

```
![picture of a cat](cat.jpg)

![picture of a cat][cat]

![cat][]

[cat]: feline.jpg
```

### オートリンク

`<` … `>` で囲まれたURLまたはメールアドレスはハイパーリンクになります。尖中括弧間の内容は文字通りに扱われます（バックスラッシュ エスケープは使用できません）。

```
<https://pandoc.org/lua-filters>
<me@example.com>
```

URLまたはメール アドレスには改行を含めることはできません。

### 逐語的に（Verbatim）

逐語的なコンテンツは、連続するバッククォート文字 `( ` )` の文字列で始まり、同じ長さの連続するバッククォート文字の文字列で終わります。

```
``Verbatim with a backtick` character``
`Verbatim with three backticks ``` character`
```

<p><code>Verbatim with a backtick` character</code>
<code>Verbatim with three backticks ``` character</code></p>

バッククォート間の内容は逐語的なテキストとして扱われます（そこではバックスラッシュ エスケープは機能しません）。

コンテンツがバッククォート文字で開始または終了する場合、開始または終了バッククォートとコンテンツの間の1つのスペースが削除されます。

```
`` `foo` ``
```

<p><code>`foo`</code></p>

インラインとして解析されるテキストが、終了バッククォート文字列に遭遇する前に終了する場合、逐語的テキストは最後まで続きます。

```
`foo bar
```

<p><code>foo bar</code></p>

### 強調/強い強調

強調されたインライン コンテンツは `_` 文字で区切られます。強い強調は `*` 文字で区切られます。

```
_emphasized text_

*strong emphasis*
```

また `_` は、`*` 直後に空白が続かない場合にのみ強調を開始できます。強調を閉じることができるのは、直前に空白がない場合、および開始文字と終了文字の間に区切り文字以外の文字がある場合のみです。

```
_ Not emphasized (spaces). _

___ (not an emphasized `_` character)
```

強調は入れ子にすることができます。

```
__emphasis inside_ emphasis_
```

<p><em><em>emphasis inside</em> emphasis</em></p>

中括弧 `{` は、`_` または `*` オープナーまたはクローザーとして強制的に解釈するために使用できます。

```
{_ this is emphasized, despite the spaces! _}
```

<p><em> this is emphasized, despite the spaces! </em></p>

### ハイライト表示

`{=` と `=}` の間のインライン コンテンツは、強調表示されたテキストとして扱われます（HTMLでの `<mark>`）。`{` と `}` は必須であることに注意してください。

```
This is {=highlighted text=}.
```

<p>This is <mark>highlighted text</mark>.</p>

### 上付き/下付き

上付き文字は `^` で区切られ、下付き文字は `~` で区切られます。

```
H~2~O and djot^TM^
```

<p>H<sub>2</sub>O and djot<sup>TM</sup></p>

中括弧を使用することもできますが、必須ではありません。

```
H{~one two buckle my shoe~}O
```

<p>H<sub>one two buckle my shoe</sub>O</p>

###  挿入/削除

インラインテキストを挿入済みとしてマークするには、`{+` と `+}` を使用します。削除済みとしてマークするには、`{-` と `-}` を使用します。`{` と `}` は必須です。

```
My boss is {-mean-}{+nice+}.
```

<p>My boss is <del>mean</del><ins>nice</ins>.</p>

### スマート句読点

直線の二重引用符`（"）`と一重引用符`（'）`は中引用符として解析されます。Djotは、文脈からどの方向の引用が必要かを判断するのが非常に上手です。

```
"Hello," said the spider.
"'Shelob' is my name."
```

<p>“Hello,” said the spider.
“‘Shelob’ is my name.”</p>

ただし、そのヒューリスティックは中かっこを使用した引用符をオープナー `{"` またはクローザー `"}` としてマークすることでオーバーライドできます。

```
'}Tis Socrates' season to be jolly!
```

<p>’Tis Socrates’ season to be jolly!</p>

直接引用符が必要な場合は、バックスラッシュとエスケープを使用します。

```
5\'11\"
```

<p>5'11"</p>

- ピリオド3つのシーケンスは _ellipses_ として解析されます。`...` 
- ハイフン3つのシーケンスは _em-dash_ として解析されます。 `—` 
- ハイフン2つのシーケンスは _en-dash_ として解析されます。 `--` 

```
57--33 oxen---and no sheep...
```

<p>57–33 oxen—and no sheep…</p>

より長いハイフンのシーケンスは、全角ダッシュ、半角ダッシュ、およびハイフンに分割されます。可能であれば均一に、どちらの方法でも均一性が達成できる場合は、全角ダッシュを使用することを推奨します。（つまり、4つのハイフンは2つの半角ダッシュになり、6つのハイフンは2つの半角ダッシュになります）。

```
a----b c------d
```

<p>a––b c——d</p>

### 数式

LaTeX 数式を含めるには、その数式を逐語的なスパンに入れ、その先頭に `$` （インラインの場合） または `$$` （数式表示の場合） を付けます。

```
Einstein derived $`e=mc^2`.
Pythagoras proved
$$` x^n + y^n = z^n `
```

<p>Einstein derived <span class="math inline">\(e=mc^2\)</span>.
Pythagoras proved
<span class="math display">\[ x^n + y^n = z^n \]</span></p>

### 脚注参照

脚注参照は、`[^参照ラベル]` です。

```
Here is the reference.[^foo]

[^foo]: And here is the note.
```

脚注自体の構文については、以下の [脚注] を参照してください。

### 改行

インライン コンテンツ内の改行は「ソフト」改行として扱われます。これらはスペースとして、または（HTMLなど、改行が意味的にスペースのように扱われるコンテキストでは）改行としてレンダリングされる場合があります。

ハード改行の（HTMLで`<br>`）を取得するには、バックスラッシュ + 改行を使用します。

```
This is a soft
break and this is a hard\
break.
```

<p>This is a soft
break and this is a hard<br>
break.</p>

### コメント

属性内で `%` に挟まれた内容はコメントとして扱われます。これにより属性にコメントを追加できるようになります。

```
{#ident % later we'll add a class %}
```

ただし、コメントを追加する一般的な方法としても機能します。コメントのみを含む属性指定子を使用してください。

```
Foo bar {% This is a comment, spanning
multiple lines %} baz.
```

<p>Foo bar  baz.</p>

### 記号

単語を:記号で囲むと「記号」が作成され、デフォルトでは文字通りに表示されるだけですが、フィルターによって特別に処理される場合があります。（たとえば、フィルターを使用して記号を絵文字に変換できます。ただし、これは djot には組み込まれていません）。

```
My reaction is :+1: :smiley:.
```

<p>My reaction is :+1: :smiley:.</p>

### Raw インライン

任意の形式のRawインライン コンテンツは、逐語的なスパンとそれに続く次の文字列を使用して含めることができます `{=FORMAT}` 。

```
This is `<?php echo 'Hello world!' ?>`{=html}.
```

このコンテンツは、指定された形式をレンダリングするときにそのまま渡されるように意図されていますが、それ以外の場合は無視されます。

### スパン

リンクや画像ではなく、直後に属性が続く角括弧内のテキストは、一般的なスパンとして扱われます。

```
It can be helpful to [read the manual]{.big .red}.
```

### インライン属性

属性は中括弧内に配置され、属性が付加されるインライン要素のすぐ後に続く必要があります(間に空白は入れません)。

中括弧内では、次の構文を使用できます。

- `.foo`はfooクラスとして指定します。この方法で複数のクラスを指定できます。それらは結合されます。
- `#foo`はfoo識別子として指定します。要素は識別子を 1 つだけ持つことができます。複数の識別子が指定された場合は、最後の識別子が使用されます。
- `key="value"`または、`key=valueキー`と値の属性を指定します。値が ASCII 英数字だけで構成されている場合、または`_`または:は必要ありません-。バックスラッシュエスケープは引用符で囲まれた値の内部で使用できます。
- `%`コメントが始まり、次の属性`%`または属性の終わり (`}`) で終わります。

属性指定子には改行が含まれる場合があります。

例：

```
An attribute on _emphasized text_{#foo
.bar .baz key="my value"}
```

属性指定子は「積み重ね」ることができ、その場合は結合されます。したがって、

```
avant{lang=fr}{.blue}
```

と同じです

```
avant{lang=fr .blue}
```

### ブロック構文

commonmark と同様に、ブロック構造はインライン解析の前に識別でき、インライン構造よりも優先されます。

実際、ブロックはバックトラックなしで行ごとに解析できます。ラインがブロックレベルの構造に与える影響は、将来のラインに依存することはありません。

インデントは、リスト項目または脚注のネストの場合にのみ重要です。

ブロックレベルの項目は空白行で区切る必要があります。2 つのブロック レベル要素が隣接する場合があります。たとえば、主題区切りまたはフェンスで囲まれたコード ブロックの直後に段落が続くことがあります。実際、行ごとに解析できる可能性があるため、ブロックレベル要素の後に空白行を入れる必要はありません。ただし、読みやすさを考慮して、ブロックレベルの要素は常に空行で区切ることをお勧めします。段落は他のブロックレベルの要素によって中断されることはなく、常に空行 (または文書またはそれを含む要素の終わり) で終了する必要があります。

###  段落

段落は、他のブロックレベル要素の 1 つであるための条件を満たさない、一連の非空白行です。テキスト コンテンツは、一連のインライン要素として解析されます。改行はソフト ブレークとして扱われ、書式設定された出力ではスペースと同様に解釈されます。段落は空白行または文書の終わりで終わります。

### 見出し

見出しは 1 つ以上の文字のシーケンスで始まり#、その後に空白が続きます。文字数によって#見出しレベルが定義されます。次のテキストはインライン コンテンツとして解析されます。

```
## A level _two_ heading!
```

見出しテキストは後続の行にまたがる場合があり、その前に同じ数の#文字が続く場合もあります (ただし、省略することもできます)。

見出しは、空行 (または文書またはそれを囲んでいるコンテナーの終わり) に達すると終了します。

```
# A heading that
# takes up
# three lines

A paragraph, finally
```

```
# A heading that
takes up
three lines

A paragraph, finally.
```

### 引用ブロック

ブロック引用符は、各行が `>` で始まり、その後にスペースまたは行末が続く一連の行です。ブロック引用符の内容 (initial `>` を除く) は、ブロックレベルの内容として解析されます。

```
> This is a block quote.
>
> 1. with a
> 2. list in it.
```

Markdown と同様に、段落の最初の行の前を除き、ブロック引用内の通常の段落行からプレフィックスを「遅延」省略することができます。

```
> This is a block
quote.
```

### リスト項目

リスト項目は、リスト マーカー、スペース (または改行)、およびリスト マーカーに対してインデントされた 1 つ以上の行で構成されます。例えば：

```
1.  This is a
 list item.

 > containing a block quote
```

段落の最初の行に続く段落行では、インデントが「遅れて」省略される場合があります。

```
1.  This is a
list item.

  Second paragraph under the
list item.
```

次の基本的なタイプのリスト マーカーを使用できます。

| マーカー | リストタイプ |
| -------- | ------------ |
| -           | bulett             |
| +           | bulett            |
| *           | bulett            |
| 1.          | 順序付き、10 進数の列挙、ピリオドが続く |
| 1)          | 順序付き、10 進数の列挙、その後に括弧が続く |
| (1)          | 順序付き、10 進数列挙、括弧で囲む |
| a.          | 順序付き、下位アルファベットで列挙され、その後にピリオドが続きます |
| a)          | 順序付けされ、下位アルファで列挙され、その後に括弧が続きます |
| (a)          | 順序付け、下位アルファ列挙、括弧で囲む |
| A.          | 順序付き、上位アルファベットで列挙され、その後にピリオドが続きます |
| A)          | 順序付け、上位アルファ列挙、その後に括弧が続く |
| (A)          | 順序付け、上位アルファ列挙、括弧で囲む |
| i.          | 順序付き、小ローマ字列挙、その後にピリオドが続く |
| i)          | 順序付き、小ローマ字列挙、その後に括弧が続く |
| (i)          | 順序付き、小ローマ字列挙、括弧で囲む |
| I.          | 順序付き、大ローマ字列挙、ピリオドが続く |
| I)          | 順序付き、大ローマ字列挙、その後に括弧が続く |
| (I)          | 順序付き、大ローマ字列挙、括弧で囲む |
| :            | 意味    |
| - [ ]        | タスク     |

順序付きリスト マーカーは、一連の任意の番号を使用できます。したがって、`(xix)`および `v)`は、どちらも有効な下位ローマ字列挙マーカーであり、有効な下位アルファ列挙マーカーでも`v)`あります。

### タスクリスト項目

`[ ]`、`[X]`、または`[x]`その後にスペースが続く箇条書きリスト項目は、チェックされていない (`[ ]`) かチェックされている (`[X]`または`[x]`) かのタスク リスト項目です。

### 定義リスト項目

定義リスト項目では、マーカーの後の最初の行がインライン コンテンツとして解析され、定義された用語:とみなされます。項目に含まれるそれ以上のブロックは定義とみなされます。

```
: orange

  A citrus fruit.
```

### リスト

リストは、単に同じタイプのリスト項目のシーケンスです (上の表の各行がタイプを定義します)。順序付きリストのスタイルまたは箇条書きを変更すると、1 つのリストが停止し、新しいリストが開始されることに注意してください。したがって、次のリスト項目は 4 つの異なるリストにグループ化されます。

```
i) one
i. one (style change)
+ bullet
* bullet (style change)
```

リスト項目のタイプがあいまいな場合があります。この場合、可能であればリストを継続する方法で曖昧さが解決されます。たとえば、

```
i. item
j. next item
```

最初の項目は、下位ローマ字列挙と下位アルファベット列挙の間で曖昧です。ただし、次の項目では後者の解釈のみが機能するため、2 つの別々のリストよりも 1 つの連続したリストを作成できる読み方を好みます。

順序付きリストの開始番号は、その最初の項目の番号によって決まります。後続の項目の数は関係ありません。

```
5) five
8) six
```

項目間または項目内のブロック間に空白行が含まれていないリストは、タイトとして分類されます。リストの先頭または末尾にある空白行は、厳密さの対象にはなりません。

```
- one
- two

  - sub
  - sub
```

厳密でないリストは緩いものです。この区別の意図された重要性は、項目間のスペースを減らしてタイトなリストを表示する必要があることです。

```
- one

- two
```

### コードブロック

コード ブロックは 3 つ以上の連続したバックティックの行で始まり、オプションで言語指定子が後に続きますが、それ以外は何もありません。(オプションで、言語指定子の前および`/`または後に空白を付けることができます。) コード ブロックは、開始バッククォート「フェンス」と同じかそれ以上の長さのバッククォートの行で終了します。そうでない場合は、ドキュメントまたは囲むブロックの終わりで終了します。そのような行に遭遇します。その内容はそのままのテキストとして解釈されます。コンテンツにバッククォートの行が含まれている場合は、「フェンス」として使用するバッククォートのより長い文字列を必ず選択してください。

```
````
This is how you do a code block:

``` ruby
x = 5 * 6
```
````
```

以下は、親コンテナが閉じられたときに暗黙的に閉じられるコード ブロックの例です。

```
> ```
> code in a
> block quote
```

Paragraph.
```

### 水平線

3 つ以上の`*`or`-`文字を含み、他に何も (スペースやタブを除く) 処理されない行は、テーマ区切り ( `<hr>`HTML の場合) です。Markdown とは異なり、テーマの区切りはインデントされる場合があります。

```
Then they went to sleep.

      * * * *

When they woke up, ...
```

### Raw ブロック

言語仕様が通常配置されるコード ブロックは、`=FORMAT`はRawコンテンツとして解釈されFORMAT、その形式で出力するためにそのまま渡されます。例えば：

```
``` =html
<video width="320" height="240" controls>
  <source src="movie.mp4" type="video/mp4">
  <source src="movie.ogg" type="video/ogg">
  Your browser does not support the video tag.
</video>
```
```

### div

div は 3 つ以上の連続するコロンの行で始まり、必要に応じて空白とクラス名が続きます (ただし、それ以外は何もありません)。これは、少なくとも開始フェンスと同じ長さの連続したコロンの行で終わるか、文書または包含ブロックの終わりで終わります。

div の内容はブロックレベルの内容として解釈されます。

```
::: warning
Here is a paragraph.

And here is another.
:::
```

### パイプテーブル

パイプ テーブルは一連の行で構成されます。各行はパイプ文字 (`|`) で始まりパイプ文字で終わり、パイプ文字`|`で区切られた1 つ以上のセルが含まれます。

```
| 1 | 2 |
```

区切り線 `-` は、すべてのセルが 1 つ以上の文字のシーケンスで構成されている行で、オプションで接頭辞または接尾辞が付けられます :。

区切り線が見つかると、前の行がヘッダーとして扱われ、その行と後続の行の配置は区切り線によって決まります (新しいヘッダーが見つかるまで)。区切り線自体は、解析されたテーブルに行を追加しません。

```
| fruit  | price |
|--------|------:|
| apple  |     4 |
| banana |    10 |
```

列の配置は、次のように区切り線によって決まります。

- 行が<code>-</code>がで始まり<code>:</code>で終わらない場合、列は左揃えになります。
- <code>:</code>で終わり、1で始まらない場合、列は右揃えになります。
- 始まりと終わりの両方が<code>:</code>である場合:、列は中央揃えになります。
- <code>:</code>で始まらない、終わらない場合:、列はデフォルトで揃えられます。

以下に例を示します。

```
| a  |  b |
|----|:--:|
| 1  | 2  |
|:---|---:|
| 3  | 4  |
```

aとbはヘッダーで、左の列はデフォルトで揃えられ、右の列は中央揃えで配置されます。1と2を含む次の実際の2行もヘッダーであり、左の列は左揃え、右の列は右揃えになります。この配置は3と4を含む後続の行にも適用されます4。

テーブルにはヘッダーが必要ありません。区切り線を省略するか、(列の配置を指定する必要がある場合)区切り線から始めます。

```
|:--|---:|
| x | 2  |
```

テーブルのセルの内容はインラインとして解析されます。パイプ テーブル セルではブロック レベルのコンテンツを使用できません。

Djot は、バックスラッシュでエスケープされたパイプや逐語的なスパン内のパイプを認識できるほど賢いです。これらはセル区切り文字としてカウントされません。

```
| just two \| `|` | cells in this table |
```

次の構文を使用して表にキャプションを追加できます。

```
^ This is the caption.  It can contain _inline formatting_
  and can extend over multiple lines, provided they are
  indented relative to the `^`.
```

キャプションは表の直後に置くことも、間に空行を入れることもできます。

### 参照リンクの定義

参照リンクの定義は、角括弧で囲まれた参照ラベル、コロン、空白 (または改行) と URL で構成されます。URL は複数の行に分割される場合があります (その場合、行は連結され、先頭または末尾のスペースは削除されます)。URL のチャンクには内部空白を含めることはできません。

```
[google]: https://google.com

[product page]: http://example.com?item=983459873087120394870128370
  0981234098123048172304
```

参照ラベルでは大文字と小文字の正規化は行われません。`[Link]`として定義された参照は、Markdown のように`[link]`として使用できません。

参照定義の属性はリンクに転送されます。

```
{title=foo}
[ref]: /url

[ref][]
```

/urlテキスト「ref」、URL 、タイトル「foo」のリンクが生成されます。ただし、同じ属性がリンクと参照定義の両方で定義されている場合は、リンク上の属性が参照定義の属性をオーバーライドします。

```
{title=foo}
[ref]: /url

[ref][]{title=bar}
```

ここでは「bar」というタイトルのリンクを取得します。

### 脚注

脚注は、脚注参照、コロン、その後に注の内容が続き、参照が開始される列を超えて任意の列にインデントされて構成されます。メモの内容はブロックレベルのコンテンツとして解析されます。

```
Here's the reference.[^foo]

[^foo]: This is a note
  with two paragraphs.

  Second paragraph.

  > a block quote in the note.
```

ブロック引用符やリスト項目と同様に、段落の後続の行ではインデントを「遅延」省略できます。

```
Here's the reference.[^foo]

[^foo]: This is a note
with two paragraphs.

  Second paragraph must
be indented, at least in the first line.
```

### ブロック属性

ブロック・レベル要素に属性を付けるには、ブロックの直前の行に属性を付けます。ブロック属性はインライン属性と同じ構文 インライン属性と同じ構文ですが、1行に収まらない場合は、それ以降の行をインデントする必要があります。繰り返し属性指定子を使うことができ、属性は累積されます。

```
{#water}
{.important .large}
Don't forget to turn off the water!

{source="Iliad"}
> Sing, muse, of the wrath of Achilles
```

### 見出しへのリンク

識別子は、明示的な識別子が付加されていない見出しに自動的に追加されます。識別子は、見出しのテキスト内容を取得し、句読点 (`-`と `_` 以外) を削除し、スペースを `-` に置き換え、一意性のために必要な場合は数値接尾辞を追加することによって形成されます。

```
## My heading + auto-identifier
```

ただし、暗黙的なリンク参照がすべての見出しに対して作成されるため、ほとんどの場合、見出しに割り当てられた識別子を知る必要はありません。したがって、同じ文書内の「Epilogue」というタイトルの見出しにリンクするには、参照リンクを使用するだけです。

```
See the [Epilogue][].

    * * * *

# Epilogue
```

### ネストの制限

準拠した実装では、スタック オーバーフローやその他の問題を回避するために、ネストに合理的な制限を課すことができます。たとえば、12 レベルを超えるネストを必要とする現実的なドキュメントはほとんどないため、512 という制限は完全に安全です。
