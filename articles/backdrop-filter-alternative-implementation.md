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


実際にインターネットで検索してみると、CSS を用いたグラスモーフィズムの実装は [`backdrop-filter`](https://developer.mozilla.org/en-US/docs/Web/CSS/backdrop-filter) を使用する方法ばかりがヒットします。しかし、 `backdrop-filter` はデフォルト設定の場合 Firefox では動きません。[^2] [^3] そこで、`backdrop-filter` を使用せずとも同等の視覚効果を得る方法について考えます。

# 本編

今回は例として、`backdrop-filter` を使用せずに以下のようなログインフォームのデザインを作成することにします。Vueを使って書いていますが、素のHTML/CSSでも使える方法です。

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

そしてCSSですが、フォームの全体に半透明の灰色 (`#2228`) を設定した上で、`backdrop-filter` を使用して彩度150%・ぼかし12pxの背景効果をかけています。

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
    <LoginForm/>
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

body {
  margin: 0;
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
    <img class="icon" src="../assets/ryokohbato.png" alt="ryokohbato's icon" />
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
import { Component, Vue } from "vue-property-decorator";

@Component
export default class LoginForm extends Vue {}
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

では、具体的な代替実装について考えたいと思います。まず前提として、`backdrop-filter` を使用せずにぼかしなどの視覚効果を得るために、[`filter`](https://developer.mozilla.org/ja/docs/Web/CSS/filter) を使用することができます。

## clip-pathを用いた実装

おそらくこれが最も思いつきやすく、かつ汎用性も高い案だと思います。[clip-path](https://developer.mozilla.org/ja/docs/Web/CSS/clip-path) は、要素を簡単にクリッピングできるCSSプロパティです。clip-pathを使用して背景効果を与える手順は以下の通りです。

1. 背景を2枚重ね、(説明のために上のものを背景1、下のものを背景2と呼びます) その上にログインフォームを重ねる。

![](https://storage.googleapis.com/zenn-user-upload/b77e4142558b-20220315.png)

2. `filter` を用いて背景1に視覚効果をかける。

![](https://storage.googleapis.com/zenn-user-upload/32661e31aac3-20220315.png)

3. `clip-path` を用いて背景1をログインフォームと同じ大きさに切り抜く。

![](https://storage.googleapis.com/zenn-user-upload/1062665a7c9e-20220315.png)

では手順ごとにコードを追っていきます。重要な部分のみ抜粋して示しますが、以下のアコーディオンからソースコードの全体を確認できます。

### 1.

背景を2枚重ねてその上にログインフォームを重ねます。

```vue:App.vue
<div id="app">
  <Background/>
  <GlassedBackground class="glassed-background"/>
  <LoginForm class="login-form"/>
</div>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import Background from "./components/Background.vue";
import GlassedBackground from "./components/GlassedBackground.vue";
import LoginForm from "./components/LoginForm.vue";

@Component({
  components: {
    Background,
    GlassedBackground,
    LoginForm,
  },
})
export default class App extends Vue {}
</script>

<style lang="scss">
#app {
  position: relative;

  .glassed-background {
    position: absolute;
    top: 0;
  }

  .login-form {
    position: absolute;
    top: 0;
  }
}
</style>
```

```vue:components/Background.vue
<template>
  <div class="background"/>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";

@Component
export default class Background extends Vue {}
</script>

<style scoped lang="scss">
.background {
  background-image: url("../assets/background.jpg");
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  height: 100vh;
  width: 100vw;
}
</style>
```

```vue:components/GlassedBackground.vue
<template>
  <div class="background"/>
</template>

<style scoped lang="scss">
.background {
  background-image: url("../assets/background.jpg");
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  height: 100vh;
  width: 100vw;
}
</style>
```

### 2.

`filter` を用いて背景2に視覚効果をかけます。先程と同様、彩度150%・ぼかし12pxの視覚効果をかけます。

```vue:components/GlassedBackground.vue
<style scoped lang="scss">
.background {
  filter: saturate(1.5) blur(12px);
}
</style>
```

### 3.

`clip-path` を用いて背景2をログインフォームと同じ大きさに切り抜きます。ウィンドウサイズの変更を検知して毎回計算しているだけなので、Vanilla JSでも簡単に書けると思います。

```vue:components/GlassedBackground.vue
<script lang="ts">
export default {
  data () {
    return {
      insetTop: 60,
      height: window.innerHeight,
      width: window.innerWidth,
    }
  },
  computed: {
    insetRight: function () {
      return (this.width - 400) / 2;
    },
    insetBottom: function () {
      return (this.height - 500 - this.insetTop);
    },
    insetLeft: function () {
      return (this.width - 400) / 2;
    },
    clipPath: function () {
      return {
        clipPath: `inset(${this.insetTop}px ${this.insetRight}px ${this.insetBottom}px ${this.insetLeft}px round 8px)`,
      }
    }
  },
  methods: {
    resizeHandler: function () {
      this.width = window.innerWidth;
      this.height = window.innerHeight;
    }
  },
  mounted () {
    window.addEventListener('resize', this.resizeHandler);
  },
  beforeDestroy () {
    window.removeEventListener('resize', this.resizeHandler);
  }
}
</script>
```

ログインフォームと同じ大きさでくり抜くためには、角を8pxの半径で丸める必要がありますが、(ログインフォームには `border-radius: 8px` が設定されている) `inset()` の末尾に `round 8px` を付けることで、角を半径8pxで丸めることが出来ます。

:::details ソースコード全体を確認
```vue:App.vue
<template>
  <div id="app">
    <Background/>
    <GlassedBackground class="glassed-background"/>
    <LoginForm class="login-form"/>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import Background from "./components/Background.vue";
import GlassedBackground from "./components/GlassedBackground.vue";
import LoginForm from "./components/LoginForm.vue";

@Component({
  components: {
    Background,
    GlassedBackground,
    LoginForm,
  },
})
export default class App extends Vue {}
</script>

<style lang="scss">
html {
  background-color: #222;
  height: 100%;
  width: 100%;
}

body {
  margin: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  position: relative;
  text-align: center;

  .glassed-background {
    position: absolute;
    top: 0;
  }

  .login-form {
    margin-top: 60px;
    position: absolute;
    top: 0;
  }
}
</style>
```

```vue:components/Background.vue
<template>
  <div class="background"/>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";

@Component
export default class Background extends Vue {}
</script>

<style scoped lang="scss">
.background {
  background-image: url("../assets/background.jpg");
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  height: 100vh;
  width: 100vw;
}
</style>
```

```vue:components/GlassedBackground.vue
<template>
  <div class="background" :style="clipPath"/>
</template>

<script lang="ts">
export default {
  data () {
    return {
      insetTop: 60,
      height: window.innerHeight,
      width: window.innerWidth,
    }
  },
  computed: {
    insetRight: function () {
      return (this.width - 400) / 2;
    },
    insetBottom: function () {
      return (this.height - 500 - this.insetTop);
    },
    insetLeft: function () {
      return (this.width - 400) / 2;
    },
    clipPath: function () {
      return {
        clipPath: `inset(${this.insetTop}px ${this.insetRight}px ${this.insetBottom}px ${this.insetLeft}px round 8px)`,
      }
    }
  },
  methods: {
    resizeHandler: function () {
      this.width = window.innerWidth;
      this.height = window.innerHeight;
    }
  },
  mounted () {
    window.addEventListener('resize', this.resizeHandler);
  },
  beforeDestroy () {
    window.removeEventListener('resize', this.resizeHandler);
  }
}
</script>

<style scoped lang="scss">
.background {
  background-image: url("../assets/background.jpg");
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  filter: saturate(1.5) blur(12px);
  height: 100vh;
  width: 100vw;
}
</style>
```
:::

この実装は確かにFirefoxでも動きます。

## backgroundを継承することによる実装

かなり限られた場合にのみ利用できる方法ですが、こちらも面白い方法です。今回のような、画像の上に小さな要素が乗っているような場合に役立ちます。

まずはページ全体に背景画像を設定します。

```vue:App.vue
<template>
  <div id="app">
    <LoginForm/>
  </div>
</template>

<style lang="scss">
#app {
  background-image: url("./assets/background.jpg");
  background-position: center;
  background-repeat: no-repeat;
  height: calc(100vh - 60px);
  padding-top: 60px;
  width: 100vw;

  .container {
    margin-left: 50%;
    transform: translateX(-50%);
  }
}
</style>
```

ログインフォームの部分も全く同様に作ります。背景用の要素を追加しておきます。

```diff vue:components/LoginForm.vue
+ <div class="container">
+   <div class="glassed-background"></div>
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
+   </div>
+ </div>
```

CSSにいくつかポイントがあります。

- `background: inherit;` を指定して、`background` の値を親から子に継承し続けます。背景用の要素の `::before` 疑似要素まで `background` を継承し続けたら、`filter` プロパティで視覚効果を適用します。
- ぼかし半径の大きさだけ外側に要素を広げておき、`overflow: hidden;` でその部分を表示しないようにします。そうすることで、要素全体にぼかしが十分にかかります。

```scss:components/LoginForm.vue
.container {
  $form-width: 400px;
  background: inherit;
  background-position: 50% calc(50% + ((100vh - 500px) / 2 - 60px));
  border-radius: 8px;
  height: 500px;
  overflow: hidden;
  position: relative;
  width: $form-width;

  .glassed-background {
    background: inherit;
    filter: blur(12px) brightness(1.2);
    position: absolute;
    left: -12px;
    right: -12px;
    top: -12px;
    bottom: -12px;
  }
}
```

`background` プロパティは非継承プロパティなので、[^4] `background: inherit;` を明示的に指定する必要があります。

:::details ソースコード全体を確認
```vue:App.vue
<template>
  <div id="app" :style="backgroundSize">
    <LoginForm/>
  </div>
</template>

<script lang="ts">
import LoginForm from "./components/LoginForm.vue";

export default {
  data () {
    return {
      backgroundHeight: window.innerHeight * 0.75 < window.innerWidth ? window.innerWidth * 4 / 3 : window.innerHeight,
      backgroundWidth: window.innerHeight * 0.75 < window.innerWidth ? window.innerWidth : window.innerHeight * 0.75,
    }
  },
  computed: {
    backgroundSize: function () {
      return {
        backgroundSize: `${this.backgroundWidth}px ${this.backgroundHeight}px`,
        // backgroundSize: `cover`,
      };
    }
  },
  methods: {
    resizeHandler: function () {
      if (window.innerHeight * 0.75 < window.innerWidth) {
        this.backgroundHeight = window.innerWidth * 4 / 3;
        this.backgroundWidth = window.innerWidth;
      } else {
        this.backgroundHeight = window.innerHeight;
        this.backgroundWidth = window.innerHeight * 0.75;
      }
    }
  },
  mounted () {
    window.addEventListener('resize', this.resizeHandler);
  },
  beforeDestroy () {
    window.removeEventListener('resize', this.resizeHandler);
  },
  components: {
    LoginForm,
  },
}
</script>

<style lang="scss">
html {
  background-color: #222;
  height: 100%;
  width: 100%;
}

body {
  margin: 0;
}

#app {
  background-image: url("./assets/background.jpg");
  background-position: center;
  background-repeat: no-repeat;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  height: calc(100vh - 60px);
  text-align: center;
  padding-top: 60px;
  width: 100vw;

  .container {
    margin-left: 50%;
    transform: translateX(-50%);
  }
}
</style>
```

```vue:components/LoginForm.vue
<template>
  <div class="container">
    <div class="glassed-background"></div>
    <div class="form">
      <img class="icon" src="../assets/ryokohbato.png" alt="ryokohbato's icon" />
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
  </div>
</template>
<script lang="ts">
import { Component, Vue } from "vue-property-decorator";

@Component
export default class LoginForm extends Vue {}
</script>

<style scoped lang="scss">
.container {
  $form-width: 400px;
  background: inherit;
  background-position: 50% calc(50% + ((100vh - 500px) / 2 - 60px));
  border-radius: 8px;
  height: 500px;
  overflow: hidden;
  position: relative;
  width: $form-width;

  .glassed-background {
    background: inherit;
    filter: blur(12px) brightness(1.2);
    position: absolute;
    left: -12px;
    right: -12px;
    top: -12px;
    bottom: -12px;
  }

  .form {
    background-color: #2228;
    border-radius: 8px;
    display: inline-block;
    height: 500px;
    left: 0;
    position: absolute;
    top: 0;
    width: $form-width;
    z-index: 10;

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
}
</style>
```
:::

# まとめ

`backdrop-filter` を使用せずに同様の視覚効果を得る方法として、同じ画像を2枚重ねて一方を `clip-path` でくり抜く方法と、`background: inherit;` を用いて `background` プロパティの値を継承し続ける方法の2つを紹介しました。どちらもほとんどのモダンブラウザで利用可能な方法なので、是非利用してみてください。

[^1]: [What is Glassmorphism? Create This New Design Effect Using Only HTML and CSS](https://www.freecodecamp.org/news/glassmorphism-design-effect-with-html-css/)
[^2]: ["backdrop-filter" | Can I use](https://caniuse.com/?search=backdrop-filter)
[^3]: [bug 1578503](https://bugzilla.mozilla.org/show_bug.cgi?id=1578503)
[^4]: [background | MDN](https://developer.mozilla.org/ja/docs/Web/CSS/background#%E5%85%AC%E5%BC%8F%E5%AE%9A%E7%BE%A9)
