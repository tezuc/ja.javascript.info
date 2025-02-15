libs:
  - d3
  - domtree

---


# DOM ナビゲーション

DOM は要素やコンテンツに対して様々なことができますが、最初に対応する DOM オブジェクトに到達する必要があります。

DOM 上のすべての操作は `document` オブジェクトから始まります。そこから任意のノードにアクセスできます。

これは DOM ノード間を移動できるリンクの図です。:

![](dom-links.svg)

これらについてより深く議論しましょう。

## トップ: documentElement と body 

最上位のツリーノードは `documet` プロパティとして直接利用可能です:

`<html>` = `document.documentElement`
: 最上位のドキュメントノードは `document.documentElement` で、`<html>` タグの DOM ノードです。

`<body>` = `document.body`
: もう１つの広く使われている DOM ノードは `<body>` 要素です --  `document.body`.

`<head>` = `document.head`
: `<head>` タグは `document.head` で利用可能です。

````warn header="落とし穴があります: `document.body` は `null` になる場合があります"
スクリプトは実行時に存在しない要素へアクセスできません。

特に、スクリプトが `<head>` の内側にいる場合、ブラウザはまだ body を読み込んでいないため、`document.body` は利用できません。

従って、以下の例では最初の `alert` は `null` を表示します:

```html run
<html>

<head>
  <script>
*!*
    alert( "From HEAD: " + document.body ); // null, まだ <body> はありません
*/!*
  </script>
</head>

<body>

  <script>
    alert( "From BODY: " + document.body ); // HTMLBodyElement, 今は存在します
  </script>

</body>
</html>
```
````

```smart header="DOM の世界では、`null` は \"存在しない\" を意味します"
DOM では、`null` 値は "存在しない" もしくは "このようなノードはない" を意味します。
```

## 子: childNodes, firstChild, lastChild 

これから使う2つの用語があります。:

- **子ノード (または子)** -- 直接の子要素です。つまり、与えられた要素にネストされています。例えば `<head>` と `<body>` は `<html>` 要素の子です。
- **子孫** -- 子要素、子要素など、指定された要素にネストされたすべての要素です。

例えば、ここで `<body>` は子 `<div>` と `<ul>` (といくつかの空のテキストノード)を持ちます。:

```html run
<html>
<body>
  <div>Begin</div>

  <ul>
    <li>
      <b>Information</b>
    </li>
  </ul>
</body>
</html>
```

...また、 `<body>` のすべての子孫は、直接の子 `<div>`, `<ul>` だけでなく、`<li>` (`<ul>` の子) や `<b>` (`<li>` の子)のような、さらにネストされた要素を含む -- サブツリー全体です。

**`childNodes` のコレクションは、テキストノードを含むすべての子ノードを持ちます。**

以下は `document.body` の子を表示します:

```html run
<html>
<body>
  <div>Begin</div>

  <ul>
    <li>Information</li>
  </ul>

  <div>End</div>

  <script>
*!*
    for (let i = 0; i < document.body.childNodes.length; i++) {
      alert( document.body.childNodes[i] ); // Text, DIV, Text, UL, ..., SCRIPT
    }
*/!*
  </script>
  ...他の要素...
</body>
</html>
```

ここで興味深い点に注目してください。上の例を実行すると、表示される最後の要素は `<script>` です。実際には、ドキュメントはそれ以降により多くのものを持ちますが、スクリプト実行時点でブラウザはまだそれらは読み込んでいないため、スクリプトにそれらは見えません。

**プロパティ `firstChild` と `lastChild` で最初と最後の子への高速なアクセスができます**

これらは単なる簡略表記です。子ノードが存在する場合は、常に次のようになります:
```js
elem.childNodes[0] === elem.firstChild
elem.childNodes[elem.childNodes.length - 1] === elem.lastChild
```

子ノードがあるかどうかをチェックするための特別な関数 `elem.hasChildNodes()` もあります。

### DOM コレクション 

ご覧の通り、`childNodes` は配列のように見えます。が、実際には配列ではなくむしろ *コレクション* -- 特別な配列ライクで反復可能なオブジェクトです。

2つの重要な点があります:

1. 反復する際 `for..of` が使えます:
  ```js
  for (let node of document.body.childNodes) {
    alert(node); // コレクションのすべてのノードを表示する
  }
  ```
  これは反復可能(必須で `Symbol.iterator` プロパティを提供する)のためです。

2. 配列ではないため、配列メソッドは動作しません:
  ```js run
  alert(document.body.childNodes.filter); // undefined (フィルタメソッドを持っていません!)
  ```

最初の1つ目は良いです。2つ目も、配列メソッドが必要な場合は、`Array.from` でコレクションから "本当の" 配列を作ることができるので許容できます。:

  ```js run
  alert( Array.from(document.body.childNodes).filter ); // これで使えます
  ```

```warn header="DOM コレクションは読み取り専用です"
DOM コレクションやさらに -- この章でリストされている *すべての* ナビゲーションプロパティは読み取り専用です。

代入 `childNodes[i] = ...` などで子ノードを置き換えることはできません。

DOM の変更は他のメソッドを必要とします。それらについては次の章で見ていきましょう。
```

```warn header="DOM コレクションはライブです"
一部の例外を除き、ほぼすべての DOM コレクションは *ライブ* です 。つまり、それらは DOM の現在の状態を反映しています。

もし　`elem.childNodes` への参照を維持し、DOM にノードを追加/削除すると、コレクションの中に自動的に反映されます。
```

````warn header="コレクションのループで、`for..in` を使わないでください"
コレクションは `for..of` で反復可能です。時々 `for..in` を使おうとする人がいます。

