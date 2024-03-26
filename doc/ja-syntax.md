# Djot構文リファレンス

## インライン構文

### 優先順位

ほとんどのインライン構文は、インラインコンテンツを囲む前後の区切り文字によって定義され、強調表示またはリンクテキストなどとして定義されます。インラインコンテナの「優先順位」を決定する基本原則は、最初に閉じられたオープナーが優先されるということです。コンテナは重複できないため、オープナーが閉じられると、オープナーとクローザーの間の潜在的なオープナーは通常のテキストとしてマークされ、インライン構文を開くことができなくなります。たとえば、

    _This is *regular_ not strong* emphasis

```html
<p><em>This is *regular</em> not strong* emphasis</p>
```

最初の強調 `_` によって開かれた通常の強調は2番目の `_` によって閉じられ、その時点で強い強調 `*` は強い強調を開始する資格を失います。

    *This is _strong* not regular_ emphasis

```html
<p><strong>This is _strong</strong> not regular_ emphasis</p>
```

逆もまた同じです。

同様に、

    [Link *](url)*

```html
<p><a href="url">Link *</a>*</p>
```

はリンクの処理だけ行われます。

    *Emphasis [*](url)

```html
<p><strong>Emphasis [</strong>](url)</p>
```

リンクは無効です（強い強調が区切り文字 `[` を超えて終了するため）。

重複するコンテナは除外されますが、 *ネスト（入れ子に）* されたコンテナは問題ありません。

    _This is *strong within* regular emphasis_

```html
<p><em>This is <strong>strong within</strong> regular emphasis</em></p>
```

