# Djot

Djotは軽量なマークアップ構文です。この機能のほとんどは[commonmark](https://spec.commonmark.org)から派生していますが、commonmarkにおいて構文が複雑で効率的な解析が難しい点をいくつか修正しています。また定義リスト、脚注、表、いくつかの新しい種類のインライン形式（挿入/削除、ハイライト、上付き/下付き）、数式、スマート句読点、どの要素にも適用できる属性、ブロックレベル、インラインレベル、未加工コンテンツ用の汎用コンテナをサポートしており、commonmarkよりもはるかに機能が充実しています。

このプロジェクトは、私（[jgm](https://johnmacfarlane.net/)）がエッセイ「[Beyond Markdown](https://johnmacfarlane.net/beyond-markdown.html)」で提案したアイデアのいくつかを実装する試みとして始まりました。(以下の「[理論的根拠](#理論的根拠)」を参照してください。)

このリポジトリには、[構文の説明](https://htmlpreview.github.io/?https://github.com/dai/djot/blob/master/doc/ja-syntax.html)、[チートシート](https://github.com/dai/djot/blob/main/doc/ja-cheatsheet.md)、およびdjotとMarkdownの主な違いの概要を説明するMarkdownユーザー向けの[クイックスタートガイド](https://github.com/dai/djot/blob/main/doc/ja-quick-start-for-markdown-users.md)が含まれています。

ローカルにインストールしなくても、[djotプレイグラウンド](https://djot.net/playground/)でdjotを試すことができます。

## 理論的根拠

設計目標は次のとおりです

1. djotマークアップはバックトラッキングなしで線形時間にて解析できる必要があります。

2. インライン要素の解析は「ローカル」である必要があり、後方で定義される参照に依存しません。これはcommonmarkには当てはまらないことです:  `[foo][bar]`で「[foo]」の後はリンクを含むテキスト「bar」または「[foo][bar]」、あるいはテキスト「foo」を含むリンク、「foo」の後に「[bar]」が続く場合があります。これは、参照`[foo]`と`[bar]`がドキュメント内の別の場所（おそらく後方）で定義されているかどうかに応じて異なります。この*非局所性*により、正確な構文の強調表示はほぼ不可能になります。

3. 強調のルールはもっとシンプルであるべきです。commonmarkでは二重文字が強い強調に使用されているため多くの潜在的な曖昧さが生じますが、これらは17個のルールの気の遠くなるようなリストによって解決されます。これらのルールの優れたメンタルモデルを形成するのは困難です。ほとんどの場合、彼らは人間が最も自然に解釈する方法で物事を解釈しますが ---常にそうとは限りません。

4. 表現上の盲点は避けるべきです。commonmarkでは、`a*?*b` と書くとフランキングルール [^1] によって最初のアスタリスクが右フランキングとして分類されるため、HTML `a<em>?</em>b` を生成したい場合、運が悪かったとしか言いようがありません。これを回避する方法はありますが醜いです（ `a` の代わりに数値参照を使用します。）。djotでは、この種の表現上の死角があってはなりません。

[^1]: いわゆる*分かち書き*。左フランキングで開いて右フランキングで閉じてはじめて強調されるというルールです。

5. どのコンテンツがリスト項目に属するかのルールは単純である必要があります。commonmarkでは、リスト項目の下のコンテンツは、リストマーカーの後の最初の空白以外のコンテンツまでインデントする必要があります（リスト項目がインデントされたコードで始まる場合は、マーカーの後ろに5つの空白）。インデントされたコンテンツが十分にインデントされておらず、リスト項目に含まれない場合、多くの人が混乱します。

6. パーサーは、Unicode文字クラス、HTMLタグ、またはエンティティを認識したり、Unicodeの大文字と小文字のCase Foldingを実行したりすることを強制されるべきではありません。これにより、さらに複雑さが増します。

7. 構文はハード改行に適している必要があります。段落をハード改行することにより、異なる解釈が生じてはなりません（たとえば、ピリオドが続く数字が行頭になる場合など）。（多くの人が、なぜハード改行するのか疑問に思うと思います。  
  答え: HTMLへの変換や、長い行をソフト改行する特別なエディターを使用せずに、文書をそのまま読めるようにするためです。ソースの可読性は1つであったことを思い出してください。MarkdownとCommonmarkの主な目的の一部です）。

8. 構文は次の意味で均一に構成されている必要があります。つまり、一連の行がリスト項目またはブロック引用符の外側で特定の意味を持つ場合、その内側でも同じ意味を持つ必要があります。この原則は[commonmark spec](https://spec.commonmark.org/0.30/#principle-of-uniformity)で明確に示されていますが、この仕様は完全に準拠しているわけではありません（[commonmark/commonmark-spec#634](https://github.com/commonmark/commonmark-spec/issues/634)を参照）。

9. 任意の属性を任意の要素に付加できる必要があります。

10. テキスト、インラインコンテンツ、およびブロックレベルのコンテンツには、任意の属性を適用できる汎用コンテナが必要です。これによりAST変換を使用した拡張性が可能になります。

11. これらの目標に合わせて構文は可能な限り単純にする必要があります。したがってたとえば、2つの異なるスタイルの見出しやコードブロックは必要ありません。

* * *

これらの目標が次の決定の動機となりました。

- 目標7のため、ブロックレベルの要素は段落（または見出し）を中断することはできません。したがってdjotにおいて以下は1つの段落であり、（commonmarkが考えるように）段落の後に順序付きリストが続き、その後にブロック引用符が続き、その後にセクションの見出しが続くということではありません:

```
My favorite number is probably the number
1. It's the smallest natural number that is
> 0. With pencils, though, I prefer a
# 2.
```
Commonmarkでは、`1.` 以外のマーカーで始まるリストが段落を中断することを禁止するなど、目標7に対していくらか譲歩しています。しかしこれは構文の規則性と予測可能性を犠牲にした妥協です。一般的なルールを決めたほうが良いでしょう。

最後の決定の意味は、「タイトな」リストは引き続き可能ですが（項目間に空行を入れない）、サブリストの前には常に空行を置く必要があるということです。したがって代わりに

```
- Fruits
  - apple
  - orange
```

は以下のように書かなければなりません。

```
- Fruits

  - apple
  - orange
```

（この空行は「タイトさ」にはカウントされません）。reStructuredTextも同じ設計上の決定を行います。

- また、目標7を促進するために、見出しが複数行にわたる「怠惰さ」を許可します。

```
 ## My excessively long section heading is too
long to fit on one line.
```

作業中に、setext-style（下線付き）の見出しを削除して簡素化します。実際には2つの見出し構文は必要ありません（目標11）。

- 目標5を達成するために、非常に単純なルールがあります:  リストマーカーの先頭を超えてインデントされているものはすべてリスト項目に属します。

```
1. list item

  > block quote inside item 1

2. second item
```

commonmarkでは、ブロック引用符が十分にインデントされていないため、これはブロック引用符で囲まれた2つの別個のリストとして解析されます。commonmarkでこの単純なルールを使用できなかったのは、インデントされたコードブロックが要因でした。リスト項目にインデントされたコードブロックが含まれる場合、どの列でインデントのカウントを開始するかを知る必要があるため、リストの見栄えを最も良くする列（マーカーの後の空白以外のコンテンツの最初の列）に固定しました。

```
1.  A commonmark list item with an indented code block in it.

        code!
```

djotでは、インデントされたコードブロックを削除するだけです。いずれにせよほとんどの人はフェンスで囲まれたコードブロックを好みます。コードブロックを記述する2つの異なる方法は必要ありません（目標11）。

- 目標6を達成しRaw HTMLを処理するために採用されている複雑なルールを回避するために、明示的にマークされたコンテキストを除きRaw HTMLを許可しません。
例:  
`` `<a id="foo">`{=html} `` あるいは

````
``` =html
<table>
<tr><td>foo</td></tr>
</table>
```
````

Markdownとは異なりdjotはHTML中心ではありません。Djotドキュメントはさまざまな形式でレンダリングされる可能性があるため、生のコンテンツをあらゆる出力形式に含める柔軟性を提供したいと考えていますが、HTMLに特権を与える理由はありません。同様の理由で、commonmarkとは異なり、HTMLエンティティを解釈しません。

- 目標2を達成するために、参照リンクの解析をローカルにします。`[foo][bar]`または`[foo][]`のように見えるものは、`[foo]`が文書の後半で定義されているかどうかに関係なく、参照リンクとして扱われます。したがって、`[like this]`のような括弧1つだけのショートカットリンク構文は削除しなければなりません。周囲の文脈を知らなくても何がリンクであるかは常に明確でなければなりません。

- 目標6をサポートするため、参照リンクでは大文字と小文字が区別されなくなりました。ASCIIコンテキストを超えてこれをサポートするには、すべての実装にUnicodeのCase Folding[^2]を組み込む必要がありますが、その必要はないようです。

[^2]: ケースの違い、つまり大文字小文字がばらつくときに一旦どちらかに統一化する操作、似た存在に「小文字化を行うCase Mappingがある」

- 目標8に記載されている均一性の原則の違反を避けるために、ブロック引用符 `>` の後に空白または改行が必要です:

```
>This is not a
>block quote in djot.
```

- 目標3を達成するために、強調のために二重文字を使用することは避けます。代わりに `_` 強調や `*` 強い強調を使用します。強調は後に空白が続かない限り、これらの文字1つで開始でき、前に空白がなく、間に異なる文字が存在しない限り、類似の文字が見つかったときに終了します。重複した場合は、最初に閉じられたものが優先されます。（この単純なルールにより、commonmarkでUnicode文字クラスを決定する必要性も回避できます ---目標6）。

- この最後の変更だけを見ても、表現上の盲点が多数発生することになります。たとえば単純なルールを考えると、

```
_(_foo_)_
```

は以下のように解析します

``` html
<em>(</em>foo<em>)</em>
```

それよりも

``` html
<em>(<em>foo</em>)</em>
  ```

後者の解釈が必要な場合、djotでは次の構文を使用できます。

```
_({_foo_})_
```

`{_` は強調を開くだけの `_` であり、`_}` は強調を閉じるだけの `_` です。同じことが `*` やオープナー、クローザー間があいまいなその他インライン書式設定マーカーでも同じです。これらの中括弧は、特定のインラインマークアップ、例えば `{=highlighting=}` 、 `{+insert+}` 、 `{-delete-}` に必要です。

- 目標1をサポートするために、コードスパン解析はバックトラックしません。したがって、コードスパンを開いて閉じないと、段落の終わりまで拡張されます。これは、commonmarkでフェンスドコードブロックが動作する方法と似ています。

```
This is `inline code.
```

- 目標9をサポートするために汎用属性構文が導入されています。属性はその前の行に配置することで任意のブロックレベルの要素に付加でき、その直後に配置することで任意のインラインレベルの要素に付加できます。

```
{#introduction}
This is the introductory paragraph, with
an identifier `introduction`.

           {.important color="blue" #heading}
## heading

The word *atelier*{weight="600"} is French.
```

- 汎用属性を使用する予定であるため、リンク内での引用タイトルはサポートされなくなりました。必要に応じて `title` 属性を追加できますが、これはあまり一般的ではないため特別な構文は必要ありません。

```
[Link text](url){title="Click me!"}
```

- ブロックレベルまたはインラインレベルの要素の任意のシーケンスに属性を付加できるようにするためフェンスで囲まれたdivと括弧で囲まれたスパンが導入されました。例えば、

```
{#warning .sidebar}
::: Warning
This is a warning.
Here is a word in [français]{lang=fr}.
:::
```

## 構文

完全な構文リファレンスについては、「[構文の説明](https://htmlpreview.github.io/?https://github.com/dai/djot/blob/master/doc/ja-syntax.html)」を参照してください。

djotのvim構文強調表示定義は、`editors/vim/` で提供されています。

## 実装

現在、6つのdjot実装があります。

- [djot.js (JavaScript/TypeScript)](https://github.com/jgm/djot.js)
- [djot.lua (Lua)](https://github.com/jgm/djot.lua)
- [djota (Prolog)](https://github.com/aarroyoc/djota)
- [jotdown (Rust)](https://github.com/hellux/jotdown)
- [godjot (Go)](https://github.com/sivukhin/godjot)
- [djoths (Haskell)](https://github.com/jgm/djoths)

[ここ](https://github.com/dcampbell24/djot-implementations)にはこれら実装のベンチマークがあります。


`djot.lua` はオリジナルのリファレンス実装でしたが、現在の開発は `djot.js` に重点を置いており、`djot.lua` が最新の構文変更に対応していない可能性があります。


## Tooling

- [Vim tooling](./editors/vim/) (located in this repo)
- Visual Studio Code tooling
  - [djot-vscode](https://github.com/ryanabx/djot-vscode)
  - [Djot-Marker](https://github.com/wisim3000/Djot-Marker)
- [Treesitter grammar](https://github.com/treeman/tree-sitter-djot)
- [Emacs major mode](./editors/emacs/)
  (located in this repo, requires the treesitter grammar)
- [Djockey](https://steveasleep.com/djockey/), a static site generator
  for technical writing and project documentation.

## ファイル拡張子

拡張子は、`.dj` ファイルの内容がdjot形式のテキストであることを示すために使用できます。

## ライセンス

コードとドキュメントはMITライセンスに基づいてリリースされています。

## 連絡

誤訳などお気づきの点がありましたら、開発者の [jgm](https://github.com/jgm) さんではなく、翻訳を行った [dai](https://github.com/dai) に連絡をしてください。
