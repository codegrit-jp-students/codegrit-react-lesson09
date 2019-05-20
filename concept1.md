# Reactとフォーム

## HTMLフォームとReactでのフォームの違い

通常HTMLでフォームを扱う際、フォームに入力した値はDOMが自動的に管理してくれるようになっています。入力された値は例えば以下のようにして取り出すことが出来ます。

```js
document.getElementById('name').value
```

しかしReactを利用する場合は、この方法が利用出来ません。例えば、次のフォームをReactで書いてみましょう。

```js
import React, { Component } from 'react'

export default class extends Component {

  render() {
    return (
      <form>
        <label>名前: </label>
        <input type="text" name="name">
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

このフォーム上で、名前のフィールドになにかを入力しようとしても、何も起こらないことが確認出来るかと思います。

## Controlled Component

Reactでフォームを扱う際は、一般的にフォームに入力されたデータをsetStateファンクションを利用して保存し、そのstateを通して値を得るという方法を取ります。このようにstateを通して、React側でフォームの値をコントロールするため、この方法を"Controlled Component"と呼びます。

例えば、上記のフォームはControlled Componentを利用すると以下のように書き換えることが出来ます。

```js
import React, { Component } from 'react'

export default class extends Component {

  constructor(props) {
    super(props)
    this.state = {
      name: props.name ? props.name : ""
    }
  }
  handleChange = (e) => {
    e.preventDefault()
    this.setState({
      name: e.target.value
    })
  }

  handleSubmit = (e) => {
    e.preventDefault()
    const { name } = this.state;
    this.props.handleSubmit(name)
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>名前: </label>
        <input type="text" name="name" value={this.state.name} onChange={this.handleChange} />
        <input type="submit" value="Submit"/>
      </form>
    );
  }
}
```

## 各タグの書き方

### input, textareaタグ

これらはそれぞれ、同じように書くことが出来ます。checkboxの場合はvalueではなくcheckedで値を取得しなければいけないことに注意しましょう。

例: 要素の書き方

```js
<input type="text" onChange={this.handleChange} />
<input type="password" onChange={this.handleChange} />
<input type="email" onChange={this.handleChange} />
<input type="checkbox" onChange={this.handleChange} />
<textarea onChange={this.handleChange} />
```

例: 値の取得方法

```js
// checkbox以外
e.target.value

// checkboxの場合
e.target.checked
```

もう一点、inputタグを利用する場合でtypeがfileの場合は注意が必要です。この場合、ファイルの値を取得をするにはDOMノードに直接アクセスする必要があり、下記で説明するRefsとUncontrolled Componentを利用する必要があります。これについては後述します。


### selectタグ

selectタグを利用する際には、以下のようにします。

```js
<select value={this.state.value} onChange={this.handleChange}>
  <option value="option1">Option1</option>
  <option value="option2">Option2</option>
  <option value="option3">Option3</option>
  <option value="option4">Option4</option>
</select>
```

stateから取得したバリューをselectタグ内に`value={}`として与えてあげると、自動的オプションが選択された状態となります。

## 複数の項目を扱うフォーム

一般的なフォームでは一個ではなく、複数の項目を扱うことが多いでしょう。こうした場合に、インプット一個ずつにhandleChangeイベントを設置するのは同じことの繰り返しとなり非効率です。そこで、こうした場合には、フォームの要素にname Propsを付与し、nameを利用して値をstateに保存するのが便利です。

この際に、name propsは、`e.target.name`で取得することが出来ます。またオブジェクトのkeyをダイナミックに入力するためには以下のフォーマットを利用します。

```js
{
  [ダイナミックなkey]: 値
}
```

```js
import React, { Component } from 'react'

export default class extends Component {

  constructor(props) {
    super(props)
    this.state = {
      name: props.name ? props.name : ""
    }
  }
  handleChange = (e) => {
    e.preventDefault()
    const { name, type, checked, value } = e.target
    const val = type === "checkbox" ? checked : value // typeがcheckboxの場合と他の場合とで分ける
    this.setState({
      [name]: val
    })
  }

  handleSubmit = (e) => {
    e.preventDefault()
    const { name, aboutMe, termsAgreement } = this.state;
    const vals = {
      name,
      aboutMe
    }
    if (termsAgreement) {
      this.props.handleSubmit(vals)
    } else {
      alert("規約に同意していません")
    }
  }

  render() {
    const { name, aboutMe, termsAgreement } = this.state
    return (
      <form onSubmit={this.handleSubmit}>
        <label>名前: </label>
        <input type="text" name="name" value={name} onChange={this.handleChange} />
        <textarea name="aboutMe" value={aboutMe} onChange={this.handleChange} />
        <input type="checkbox" name="termsAgreement" value={termsAgreement} onChange={this.handleChange} />
        <input type="submit" value="Submit"/>
      </form>
    );
  }
}
```