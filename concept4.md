# Formik

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