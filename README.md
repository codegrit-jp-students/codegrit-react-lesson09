# レッスン9. フォーム

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

## Refs

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

## Uncontrolled Component

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

## Formikを利用する

上記では、ライブラリを利用せずフォームを構築する方法について説明してきました。しかし実際のアプリケーションでは、フォームの値の取得だけではなくフォームの値のバリデーションなどより多くのことを扱う必要があります。こうしたフォーム周りのことを扱うためのライブラリが多くあります。

ここでは、この中でもReact公式ドキュメントでも紹介されており、最も人気のあるものの一つFormikについて見ていきましょう。

## Formikを利用するメリット

公式ドキュメントではFormikはフォームに関して以下の3つの点でユーザーを助けてくれると説明しています。

- stateからフォームの入力値を出し入れする。
- バリデーションを行いエラーメッセージを表示する。
- フォームの提出。

JavaScriptコースのUnit2ではこれらをそれぞれ一から実装したので、そうした作業から開放されるのは大変大きいと感じるかと思います。

## 他のライブラリとの違い

Formik以外にもフォームを扱うライブラリは、react-final-form、redux-formなど存在しています。

こうした中でFormikは以下の点を主眼において開発されています。

- ファイルサイズが小さいこと
- フォーム実装で最も面倒な問題を解決すること
- ユーザーの見えないところで行われること(英語ではよくmagicと言われます)が少なく分かりやすいこと

## Formikをインストールする

いつもどおり、yarnを利用してformikライブラリをインストールします。

```js
yarn add formik
```

## Formikの基本形

Formikを利用したフォームは基本的にrender propsを利用して以下のように書くことが出来ます。

```js
const validateEmail = (val) => { /* メールのバリデーション */ }
const validateName = (val) => { /* 名前のバリデーション */ }
const validatePassword = (val) => { /* パスワードのバリデーション */ }

const sampleForm = () => {
  return (
    <Formik
      initialValues={{
        name: '',
        email: '',
        password: ''
      }}
      onSubmit={values => {
        const { name, email, rawPassword } = values;
        // ここでvalidationを行う。
        console.log("フォーム提出をここで扱います。")
      }}
      render={({ onSubmit }) => (
        <Form onSubmit={onSubmit}>
          <input type='text' name='name' />
          <input type='email' name='email' />
          <input type='password' name='password' />
          <button type="submit">
            登録する
          </button>
        </Form>
      )}
    />
  );
}
```

Lesson8でも学んだように、上記のコードのrender部分はpropsにchildrenを利用して書き換えることも出来ます。

```js
// childrenをpropsに使いFormikタグの間にrender propsを書く場合

const sampleForm = () => {
  return (
    <Formik
      initialValues={{
        name: '',
        email: '',
        password: ''
      }}
      onSubmit={values => {
        const { name, email, rawPassword } = values;
        // ここでvalidationを行う。
        console.log("フォーム提出をここで扱います。")
      }}
    >
      {({ onSubmit }) => (
        <Form>
          <Field type='text' name='name' />
          <Field type='email' name='email' />
          <Field type='password' name='password' />
          <button type="submit">
            登録する
          </button>
        </Form>
      )}
    </Formik>
  );
}
```

## Formicコンポーネント

上の基本の例でも分かる通り、Formikを利用するには、Formikタグでフォームの部分を囲います。Formikタグには以下のようなpropsを渡すことが出来ます。ここで説明する以外にもpropsはありますので、必要な場合公式ドキュメントを参照しながら実装しましょう。

### initialValues

フォームの初期の値を決めることが出来ます。例えば上記の例では、名前、メールアドレス、パスワードそれぞれの値を空白の文字列にしています。例えばユーザーの登録情報の編集などを行う場合は既に登録された値がありますので、これを読み込んでinitialValuesにセットします。

```js
initialValues={{
  name: '',
  email: '',
  password: ''
}}
```

### validate

ここでバリデーション用のファンクションを定義します。

```js
const validate = (values) => {
  let errors = {}
  // validation失敗ならerrorsオブジェクトにエラー内容を格納します。
  if (name === '') {
    errors.name = '名前は必須です。'
  }
  return errors
}
```

### onSubmit

フォームが提出された時の振る舞いを定義します。典型的な流れは以下のようになります。

1. バリデーションを行う。
2. フォームの値をサーバーにFetch APIを利用して送信する。
3. 成功時またはエラー時それぞれの処理を行う。

### validateOnBlur

onBlurイベント(編集しているフォームのフィールドからカーソルが離れること)の際にバリデーションを行うかどうかを指定します。デフォルトでは`true`です。

### validateOnChange

onChangeイベントの際にバリデーションを行うかどうかを指定します。デフォルトでは`true`です。