曖昧な場合は、区切り文字をオープナー `{` またはクローザー `}` としてマークに使用される場合があります。したがって、 `{_` は `_` のように動作しますが、強調を開くことしかできません。一方 `_}` は `_` のように動作しますが強調を閉じることしかできません。

    {_Emphasized_}
    _}not emphasized{_

```html
<p><em>Emphasized</em>
_}not emphasized{_</p>
```

明示的にマークされたクローザーは明示的にマークされたオープナーとのみ一致し、マークされていないクローザーはマークされていないオープナーのみと一致します（したがって、たとえば `{_hi_)` は強調を生成しません）。

特定のクローザーと一致する可能性のあるオープナーが複数ある場合は、最も近いものが使用されます。例えば：

    *not strong *strong*

```html
<p>*not strong <strong>strong</strong></p>
```

### 通常テキスト

特別な意味が与えられていないものはすべてリテラルなテキストとして解析されます。

すべてのASCII句読点文字（djotで特別な意味を持たないものであっても）はバックスラッシュでエスケープできます。したがって、`\*` はリテラル文字 `*` が含まれます。ASCII句読点文字以外の文字の前のバックスラッシュは、次の例外を除き、リテラルのバックスラッシュとして扱われます。

-   改行の前（または改行に続く空白またはタブの前）のバックスラッシュは、*ハード改行*として解析されます。この場合、バックスラッシュの前の空白とタブ文字は無視されます。
-   空白の前のバックスラッシュは*非改行空白*として解析されます。

### リンク

リンクには、*インライン*リンクと*参照*リンクの2種類があります。

どちらの種類も `[ … ]` の内側でリンクテキスト（任意のインライン形式が含まれる場合があります）から始まります。

*インラインリンク*には、括弧内にリンク先 `(URL)` が含まれます。リンクテキスト末尾の `]` とリンク先を定義するオープナー `(` の間に空白を入れてはいけません。

    [My link text](http://example.com)

```html
<p><a href="http://example.com">My link text</a></p>
```

URLは複数行に分かれている場合があります。この場合改行が行われ先頭と末尾の空白は無視され行が連結されます。

    [My link text](http://example.com?product_number=234234234234
    234234234234)

```html
<p><a href="http://example.com?product_number=234234234234234234234234">My link text</a></p>
```

*参照リンク*では括弧内の宛先ではなく角括弧内の参照ラベルを使用します。これはリンクテキストの直後に置く必要があります。

    [My link text][foo bar]

    [foo bar]: http://example.com

```html
<p><a href="http://example.com">My link text</a></p>
```

参照ラベルはドキュメント内のどこかで定義する必要があります。以下の[参照リンクの定義](#reference-link-definition)を参照してください。ただしリンクの解析は「ローカル」でありラベルが定義されているかどうかには依存しません。

    [foo][bar]

```html
<p><a>foo</a></p>
```

ラベルがない場合、リンクテキストはリンクテキストだけでなく参照ラベルとしても解釈されます:

    [My link text][]

    [My link text]: /url

```html
<p><a href="/url">My link text</a></p>
```

### 画像

画像はリンクと同じように機能しますが、接頭辞 `!` が付いています。リンクと同様にインラインバリアントと参照バリアントの両方が可能です。

    ![picture of a cat](cat.jpg)
    
    ![picture of a cat][cat]
    
    ![cat][]
    
    [cat]: feline.jpg

```html
<p><img alt="picture of a cat" src="cat.jpg"></p>
<p><img alt="picture of a cat" src="feline.jpg"></p>
<p><img alt="cat" src="feline.jpg"></p>
```

### 自動リンク

`<` ... `>` で囲まれたURLまたはメールアドレスはハイパーリンクになります。中括弧間の内容は文字通りに扱われます（バックスラッシュエスケープは使用できません）。

    <https://pandoc.org/lua-filters>
    <me@example.com>

```html
<p><a href="https://pandoc.org/lua-filters">https://pandoc.org/lua-filters</a>
<a href="mailto:me@example.com">me@example.com</a></p>
```

URLまたはメールアドレスには改行を含めることはできません。

### Verbatim

Verbatimコンテンツは、連続するバックティック文字（`` ` ``）の文字列で始まり、同じ数が連続するバックティック文字の文字列で終わります。

    ``Verbatim with a backtick` character``
    `Verbatim with three backticks  character`

```html
<p><code>Verbatim with a backtick` character</code>
<code>Verbatim with three backticks  character</code></p>
```

バックティック間の内容はVerbatimテキストとして扱われます（そこではバックスラッシュ エスケープは機能しません）。

コンテンツがバックティック文字で開始または終了する場合、開始または終了バックティックとコンテンツの間の1つの空白が削除されます。

    `` `foo` ``

```html
<p><code>`foo`</code></p>
```

インラインとして解析されるテキストが、終了バックティック文字列に遭遇する前に終了する場合、Verbatimテキストは最後まで続きます。

    `foo bar

```html
<p><code>foo bar</code></p>
```

### 強調と強い強調（Emphasis/strong）

強調されたインラインコンテンツは `_` で囲みます。強い強調は `*` で囲みます。

    _emphasized text_
    
    *strong emphasis*

```html
<p><em>emphasized text</em></p>
<p><strong>strong emphasis</strong></p>
```

また `_` は `*` 直後に空白が続かない場合にのみ強調を開始できます。強調を閉じることができるのは、直前に空白がない場合、および開始文字と終了文字の間に区切り文字以外の文字がある場合のみです。

    _ Not emphasized (spaces). _
    
    ___ (not an emphasized `_` character)

```html
<p>_ Not emphasized (spaces). _</p>
<p>___ (not an emphasized <code>_</code> character)</p>
```

強調はネスト（入れ子に）することができます:

    __emphasis inside_ emphasis_

```html
<p><em><em>emphasis inside</em> emphasis</em></p>
```

中括弧 `{` は `_` または `*` をオープナーまたはクローザーとして強制的に解釈するために使用できます。

    {_ this is emphasized, despite the spaces! _}

```html
<p><em> this is emphasized, despite the spaces! </em></p>
```

### ハイライト（mark）

`{=` と `=}` の間のインラインコンテンツは、強調表示されたテキストとして扱われます（HTMLでの `<mark>`）。 `{` と `}` は必須であることに注意してください。

    This is {=highlighted text=}.

```html
<p>This is <mark>highlighted text</mark>.</p>
```

### 上付きと下付き（Super/subscript）

上付き文字は `^` で囲みます、下付き文字は `~` です。

    H~2~O and djot^TM^

```html
<p>H<sub>2</sub>O and djot<sup>TM</sup></p>
```

中括弧を使用することもできますが、必須ではありません:

    H{~one two buckle my shoe~}O

```html
<p>H<sub>one two buckle my shoe</sub>O</p>
```

### 挿入と取り消し線（Insert/delete）

インラインテキストを挿入済みとしてマークするには、 `{+` と `+}` を使用します。削除済みとしてマークするには、 `{-` と `-}` を使用します。 `{` と `}` は必須です。

    My boss is {-mean-}{+nice+}.

```html
<p>My boss is <del>mean</del><ins>nice</ins>.</p>
```

### スマート句読点

直線の二重引用符（`"`）と一重引用符（`'`）は中引用符として解析されます。Djotは、文脈からどの方向の引用が必要かを判断するのが非常に上手です。

    "Hello," said the spider.
    "'Shelob' is my name."

```html
<p>&ldquo;Hello,&rdquo; said the spider.
&ldquo;&lsquo;Shelob&rsquo; is my name.&rdquo;</p>
```

ただし、そのヒューリスティックは中かっこを使用した引用符をオープナー `{"` またはクローザー `"}` としてマークすることでオーバーライドできます:

    '}Tis Socrates' season to be jolly!

```html
<p>&rsquo;Tis Socrates&rsquo; season to be jolly!</p>
```

直接引用符が必要な場合は、バックスラッシュとエスケープを使用します:

    5\'11\"

```html
<p>5'11"</p>
```

ピリオド3つのシーケンスは *ellipses* として解析されます。 `...`

ハイフン3つのシーケンスは *em-dash* として解析されます。 `—`

ハイフン2つのシーケンスは *en-dash* として解析されます。 `--`

    57--33 oxen---and no sheep...

```html
<p>57&ndash;33 oxen&mdash;and no sheep&hellip;</p>
```

より長いハイフンのシーケンスは、全角ダッシュ、半角ダッシュ、およびハイフンに分割されます。可能であれば均一に、どちらの方法でも均一性が達成できる場合は、全角ダッシュを使用することを推奨します。（つまり、4つのハイフンは2つの半角ダッシュになり、6つのハイフンは2つの半角ダッシュになります）。

    a----b c------d

```html
<p>a&ndash;&ndash;b c&mdash;&mdash;d</p>
```

### 数式

LaTeX数式を含めるには、その数式をVerbatimスパンに入れ、その先頭に `$` （インラインの場合）または `$$` （数式表示の場合）を付けます:

    Einstein derived $`e=mc^2`.
    Pythagoras proved
    $$` x^n + y^n = z^n `

```html
<p>Einstein derived <span class="math inline">\(e=mc^2\)</span>.
Pythagoras proved
<span class="math display">\[ x^n + y^n = z^n \]</span></p>
```

### 脚注参照

脚注参照は、 `[^参照ラベル]` 形式を用います。

    Here is the reference.[^foo]

    [^foo]: And here is the note.

```html
<p>Here is the reference.<a id="fnref1" href="#fn1" role="doc-noteref"><sup>1</sup></a></p>
<section role="doc-endnotes">
<hr>
<ol>
<li id="fn1">
<p>And here is the note.<a href="#fnref1" role="doc-backlink">↩︎︎</a></p>
</li>
</ol>
</section>
```

脚注自体の構文については、以下の[脚注](#footnote)を参照してください。

### 改行

インラインコンテンツ内の改行は「ソフト」改行として扱われます。これらは空白として、または（HTMLなど、改行が意味的に空白のように扱われるコンテキストでは）改行としてレンダリングされる場合があります。

ハード改行（HTMLでの `<br>`）を行うには、「バックスラッシュ + 改行」を使用します:

    This is a soft
    break and this is a hard\
    break.

```html
<p>This is a soft
break and this is a hard<br>
break.</p>
```

### コメント

属性内で `%` に挟まれた内容はコメントとして扱われます。これにより属性にコメントを追加できるようになります:

    {#ident % later we'll add a class %}

ただし、コメントを追加する一般的な方法としても機能します。コメントのみを含む属性指定子を使用してください:

    Foo bar {% This is a comment, spanning
    multiple lines %} baz.

```html
<p>Foo bar  baz.</p>
```

### 記号

単語を記号 `:` で囲むと「記号」が作成され、デフォルトでは文字通りに表示されるだけですが、フィルターによって特別に処理される場合があります。（たとえば、フィルターを使用して記号を絵文字に変換できます。ただし、これはdjotには組み込まれていません）。

    My reaction is :+1: :smiley:.

```html
<p>My reaction is 👍 😃.</p>
```

### Rawインライン

任意の形式のRawインラインコンテンツは、Verbatimスパンとそれに続く次の文字列を使用して含めることができます `{=FORMAT}` :

    This is `<?php echo 'Hello world!' ?>`{=html}.

```html
<p>This is <?php echo 'Hello world!' ?>.</p>
```

このコンテンツは、指定された形式をレンダリングするときにそのまま渡されるように意図されていますが、それ以外の場合は無視されます。

### スパン

リンクや画像ではなく、直後に属性が続く角括弧内のテキストは、一般的なスパンとして扱われます。

    It can be helpful to [read the manual]{.big .red}.

```html
<p>It can be helpful to <span class="big red">read the manual</span>.</p>
```

### インライン属性

属性は中括弧内に配置され、属性を付加したいインライン要素の *直後* に置く必要があります（空白は入れません）。

中括弧内では、次の構文を使用できます:

-   `.foo` は `foo` クラスとして指定します。この方法で複数のクラスを指定でき、それらは結合されます。
-   `#foo` は `foo` 識別子として指定します。要素は識別子を1つだけ持つことができます。複数の識別子が指定された場合は、最後の識別子が使用されます。
-   `key="value"` または、 `key=value` はkey-value属性を指定します。値がすべてASCII英数字で構成されている場合、 `_` 、 `:` 、 `-` で構成されている場合、引用符は必要ありません。バックスラッシュエスケープは引用符で囲まれた値の内部で使用できます。
-   `%` コメントが始まり、次の属性 `%` または属性の終わり（`}`）で終わります。

属性指定子には改行が含まれる場合があります。

例:

    An attribute on _emphasized text_{#foo
    .bar .baz key="my value"}

```html
<p>An attribute on <em class="bar baz" id="foo" key="my value">emphasized text</em></p>
```

属性指定子は「stacked」（積み重ねること）ができ、その場合は結合されます。したがって、

    avant{lang=fr}{.blue}

```html
<p><span class="blue" lang="fr">avant</span></p>
```

は、以下と同じです

    avant{lang=fr .blue}

```html
<p><span class="blue" lang="fr">avant</span></p>
```

## ブロック構文

commonmarkと同様に、ブロック構造はインライン解析の前に識別でき、インライン構造よりも優先されます。

実際、ブロックはバックトラックなしで行ごとに解析できます。行がブロックレベルの構造に与える影響は、将来の行に依存することはありません。

インデントは、リスト項目または脚注のネストの場合にのみ重要です。

ブロックレベルの項目は空白行で区切る必要があります。2つのブロックレベル要素が隣接する場合があります。たとえば、水平線またはフェンスで囲まれたコードブロックの直後に段落が続くことがあります。実際、行ごとに解析できる可能性があるため、ブロックレベル要素の後に空白行を入れる必要はありません。ただし、読みやすさを考慮して、ブロックレベルの要素は*常に*空行で区切ることをお勧めします。段落は他のブロックレベルの要素によって中断されることはなく、常に空行（または文書またはそれを含む要素の終わり）で終了する必要があります。

### 段落

段落は、他のブロックレベル要素の1つであるための条件を満たさない、一連の非空白行です。テキストコンテンツは、一連のインライン要素として解析されます。改行はソフト改行として扱われ、書式設定された出力では空白と同様に解釈されます。段落は空白行または文書の終わりで終わります。

### 見出し

見出しは1つ以上の `#` 文字シーケンスで始まり、その後に空白が続きます。`#` 文字の数によって見出しレベルは定義されます。次のテキストはインラインコンテンツとして解析されます。

    ## A level _two_ heading!

```html
<section id="A-level-two-heading">
<h2>A level <em>two</em> heading!</h2>
</section>
```

見出しテキストは後続の行にまたがる場合があり、その前に同じ数の `#` 文字が続く場合もあります（ただし、省略することもできます）。

見出しは、空行（または文書またはそれを囲んでいるコンテナの終わり）に達すると終了します。

    # A heading that
    # takes up
    # three lines

    A paragraph, finally

```html
<section id="A-heading-that-takes-up-three-lines">
<h1>A heading that
takes up
three lines</h1>
<p>A paragraph, finally</p>
</section>

# A heading that
takes up
three lines

A paragraph, finally.

<section id="A-heading-that-takes-up-three-lines">
<h1>A heading that
takes up
three lines</h1>
<p>A paragraph, finally.</p>
</section>
```

### 引用ブロック（Block quote）

引用ブロックは、各行が `>` で始まり、その後に空白または行末が続く一連の行です。引用ブロック内のコンテンツ（最初の `>` を除く）は、ブロックレベルのコンテンツとして解析されます。

    > This is a block quote.
    >
    > 1. with a
    > 2. list in it.

```html
<blockquote>
<p>This is a block quote.</p>
<ol>
<li>
with a
</li>
<li>
list in it.
</li>
</ol>
</blockquote>
```

Markdownと同様に、段落はじまり行の前を除き、引用ブロック内の通常段落行から `>` プレフィックスを省略することができます:

    > This is a block
    quote.

```html
<blockquote>
<p>This is a block
quote.</p>
</blockquote>
```

### リスト項目

リスト項目は、リストマーカー、空白（または改行）、リストマーカーに対してインデントされた1つ以上の行で構成されます。例えば:

    1.  This is a
     list item.

     > containing a block quote

```html
<ol>
<li>
<p>This is a
list item.</p>
<blockquote>
<p>containing a block quote</p>
</blockquote>
</li>
</ol>
```

段落の開始行に続く段落行では、インデントが省略される場合があります:

    1.  This is a
    list item.

      Second paragraph under the
    list item.

```html
<ol>
<li>
<p>This is a
list item.</p>
<p>Second paragraph under the
list item.</p>
</li>
</ol>
```

次の基本的なタイプのリストマーカーを使用できます:

| マーカー        | リストの種類                      |
| ------------- | -------------------------- |
| `-`                | bullet                      |
| `+`                | bullet                      |
| `*`                | bullet                      |
| `1.`                | 順序付き、10進数、直後にピリオド                      |
| `1)`                | 順序付き、10進数、直後に終わり括弧                      |
| `(1)`                | 順序付き、10進数、括弧で挟む                      |
| `a.`                | 順序付き、アルファベット小文字、直後にピリオド                      |
| `a)`                | 順序付き、アルファベット小文字、直後に終わり括弧                      |
| `(a)`                | 順序付き、アルファベット小文字、括弧で挟む                      |
| `A.`                | 順序付き、アルファベット大文字、直後にピリオド                      |
| `A)`                | 順序付き、アルファベット大文字、直後に終わり括弧                      |
| `(A)`              | 順序付き、アルファベット大文字、括弧で挟む                      |
| `i.`                | 順序付き、ローマ数字小文字、直後にピリオド                      |
| `i)`                | 順序付き、ローマ数字小文字、直後に終わり括弧                      |
| `(i)`                | 順序付き、ローマ数字小文字、括弧で挟む                      |
| `I.`                | 順序付き、ローマ数字大文字、直後にピリオド                      |
| `I)`                | 順序付き、ローマ数字大文字、直後に終わり括弧                      |
| `(I)`                 | 順序付き、ローマ数字大文字、括弧で挟む                      |
| `:`                     | 定義語                            |
| `- [ ]`                 | タスク                            |

順序付きリストマーカーは、一連の任意の番号を使用できます: したがって `(xix)` および `v)` は、どちらも有効なローマ字小文字列挙マーカーであり、`v)` は有効なアルファベット小文字列挙マーカー *でも* あります。

#### タスクリスト項目

`[ ]` 、 `[X]` または `[x]` の後に空白で続く箇条書きリスト項目は、チェックされていない（`[ ]`）かチェックされている（`[X]` または `[x]`）かのタスクリスト項目です。

#### 定義リスト項目

定義リスト項目では、マーカー `:` の後の最初の行がインラインコンテンツとして解析され、定義された *用語* とみなされ、項目に含まれるそれ以上のブロックは *定義* とみなされます。

    : orange

      A citrus fruit.

```html
<dl>
<dt>orange</dt>
<dd>
<p>A citrus fruit.</p>
</dd>
</dl>
```

### リスト

リストは、単純に同じタイプのリスト項目シーケンスです（上の表の各行が種類を定義します）。順序付きリストのスタイルまたは箇条書きを変更すると、1つのリストが停止し、新しいリストが開始されることに注意してください。したがって、次のリスト項目は4つの異なるリストにグループ化されます:

    i) one
    i. one (style change)
    + bullet
    * bullet (style change)

```html
<ol start="9" type="a">
<li>
one
</li>
</ol>
<ol start="9" type="a">
<li>
one (style change)
</li>
</ol>
<ul>
<li>
bullet
</li>
</ul>
<ul>
<li>
bullet (style change)
</li>
</ul>
```

リスト項目の種類が曖昧な場合があります。この場合、可能であればリストを継続する方法で曖昧さが解決されます。たとえば、

    i. item
    j. next item

```html
<ol start="9" type="a">
<li>
item
</li>
<li>
next item
</li>
</ol>
```

最初の項目は、ローマ字小文字列挙とアルファベット小文字列挙の間で曖昧です。ただし、次の項目では後者の解釈のみが機能するため、2つの別々のリストよりも1つの連続したリストを作成できる読み方を好みます。

順序付きリストの開始番号は、その最初の項目の番号によって決まります。後続の項目の数は関係ありません。

    5) five
    8) six

```html
<ol start="5">
<li>
five
</li>
<li>
six
</li>
</ol>
```

項目間または項目内のブロック間に空白行が含まれていないリストは、 *厳密に* （tightとして）分類されます。リストの先頭または末尾にある空白行は、厳密さの対象にはなりません。

    - one
    - two

      - sub
      - sub

```html
<ul>
<li>
one
</li>
<li>
two
<ul>
<li>
sub
</li>
<li>
sub
</li>
</ul>
</li>
</ul>
```

厳密でないリストは *緩い* （looseな）リストです。この区別の意図された重要性は、項目間の空白を減らして厳密（タイト）なリストを表示する必要があることです。

    - one

    - two

```html
<ul>
<li>
<p>one</p>
</li>
<li>
<p>two</p>
</li>
</ul>
```

### コードブロック

コードブロックは3つ以上の連続したバックティックの行で始まります。オプションで言語指定子が付与される場合もありますが、それだけです（オプションで、言語指定子の前および後に空白を付けることができます）。
コードブロックは、バックティック「フェンス」の開始と同じか、それ以上の数のバックティック行で終了、そのような行がない場合は、文書または囲んでいるブロックの終了で終わります。
コンテンツはverbatimテキストとして解釈されます。
コンテンツにバックティック行が含まれている場合は、「フェンス」として用いたバックティックより多い数を指定してください:

    This is how you do a code block:

     ruby
    x = 5 * 6

```html
<pre><code>This is how you do a code block:

 ruby
x = 5 * 6
</code></pre>
```

以下は、親コンテナが閉じられたときに暗黙的に閉じられるコードブロックの例です;

    > 
    > code in a
    > block quote

    Paragraph.

```html
<blockquote>
<pre><code>code in a
block quote
</code></pre>
</blockquote>
<p>Paragraph.</p>
```

### 水平線（hr）

3つ以上の `*` か `-` を含み、他に何も（空白やタブを除く）処理されない行は、水平線（HTMLでの `<hr>`）です。Markdownとは異なり、水平線（テーマの区切り）はインデントされる場合があります:

    Then they went to sleep.

          * * * *

    When they woke up, ...

```html
<p>Then they went to sleep.</p>
<hr>
<p>When they woke up, &hellip;</p>
```

### Raw ブロック

`=FORMAT` と記述されたコードブロックは、通常言語仕様が記述されるところですが、 `FORMAT` のRawコンテンツとして解釈され、そのフォーマットで出力するためにそのまま渡されます。例えば:

     =html
    <video width="320" height="240" controls>
      <source src="movie.mp4" type="video/mp4">
      <source src="movie.ogg" type="video/ogg">
      Your browser does not support the video tag.
    </video>
    

```html
<video width="320" height="240" controls>
  <source src="movie.mp4" type="video/mp4">
  <source src="movie.ogg" type="video/ogg">
  Your browser does not support the video tag.
</video>
```

### Div

divは3つ以上の連続するコロン（`:::`）の行で始まり、必要に応じて空白とクラス名が続きます（ただし、これだけです）。これは、少なくともフェンスの開始と同じ数の連続したコロンの行で終わるか、文書またはブロックコンテナの終わりで終わります。

divのコンテンツはブロックレベルコンテンツとして解釈されます。

    ::: warning
    Here is a paragraph.

    And here is another.
    :::

```html
<div class="warning">
<p>Here is a paragraph.</p>
<p>And here is another.</p>
</div>
```

### パイプテーブル

パイプテーブルは一連の *行* で構成されます。各行はパイプ文字（`|`）で始まりパイプ文字で終わり、パイプ文字（`|`）で区切られた1つ以上の *セル* が含まれます:

    | 1 | 2 |

```html
<table>
<tr>
<td>1</td>
<td>2</td>
</tr>
</table>
```

*区切り線* `-` は、すべてのセルが1つ以上の文字のシーケンスで構成されている行で、オプションで接頭辞または接尾辞に `:` が付けられます。

区切り線が見つかると、前の行がヘッダーとして扱われ、その行と後続の行の配置は区切り線によって決まります（新しいヘッダーが見つかるまで）。区切り線自体は、解析されたテーブルに行を追加しません。

    | fruit  | price |
    |--------|------:|
    | apple  |     4 |
    | banana |    10 |

```html
<table>
<tr>
<th>fruit</th>
<th style="text-align: right;">price</th>
</tr>
<tr>
<td>apple</td>
<td style="text-align: right;">4</td>
</tr>
<tr>
<td>banana</td>
<td style="text-align: right;">10</td>
</tr>
</table>
```

列の配置は、次のように区切り線によって決まります:

-   行が `-` がで始まり `:` で終わらない場合、列は左揃えになります
-   `:` で終わり、 `:` で始まらない場合、列は右揃えになります
-   始まりと終わりの両方が `:` である場合、列は中央揃えになります
-   `:`     で始まらず、終わらない場合、列はデフォルトで揃えられます（均一化）

以下に例を示します:

    | a  |  b |
    |----|:--:|
    | 1  | 2  |
    |:---|---:|
    | 3  | 4  |

```html
<table>
<tr>
<th>a</th>
<th style="text-align: center;">b</th>
</tr>
<tr>
<th style="text-align: left;">1</th>
<th style="text-align: right;">2</th>
</tr>
<tr>
<td style="text-align: left;">3</td>
<td style="text-align: right;">4</td>
</tr>
</table>
```

`a` と `b` はヘッダーで、左の列はデフォルトで揃えられ、右の列は中央揃えで配置されます。 `1` と `2` を含む次の実際の2行もヘッダーであり、左の列は左揃え、右の列は右揃えになります。この配置は `3` と `4` を含む後続の行にも適用されます。

テーブルにヘッダーが不要な場合:  区切り線を省略するか、（列の配置を指定する必要がある場合)区切り線から *開始* します:

    |:--|---:|
    | x | 2  |

```html
<table>
<tr>
<td style="text-align: left;">x</td>
<td style="text-align: right;">2</td>
</tr>
</table>
```

テーブルのセル内コンテンツはインラインとして解析されます。パイプテーブルのセル内ではブロックレベルのコンテンツを使用できません。

Djotは、バックスラッシュでエスケープされたパイプやverbatimスパン内のパイプを認識できるほど賢いです。これらはセルの区切り文字としてカウントされません:

    | just two \| `|` | cells in this table |

```html
<table>
<tr>
<td>just two | <code>|</code></td>
<td>cells in this table</td>
</tr>
</table>
```

次の構文を使用して表にキャプションを追加できます:

    ^ This is the caption.  It can contain _inline formatting_
      and can extend over multiple lines, provided they are
      indented relative to the `^`.

キャプションは表の直後に置くことも、間に空行を入れることもできます。

### 参照リンクの定義

参照リンクの定義は、角括弧で囲まれた参照ラベル、コロン、空白（または改行）とURLで構成されます。URLは複数の行に分割される場合があります（その場合、行は連結され先頭または末尾の空白は削除されます）。URLのチャンク内では空白を含めることはできません。

    [google]: https://google.com

    [product page]: http://example.com?item=983459873087120394870128370
      0981234098123048172304

参照ラベルでは大文字と小文字の正規化は行われません。 `[link]` として定義された参照は、Markdownのように `[Link]` として使用できません。

参照定義の属性はリンクに転送されます:

    {title=foo}
    [ref]: /url

    [ref][]

```html
<p><a href="/url" title="foo">ref</a></p>
```

URLは `/url` 、テキストリンクは「ref」、タイトルは「foo」として生成されます。ただし、同じ属性がリンクと参照定義の両方で定義されている場合は、リンク上の属性が参照定義の属性をオーバーライドします。

    {title=foo}
    [ref]: /url

    [ref][]{title=bar}

```html
<p><a href="/url" title="bar">ref</a></p>
```

上記は「bar」というタイトルのリンクを取得します。

### 脚注

脚注は、脚注参照、コロン、その後にメモ内容が続き、参照が開始される列を超えて任意の列にインデントされて構成されます。メモの内容はブロックレベルのコンテンツとして解析されます。

    Here's the reference.[^foo]

    [^foo]: This is a note
      with two paragraphs.

      Second paragraph.

      > a block quote in the note.

```html
<p>Here&rsquo;s the reference.<a id="fnref1" href="#fn1" role="doc-noteref"><sup>1</sup></a></p>
<section role="doc-endnotes">
<hr>
<ol>
<li id="fn1">
<p>This is a note
with two paragraphs.</p>
<p>Second paragraph.</p>
<blockquote>
<p>a block quote in the note.</p>
</blockquote>
<p><a href="#fnref1" role="doc-backlink">↩︎︎</a></p>
</li>
</ol>
</section>
```

引用ブロックやリスト項目と同様に、段落の後続行ではインデントを「lazily」省略できます:

    Here's the reference.[^foo]

    [^foo]: This is a note
    with two paragraphs.

      Second paragraph must
    be indented, at least in the first line.

```html
<p>Here&rsquo;s the reference.<a id="fnref1" href="#fn1" role="doc-noteref"><sup>1</sup></a></p>
<section role="doc-endnotes">
<hr>
<ol>
<li id="fn1">
<p>This is a note
with two paragraphs.</p>
<p>Second paragraph must
be indented, at least in the first line.<a href="#fnref1" role="doc-backlink">↩︎︎</a></p>
</li>
</ol>
</section>
```

### ブロックの属性

ブロックレベル要素に属性を付けるには、ブロックの直前の行に属性を付けます。ブロック属性はインライン属性と同じ構文ですが、1行に収まらない場合は、それ以降の行をインデントする必要があります。繰り返し属性指定子を使うことができ、属性は累積されます。

    {#water}
    {.important .large}
    Don't forget to turn off the water!

    {source="Iliad"}
    > Sing, muse, of the wrath of Achilles

```html
<p class="important large" id="water">Don&rsquo;t forget to turn off the water!</p>
<blockquote source="Iliad">
<p>Sing, muse, of the wrath of Achilles</p>
</blockquote>
```

### 見出しへのリンク

識別子は、明示的な識別子が付加されていない見出しに自動的に追加されます。識別子は、見出しのテキスト内容を取得し、句読点（`_` と `-` 以外）を削除し、空白を `-` に置き換え、一意性を保つために必要な場合は数値接尾辞を追加することによって形成されます。

    ## My heading + auto-identifier

```html
<section id="My-heading-auto-identifier">
<h2>My heading + auto-identifier</h2>
</section>
```

ただし、暗黙的なリンク参照がすべての見出しに対して作成されるため、ほとんどの場合、見出しに割り当てられた識別子を知る必要はありません。したがって、同じ文書内の「Epilogue」というタイトルの見出しにリンクするには、参照リンクを使用するだけです:

    See the [Epilogue][].

        * * * *

    # Epilogue

```html
<p>See the <a href="#Epilogue">Epilogue</a>.</p>
<hr>
<section id="Epilogue">
<h1>Epilogue</h1>
</section>
```

## ネストの制限

準拠した実装では、スタックのオーバーフローやその他の問題を回避するために、ネストに合理的な制限を課すことができます。たとえば、12レベルを超えるネストを必要とする現実的なドキュメントはほとんどないため、512という制限は完全に安全です。

