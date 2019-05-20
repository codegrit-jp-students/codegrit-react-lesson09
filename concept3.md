# Uncontrolled Component

## Uncontrolled Componentとは

この方法ではReact側のstateで値を管理しないため`Uncontrolled Component`という呼び方をします。stateに代わって、Refsを利用してDOMノードの値を取得することでフォームの提出を行います。

上記で説明したRefsを利用してフォームを作ってみましょう。

```js
import React, { Component, createRef } from 'react'

export default class extends Component {
  constructor() {
    super()
    this.nameInput = createRef()
  }

  handleSubmit = (e) => {
    e.preventDefault()
    const name = this.nameInput.current.value
    this.props.handleSubmit(name)
  }
  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <lable>名前: </lable>
        <input type="text" ref={this.nameInput} />
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

### defaultValueを設定する

例えば、既存のユーザーの情報を編集する場合、ユーザーの既に持っている情報を予めフォームのValueとして入れたい場合もあるでしょう。Uncontrolled Componentを利用する場合はdefaultValueというpropsを設定することでこれを達成出来ます。

```js
import React, { Component, createRef } from 'react'

export default class extends Component {
  constructor() {
    super()
    this.nameInput = createRef()
  }

  handleSubmit = (e) => {
    e.preventDefault()
    const name = this.nameInput.current.value
    this.props.handleSubmit(name)
  }
  render() {
    const { user } = this.props
    const defaultName = user ? user.name : ""
    return (
      <form onSubmit={this.handleSubmit}>
        <lable>名前: </lable>
        <input
          type="text"
          ref={this.nameInput}
          defaultValue={defaultName} />
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

### fileインプットを扱う

fileインプットの値は読み込みのみ可能なため、Controlled Componentでstateを通して値を設定するということが出来ません。そのためUncontrolled Componentを利用する必要があります。

アップロードするファイルについては、以下のようにしてアクセス出来ます。

```js
this.refName.current.files[0]
```

例えば、プロフィール写真を扱うためのフォームであれば以下のようなコードとなります。

```js
import React, { Component, createRef } from 'react'

export default class extends Component {
  constructor() {
    super()
    this.fileInput = createRef()
  }
  handleSubmit = (e) => {
    e.preventDefault()
    const file = this.fileInput.current.files[0]
    switch (file.type) {
      case "image/jpeg":
      case "image/png":
        this.props.handleSubmit(name)
        break;
      default
        alert("ファイルの形式はJPEG形式かPNG形式を利用してください")
    }
  }
  render() {
    const { user } = this.props
    const defaultName = user ? user.name : ""
    return (
      <form onSubmit={this.handleSubmit}>
        <lable>プロフィール写真: </lable>
        <input
          type="file"
          ref={this.fileInput} />
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```