# Formikを利用してみよう

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