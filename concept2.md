# Refs

## Refsとは

さて、このレッスンの最初で通常はフォームの値はDOMがコントロールし、Reactではその方法を使えないと書きました。しかし、Reactでは`Refs`という仕組みを通してDOMノードにアクセスすることが出来ます。このRefsを利用することで、React側ではなくDOM側でフォームの値を管理することが出来ます。

*RefはReferenceの略で日本語の参照を意味します。sがついているのは複数形だからです。今後Refとsを省略する場合は、参照するものが一つだけであることを意味します。

ここでは、このRefsという仕組みについて最初に詳しく見てみましょう。

### Refsの使いどころ

RefsはDOMの管理するステートへとアクセスするものですが、Reactでは基本的にステートはコンポーネントが管理するべきと考えられています。そのためRefsの使用は最低限に抑え、どうしても必要な場合のみに使用は限定しましょう。主に使い所と言えるのは以下のようなパターンです。

- jQueryなど直接DOMを操作するような外部のライブラリとやり取りしたいとき
- ある要素のDOM上での位置を取得したい場合

### Refsの作り方

Refsを利用するには、まずcreateRefファンクションをインポートします。

```js
import React, { Component, createRef } from 'react'
```

その上で、constructor上でRefsでDOMにアクセスしたいReact要素用にRefを作成します。作成には先程インポートしたcreateRefファンクションを利用します。

```js
import React, { Component, createRef } from 'react'

export default class extends Component {
  constructor() {
    super()
    this.firstRef = createRef() 
  }
}
```

次に、React要素と作成したRefとを結びつけます。

```js
import React, { Component, createRef } from 'react'

export default class extends Component {
  constructor() {
    super()
    this.firstRef = createRef()
  }
  render() {
    return (
      <div ref={this.firstRef} />
    );
  }
}
```

## Refsにアクセスする

Refsを利用してDOMノードにアクセスするには以下のようにします。

```js
this.firstRef.current
```

また、DOMノードに保存された値にアクセスするには以下のようにします。

```js
this.firstRef.current.value
```