### render propsに渡される引数

render propsには以下の引数が渡されます。

- dirty - フォームの内容がinitialValuesに渡された値と異なる場合は`true`、そうでない場合は`false`を返します。例えばこれを利用してSubmitボタンのdisabledを有効にしたり出来ます。

- isSubmitting - onSubmitが進行している間は`true`、そうでない場合は`false`を返します。dirtyと同じような用途で利用出来ます。あるいはローディングスクリーンを出すために利用できます。

- isValidating - バリデーション中は`true`、そうでない場合は`false`を返します。バリデーションに時間がかかるケースで使いましょう。

- values - フォームの値が格納されたオブジェクトです。初期値はinitialValuesがセットされています。

- errors - バリデーションで返したerrorsオブジェクトです。このオブジェクトを利用してエラーメッセージなどを表示します。

- handleSubmit - FormikのPropsに指定されているhandleSubmitファンクションです。

## Formコンポーネント

Formコンポーネントを利用するとFormikが自動的にonSubmitイベントを取り扱ってくれます。

## Fieldコンポーネント

Fieldコンポーネントを利用することで、input、Select、textareaの各フィールドを簡単に書くことが出来、onChangeイベントやonBlurイベントを自動的に扱ってくれます。また独自のコンポーネントを書くことも出来ます。(このレッスンでは詳細は説明しないので必要があれば公式ドキュメントを参照してください。)

以下は各フィールドを書いた例です。

```js
// input
<Field type="email" name="email" placeholder="Email" />
// Select
<Field component="select" name="color">
  <option value="red">Red</option>
  <option value="green">Green</option>
  <option value="blue">Blue</option>
</Field>
// textarea
<Field component="textarea" name="bio" />
```

## 実際にフォームを書いてみよう

さて、ここまでで基本的なFormikの説明をしてきましたが実際に書いてみないことにはイメージが固まらないかと思います。ここでは、Formikを利用して典型的なユーザー登録フォームを作成してみましょう。

サンプルコードはこちらにあります。

