---
title: "Firefoxでも動くbackdrop-filterの代替実装"
emoji: "🦊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["css"]
published: false
---

この記事は[KMCアドベントカレンダー2021](https://adventar.org/calendars/6895)の19日目の記事です。大遅刻です。

# 概要

1月の半ば、[KMC](https://www.kmc.gr.jp) のSlack上でグラスモーフィズムが話題にあがりました。グラスモーフィズムは、すりガラスのように背景を透かしつつぼかすデザインです。[^1] ここで私は、グラスモーフィズムを素直に実装しても Firefox では動かないのではないか、という話を持ち出しました。

![それはそうと、グラスモーフィズム素直に実装するとFirefoxで動かない説があるんだけどもう対応してたっけ](https://storage.googleapis.com/zenn-user-upload/fd15a8c5a783-20220302.png)

![Can be enabled by setting the layout.css.backdrop-filter.enabled and gfx.webrender.all preference to true in about:config. ふむ](https://storage.googleapis.com/zenn-user-upload/08e0f9434031-20220302.png)

![一応黒魔術を使えば実装できたはず (忘れた)](https://storage.googleapis.com/zenn-user-upload/b9a899647a65-20220302.png)


実際にインターネットで検索してみると、CSS を用いたグラスモーフィズムの実装は `backdrop-filter` を使用する方法ばかりがヒットします。しかし、 `backdrop-filter` はデフォルト設定の場合 Firefox では動きません。[^2] [^3] そこで、`backdrop-filter` を使用せずとも同等の視覚効果を得る方法について考えます。

# 本編

今回は例として、`backdrop-filter` を使用せずに以下のようなログインフォームのデザインを作成することにします。

![本記事で解説に使用するデザインのプレビュー画像](https://raw.githubusercontent.com/ryokohbato/zenn.backdrop-filter-alternative-implementation/images/images/preview.png)

:::message
この記事で掲載するコードは全て [こちらのリポジトリ](https://github.com/ryokohbato/zenn.backdrop-filter-alternative-implementation) で公開しており、実際に動かして確認していただけます。
:::

まずはこのデザインを素直に実装してみます。重要な部分のみ抜粋します。ソースコードの全体を示すと長くなるので、以下のアコーディオンを開いて確認してください。

まずはページ全体に背景画像を設定します。

```css
html {
  background-image: url("./assets/background.jpg");
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  height: 100%;
  width: 100%;
}
```

続いてログインフォームの部分を作ります。HTMLは概ね以下のようになっています。

```html
<div class="form">
  <img class="icon" src="../assets/ryokohbato.png" alt="" />
  <p class="name">ryokohbato</p>
  <div class="password">
    <input
      class="password__input"
      type="password"
      name="password"
      id="password"
      placeholder="PASSWORD"
    />
  </div>
</div>
```

そしてCSSですが、フォームの全体に半透明の灰色 (`#2228`) を設定した上で、`backdrop-filter` を使用して彩度150%、ぼかし12pxの背景効果をかけています。

```scss
.form {
  background-color: #2228;
  backdrop-filter: saturate(1.5) blur(12px);
}
```

:::details ソースコード全体を確認
```vue:App.vue
<template>
  <div id="app">
    <LoginForm></LoginForm>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import LoginForm from "./components/LoginForm.vue";

@Component({
  components: {
    LoginForm,
  },
})
export default class App extends Vue {}
</script>

<style lang="scss">
html {
  background-color: #222;
  background-image: url("./assets/background.jpg");
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  height: 100%;
  width: 100%;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  margin-top: 60px;
}
</style>
```

```vue:components/LoginForm.vue
<template>
  <div class="form">
    <img class="icon" src="../assets/ryokohbato.png" alt="" />
    <p class="name">ryokohbato</p>
    <div class="password">
      <input
        class="password__input"
        type="password"
        name="password"
        id="password"
        placeholder="PASSWORD"
      />
    </div>
  </div>
</template>
<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";

@Component
export default class LoginForm extends Vue {
  @Prop() private msg!: string;
}
</script>

<style scoped lang="scss">
.form {
  $form-width: 400px;
  background-color: #2228;
  backdrop-filter: saturate(1.5) blur(12px);
  border-radius: 8px;
  display: inline-block;
  height: 500px;
  width: $form-width;

  .icon {
    border-radius: 50%;
    height: calc($form-width * 0.5);
    margin-top: 60px;
    width: calc($form-width * 0.5);
  }

  .name {
    color: white;
    font-size: 28px;
    font-weight: bold;
  }

  .password {
    display: inline-block;
    width: 60%;

    &__input {
      background-color: #7773;
      border-radius: 4px;
      border: none;
      border-bottom: 2px solid white;
      color: white;
      font-size: 28px;
      padding: 0 12px;
      width: 80%;

      &::placeholder {
        font-size: 16px;
        color: #fff9;
        transform: translateY(-2px);
      }

      &:focus {
        background-color: #aaa3;
        border-bottom: 2px solid #42b983;
        outline: 0;
        transition: all .3s;
      }

    }

    &__submit {
      border-radius: 4px;
      border: 1px solid white;
      font-size: 18px;
      font-weight: bold;
      margin-top: 140px;
      padding: 5px 60px;
      width: 80%;
    }
  }
}
</style>
```
:::

さて、先程述べた通り、このコードはFirefoxでは動きません。以下のように、背景効果が適用されていない状態で表示されるはずです。

![Firefoxでのプレビュー画像](https://raw.githubusercontent.com/ryokohbato/zenn.backdrop-filter-alternative-implementation/images/images/preview-firefox.png)

[^1]: [What is Glassmorphism? Create This New Design Effect Using Only HTML and CSS](https://www.freecodecamp.org/news/glassmorphism-design-effect-with-html-css/)
[^2]: ["backdrop-filter" | Can I use](https://caniuse.com/?search=backdrop-filter)
[^3]: [bug 1578503](https://bugzilla.mozilla.org/show_bug.cgi?id=1578503)