`for..in` は使わないでください。`for..in` ループはすべての列挙可能なプロパティを反復します。コレクションは、通常は取得不要ないくつかの "余分な" ほとんど使われないプロパティを持っています: 

```html run
<body>
<script>
  // 0, 1, length, item, values などが表示されます。
  for(let prop in document.body.childNodes) alert(prop);
</script>
</body>
````

## 兄弟と親 

*兄弟(Siblings)* は同じ親(parent)の子ノードです。

例えば、`<head>` と `<body>` は兄弟です:

```html
<html>
  <head>...</head><body>...</body>
</html>
```

- `<body>` は `<head>` の "次の" または "右の" 兄弟と言われます。 
- `<head>` `<body>` の "前の" または "左の" 兄弟と言われます。

次のノード(次の兄弟) は `nextSibling` であり、前のノードは `previousSibling` です。

親は `parentNode` として利用可能です。

例えば:

```js run
  // <body> の親は <html> です。
  alert( document.body.parentNode === document.documentElement ); // true

  // <head> の後は <body> に行きます。
  alert( document.head.nextSibling ); // HTMLBodyElement

  // <body> の前は <head> です。
  alert( document.body.previousSibling ); // HTMLHeadElement
```

## Element-only navigation

上でリストされているナビゲーションプロパティは *すべての* ノードを参照します。例えば、`childNodes` では、テキストノード、要素ノードの両方を、さらに存在する場合にはコメントノードも見ることができます。

しかし、多くのタスクでは、テキストノードやコメントノードは必要ありません。タグを表し、ページの構造を形成する要素ノードを操作したいです。

なので、*要素ノード* だけを考慮にいれたナビゲーションリンクをもっと見てみましょう:

![](dom-links-elements.svg)

リンクは上で与えられたものと似ており、`Element` という言葉が内部にあります:

- `children` -- 要素ノードの子のみです。
- `firstElementChild`, `lastElementChild` -- 最初/最後の要素の子です。
- `previousElementSibling`, `nextElementSibling` -- 隣の要素です。
- `parentElement` -- 親の要素です。

````smart header="なぜ `parentElement`? 親は要素 ではない 場合はありますか？"
`parentElement` プロパティは "要素" の親を返しますが、`parentNode` は "任意のノード" の親を返します。それらのプロパティは通常同じです: 両方とも親を取得します。

`document.documentElement` の例外を除いて:

```js run
alert( document.documentElement.parentNode ); // document
alert( document.documentElement.parentElement ); // null
```

言い換えると、`documentElement` (`<html>`) はルートノードです。公式にはその親として `document` を持っています。しかし、`document` は要素ノードではないので、`parentNode` はそれを返し、`parentElement` は返しません。

これは、任意の要素 `elem` から `<html>` に移動したいが、`document` には移動したくな場合に役立ちます:
```js
while(elem = elem.parentElement) { // go up till <html>
  alert( elem );
}
```
````

上の例の1つを修正してみましょう: `childNodes` を `children` に置き換えます。これで要素のみが表示されます。:

```html run
<html>
<body>
  <div>Begin</div>

  <ul>
    <li>Information</li>
  </ul>

  <div>End</div>

  <script>
*!*
    for (let elem of document.body.children) {
      alert(elem); // DIV, UL, DIV, SCRIPT
    }
*/!*
  </script>
  ...
</body>
</html>
```

## 他のリンク: tables 

今まで、基本的なナビゲーションプロパティを説明しました。

特定の種類の DOM 要素は、便宜上、その種類に固有の追加のプロパティを提供することがあります。

テーブルはその素晴らしい例であり重要なケースです。

**`<table>`** 要素は次のプロパティをサポートします(上で与えられたものに加えて):
- `table.rows` -- テーブルの `<tr>` 要素のコレクションです。
- `table.caption/tHead/tFoot` -- 要素 `<caption>`, `<thead>`, `<tfoot>` への参照です。
- `table.tBodies` -- `<tbody>` 要素のコレクション(標準によると多数になれます) です。

**`<thead>`, `<tfoot>`, `<tbody>`** 要素は `rows` プロパティを提供します:
- `tbody.rows` -- 内側の `<tr>` のコレクション

**`<tr>`:**
- `tr.cells` -- 与えられた `<tr>` の中の `<td>` と `<th>` セルの集合です。
- `tr.sectionRowIndex` -- 囲んでいる `<thead>/<tbody>` の内部にある与えられた `<tr>` の番号です。
- `tr.rowIndex` -- テーブル内の `<tr>` の番号です。

**`<td>` と `<th>`:**
- `td.cellIndex` -- `<tr>` で囲まれている内側でのセルの番号です。

使用例:

```html run height=100
<table id="table">
  <tr>
    <td>one</td><td>two</td>
  </tr>
  <tr>
    <td>three</td><td>four</td>
  </tr>
</table>

<script>
  // "two" の td を取得
  let td = table.*!*rows[0].cells[1]*/!*;
  td.style.backgroundColor = "red"; // ハイライト
</script>
```

仕様: [tabular data](https://html.spec.whatwg.org/multipage/tables.html).

HTMLフォームのための追加のナビゲーションプロパティもあります。 フォームを使って作業を開始するときにそれらを見ていきます。

# サマリ 

与えられた DOM ノードで、ナビゲーションプロパティを使用することで直接隣接ノードに移動できます。

それらの2つの主要なセットがあります:

- すべてのノードを取得: `parentNode`, `childNodes`, `firstChild`, `lastChild`, `previousSibling`, `nextSibling`.
- 要素ノードのみの取得: `parentElement`, `children`, `firstElementChild`, `lastElementChild`, `previousElementSibling`, `nextElementSibling`

DOM 要素のいくつかの種類 -- e.g. tables -- はそれらのコンテンツにアクセスするための追加のプロパティやコレクションを提供します。