[codegrit-react-lesson09-formik](https://github.com/codegrit-jp-students/codegrit-react-lesson09-formik)

### 1. create-react-appでアプリを作成する

まずはアプリを作成し、必要なパッケージをインストールします。今回はemotionとformikのみを利用します。

```bash
$ npx create-react-app formik-ex
$ cd formik-ex
$ yarn add formik @emotion/core @emotion/styled
```


### 2. バリデーションファンクションを作成する

ユーザー登録では、名前、メールアドレス、パスワードの3つのフィールドを入力してもらいます。

これらの3つのバリデーターを作成しましょう。場所はどこでも良いのですが、今回はsrc直下にlibフォルダーを作成し、その中にvalidators.jsを作成しましょう。

```js
// src/lib/validators.js

export const emailValidator = (val) => {
  return new Promise((resolve) => {
    let error = null;
    if (val === '') {
      error = 'メールアドレスは必須です。';
    } else if (!/^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,4}$/i.test(val)) {
      error = 'メールアドレスの形式が正しくありません。';
    }
    resolve({
      type: "email",
      error,
    });
  })
}

export const nameValidator = (val) => {
  return new Promise((resolve) => {
    let error = null;
    if (val === '') {
      error = '名前は必須です。';
    }
    resolve({
      type: "name",
      error,
    });
  })
}

export const passwordValidator = (val) => {
  return new Promise((resolve) => {
    let error = null;
    if (val === '') {
      error = 'パスワードは必須です。';
    } else if (val.length < 6) {
      error = 'パスワードは最低6文字で入力してください。'
    }
    resolve({
      type: "password",
      error,
    });
  })
}
```

### 3. Signupコンポーネントを作成する

次にSignupコンポーネントを作成します。まずはValidation以外の部分を以下のように書きましょう。

```js
// src/Signup.js

/** @jsx jsx */
import React from 'react';
import { Formik, Form, Field } from 'formik';
import styled from '@emotion/styled/macro'
import { jsx } from '@emotion/core'
import { passwordValidator, emailValidator, nameValidator } from './lib/validators';

// スタイリング

const formStyle = {
  display: 'flex',
  flexDirection: 'column',
  width: '400px'
}

const fieldStyle = {
  marginBottom: '1em',
  padding: '5px 10px',
}

const ErrorMessage = styled.div({
  color: 'red',
  margin: '0 0 3px',
  fontSize: '0.9em'
})

const CustomLabel = styled.label({
  marginBottom: 3,
  fontSize: '0.9em'
})

// Singupコンポーネント

export const Signup = () => (
  <div>
    <h1>ユーザー登録</h1>
    <Formik
      validateOnBlur={true}
      validateOnChange={false}
      initialValues={{
        name: '',
        email: '',
        password: ''
      }}
    >
      {({
        errors,
        handleSubmit,
        isSubmitting,
      }) => {
        return (
          <Form onSubmit={handleSubmit} css={formStyle}>
            {errors.server && <ErrorMessage>{errors.server}</ErrorMessage>}
            <CustomLabel>メールアドレス</CustomLabel>
            <Field type="email" name="email" css={fieldStyle} />
            {errors.email && <ErrorMessage>{errors.email}</ErrorMessage>}
            <CustomLabel>名前</CustomLabel>
            <Field type="text" name="name" css={fieldStyle} />
            {errors.name && <ErrorMessage>{errors.name}</ErrorMessage>}
            <CustomLabel>パスワード</CustomLabel>
            <Field type="text" name="password" css={fieldStyle}/>
            {errors.password && <ErrorMessage>{errors.password}</ErrorMessage>}
            <button type="submit" disabled={isSubmitting} css={{ marginTop: '1em'}}>
              Submit
            </button>
          </Form>
        );
      }}
    </Formik>
  </div>
);
```

ご覧のように、Formikコンポーネントのrender props部分には、エラーメッセージを表示する部分があり、Fieldコンポーネントを利用しています。Formikコンポーネントには、まだonSubmitとvalidateファンクションを入れていません。

```js
// パスワードのエラーがあれば、エラーメッセージを表示する。
{errors.password && <ErrorMessage>{errors.password}</ErrorMessage>}
```

また、onBlur時はバリデーションを行い、onChange時は行わないようにしています。

```js
validateOnBlur={true}
validateOnChange={false}
```

## App.jsを編集する

Signupコンポーネントが呼び出されるようにApp.jsを編集します。

```js
/** @jsx jsx */
import React from 'react';
import { jsx } from '@emotion/core'
import { Signup } from './Signup';

function App() {
  return (
    <div className="App" css={{ margin: 40 }}>
      <Signup />
    </div>
  );
}

export default App;
```

次にvalidateファンクションを作成していきましょう。

### validateファンクションを実装する

ここでは、JSコースのUnit2で学んだようにPromise.allを利用します。Formikコンポーネント内でvalidateプロップにファンクションを渡すと、フォームの入力値がvaluesという引数として渡されます。今回であれば、email、password、nameの3つです。

```js
const validate = ((values) => {
  const {
    email,
    password,
    name
  } = values
  return Promise.all([
    passwordValidator(password),
    emailValidator(email),
    nameValidator(name)
  ])
  .then((results) => {
    const errors = {} // errorsオブジェクトを作成する。
    results.forEach(result => {
      if (result.error) {
        // errorsオブジェクトに各フィールドのエラーを格納する。
        errors[result.type] = result.error
      }
    })
    // errorsオブジェクトに格納されたエラーが1つ以上あるかチェックする。
    if (Object.keys(errors).length > 0) {
      // throwを利用して、PromiseオブジェクトではなくerrorsオブジェクトをFormikコンポーネントに渡す。
      throw errors
    }
  })
})
```

ここまで出来たらFormikコンポーネントのvalidate propにこのファンクションを登録しましょう。

```js
<Formik
  validate={validate}
>
</Formik>
```

### mockSignupファンクションを実装する

実際のアプリでは、Fetch APIを利用してサーバーにフォームのデータを提出しますが、今回はmockSignupというファンクションを作成して、その挙動を擬似的に作成します。

```js
function mockSignup(values) {
  return new Promise((resolve, reject) => {
    // setTimeoutを利用して2秒間待った後に結果を出す。
    // ネットワークエラーの際の対応もするためrandomNumを利用してランダムな数を作り、10回に3回エラーが起こるようにしている。
    setTimeout(() => {
      const randomNum = Math.floor(Math.random() * 10)
      if (randomNum <= 7) {
        resolve({
          success: true,
          message: '登録に成功しました。'
        })
      } else {
        reject({
          success: false,
          message: 'ネットワークエラーが発生しました。'
        })
      }
    }, 2000)
  })
}
```

## handleSubmitファンクションを実装する

最後にhandleSubmitファンクションを実装します。このファンクションは最初にmockSignupファンクションを呼び出し、成功なら成功メッセージをalertで表示し、失敗ならサーバーエラーのメッセージをFormikに渡します。

FormikのhandleSubmit Propは渡されたファンクションにたいして以下の2つの引数を渡します。

- values - validateファンクションと同じくフォームの各フィールドの入力値
- formikBag - FormikBagという名前のオブジェクト。このオブジェクトには、Formikコンポーネントの持つファンクションが入っている。

今回はformikBagをactionという名前で渡し、以下の2つのファンクションを利用します。

- setSubmitting(bool) - isSubmittingの値を変更するためのファンクション
- setErrors({ キー: メッセージ ) - Formikのrender propsの引数の1つerrorsの値をセットするファンクション

```js
const handleSubmit = (values, actions) => {
  mockSignup(values)
  .then(data => {
    actions.setSubmitting(false); // isSubmittingをfalseにする
    alert(data.message)
  })
  .catch((error) => {
    const errors = {
      server: error.message // サーバーエラーのメッセージを登録する
    }
    actions.setSubmitting(false); // isSubmittingをfalseにする
    actions.setErrors(errors); // 新しいerrorsオブジェクトをFormikのrender propsに渡す
  })
}
```

このファンクションをFormikのonSubmit Propに渡します。

```js
<Formik
  onSubmit={handleSubmit}
>
</Formik>
```

ここまでですべてのファンクションの実装が完了しました。最終的にSignup.jsの中身は以下のようになります。

```js
/** @jsx jsx */
import React from 'react';
import { Formik, Form, Field } from 'formik';
import styled from '@emotion/styled/macro'
import { jsx } from '@emotion/core'
import { passwordValidator, emailValidator, nameValidator } from './lib/validators';

const formStyle = {
  display: 'flex',
  flexDirection: 'column',
  width: '400px'
}

const fieldStyle = {
  marginBottom: '1em',
  padding: '5px 10px',
}

const ErrorMessage = styled.div({
  color: 'red',
  margin: '0 0 3px',
  fontSize: '0.9em'
})

const CustomLabel = styled.label({
  marginBottom: 3,
  fontSize: '0.9em'
})

const handleSubmit = (values, actions) => {
  mockSignup(values)
  .then(data => {
    actions.setSubmitting(false);
    alert(data.message)
  })
  .catch((error) => {
    const errors = {
      server: error.message
    }
    actions.setSubmitting(false);
    actions.setErrors(errors)
  })
}

const validate = (({ email, name, password }) => {
  return Promise.all([
    passwordValidator(password), 
    emailValidator(email), 
    nameValidator(name)
  ])
  .then((results) => {
    const errors = {}
    results.forEach(result => {
      if (result.error) {
        errors[result.type] = result.error
      }
    })
    if (Object.keys(errors).length > 0) {
      throw errors
    }
  })
})

function mockSignup(values) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const randomNum = Math.floor(Math.random() * 10)
      if (randomNum <= 1) {
        resolve({
          success: true,
          message: '登録に成功しました。'
        })
      } else {
        reject({
          success: false,
          message: 'ネットワークエラーが発生しました。'
        })
      }
    }, 2000)
  })
}

export const Signup = () => (
  <div>
    <h1>ユーザー登録</h1>
    <Formik
      validateOnBlur={true}
      validateOnChange={false}
      initialValues={{ 
        name: '',
        email: '',
        password: ''
      }}
      validate={validate}
      onSubmit={handleSubmit}
    >
      {({ 
        errors,
        handleSubmit,
        isSubmitting,
      }) => {
        return (
          <Form onSubmit={handleSubmit} css={formStyle}>
            {errors.server && <ErrorMessage>{errors.server}</ErrorMessage>}
            <CustomLabel>メールアドレス</CustomLabel>
            <Field type="email" name="email" css={fieldStyle} />
            {errors.email && <ErrorMessage>{errors.email}</ErrorMessage>}
            <CustomLabel>名前</CustomLabel>
            <Field type="text" name="name" css={fieldStyle} />
            {errors.name && <ErrorMessage>{errors.name}</ErrorMessage>}
            <CustomLabel>パスワード</CustomLabel>
            <Field type="text" name="password" css={fieldStyle}/>
            {errors.password && <ErrorMessage>{errors.password}</ErrorMessage>}
            <button type="submit" disabled={isSubmitting} css={{ marginTop: '1em'}}>
              Submit
            </button>
          </Form>
        );
      }}
    </Formik>
  </div>
);
```

これでSignupフォームが完成しました。実際に自分で試してみましょう。各バリデーションは上手くいくでしょうか?

このレッスンではFormikについてよく使う機能にフォーカスして紹介しました。ここで紹介した他にもFormikのドキュメントでは様々な機能が紹介されています。ほとんどの場合は今回紹介した機能だけでまかなえるはずですが、実際のアプリ作成で今回の説明だけで上手く実装出来ない部分が出てきたらドキュメントを参照するようにしましょう。


## 更に学ぼう

- [React公式 - フォーム](https://ja.reactjs.org/docs/forms.html)
- [React公式 - RefとDOM](https://ja.reactjs.org/docs/refs-and-the-dom.html#___gatsby)
- [Formik公式(英語)](https://jaredpalmer.com/formik/)