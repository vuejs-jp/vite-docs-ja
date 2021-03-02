# 特徴

基本的に、Viteを使用した開発は静的ファイルサーバを使用した時とそれほど変わりません。 しかし、Viteはバンドラベースのセットアップで一般的な機能をサポートするためにネイティブESMをインポートすることで様々な拡張機能を提供します。

## NPM の依存関係の解決と先読みされるバンドル

ネイティブ ES のインポートは次のような生のモジュールをサポートしていません:

```js
import { someMethod } from 'my-dep'
```

上のようなコードはブラウザでエラーになります。Vite は提供される全てのソースファイルでこのような生のモジュールのインポートを検出し以下を実行します:
1. [先読みバンドル](./dep-pre-bundling) はページの読み込み速度を改善し、CommonJS / UMDモジュールを ESM に変換します。 先読みバンドルは [esbuild](http://esbuild.github.io/) で実行され、Vite のコールドスタート時間を  JavaScript ベースのバンドラーよりも大幅に高速化します。

2. インポートを `/node_modules/.vite/my-dep.js?v=f3sf2ebd` のように書き換えることでブラウザが正しくモジュールをインポートできるようにします。

**依存関係は強力にキャッシュされます**

ViteはHTTPヘッダーを介して依存関係のリクエストをキャッシュするため、依存関係をローカルで編集/デバッグする場合は、[ここの手順](./dep-pre-bundling#browser-cache)に従います。

## Hot Module Replacement

Vite はネイティブ ESM を介して [HMR API](./api-hmr) を提供します。 HMR 機能を備えたフレームワークは、API を活用して、ページを再読み込みしたり、アプリケーションの状態を損失することなく即座に正確な更新を提供できます。 Vite は[Vue Single File Components](https://github.com/vitejs/vite/tree/main/packages/plugin-vue) および [React Fast Refresh](https://github.com/vitejs/vite/tree/main/packages/plugin-react-refresh) ファーストパーティの HMR を提供します。[@prefresh/vite](https://github.com/JoviDeCroock/prefresh/tree/main/packages/vite) を介した preact の統合された公式のライブラリもあります。

これらを手動で設定する必要がないことには注意してください -  [create an app via `@vitejs/create-app`](./) を介してアプリケーションを作成する場合、これらはすでに構成されています。

## TypeScript

Vite は `.ts` ファイルをインポートすることをサポートしています。

Vite は `.ts` ファイルに対してのみ変換を実行し、型チェックは **実行しません**。 型チェックは IDE とビルドの過程にて実行されることを前提としています (ビルドスクリプトを `tsc --noEmit` で実行することができます)。

Vite は [esbuild](https://github.com/evanw/esbuild) を用いて TypeScriptをJavaScriptに変換します。 これは、vanilla の `tsc` よりも約20〜30倍高速であり、HMR の更新は50ミリ秒未満でブラウザーに反映されます

`esbuild` は型情報なしでビルドを実行するため、 const や enum の暗黙の型のみのインポートなどの特定の機能はサポートしていません。 TypeScript が分離されたトランスパイルで機能しない機能に対して警告するように、`compilerOptions`の下の `tsconfig.json` で `"isolatedModules"：true` を設定する必要があります。

### クライアントのタイプ

Vite はデフォルトでは Node.js の API を提供します. Vite でクライアント用のコードを使用するには `tsconfig` で `compilerOptions.types` に `vite/client`  を追加します:

```json
{
  "compilerOptions": {
    "types": ["vite/client"]
  }
}
```

これにより次のことが提供されます:

- アセットのインポート (例: `.svg` ファイルのインポート)
- `import.meta.env` に Vite が挿入した [env variables](./env-and-mode#env-variables) のタイプ
- `import.meta.hot` の [HMR API](./api-hmr) のタイプ

## Vue

Vite は Vue に対して最高のサポートをします:

- Vue 3 SFC はこちら [@vitejs/plugin-vue](https://github.com/vitejs/vite/tree/main/packages/plugin-vue)
- Vue 3 JSX はこちら [@vitejs/plugin-vue-jsx](https://github.com/vitejs/vite/tree/main/packages/plugin-vue-jsx)
- Vue 2 はこちら [underfin/vite-plugin-vue2](https://github.com/underfin/vite-plugin-vue2)

## JSX

`.jsx` と `.tsx` もすぐにサポートされます。 JSX のトランスパイルも [ESBuild](https://esbuild.github.io) を介して行われます, デフォルトはReact16フレーバーですが、ESBuildでのReact17スタイルのJSXサポートが追跡されます。[詳しくはこちら](https://github.com/evanw/esbuild/issues/334)。

Vue を使用している人は公式のプラグインである [@vitejs/plugin-vue-jsx](https://github.com/vitejs/vite/tree/main/packages/plugin-vue-jsx) を使用するべきです、 これは、HMR、グローバルコンポーネント解決、ディレクティブ、スロットなど、Vue 3 の固有の機能を提供します。

もしReact、または Vue で JSX を使用していない場合は, [`esbuild` option](/config/#esbuild) を使用して `jsxFactory` および `jsxFragment` を構成することができます。 例えば、 Preact の場合:

```js
// vite.config.js
export default {
  esbuild: {
    jsxFactory: 'h',
    jsxFragment: 'Fragment'
  }
}
```

さらに詳しく知りたい場合は [ESBuild docs](https://esbuild.github.io/content-types/#jsx) を見てください。

また、`jsxInject`（Viteのみのオプション）を使用して JSX ヘルパーを挿入し、手動インポートを回避できます。

```js
// vite.config.js
export default {
  esbuild: {
    jsxInject: `import React from 'react'`
  }
}
```

## CSS

`.css` ファイルをインポートすると、HMRをサポートする `<style>` タグを介してそのコンテンツがページに挿入されます。モジュールのデフォルトのエクスポートとして、処理されたCSSを文字列として取得することもできます。

### `@import` のインライン化と結合

Viteは、`postcss-import` を介した CSS `@ import` のインライン化をサポートするように事前構成されています。 CSS `@import` では、Viteエイリアスも尊重されます。さらに、インポートされたファイルが異なるディレクトリにある場合でも、すべてのCSS `url()` 参照は、正確性を確保するために常に自動的に結合されます。

`@import` エイリアスと URL の結合もSassファイルとLessファイルでサポートされています (詳しくはこちら [CSS Pre-processors](#css-pre-processors))。

### PostCSS

もしプロジェクトに有効な PostCSS が含まれている場合 ([postcss-load-config](https://github.com/postcss/postcss-load-config) でサポートされている任意の形式、例: `postcss.config.js`)、インポートされたすべてのCSSに自動的に適用されます。

### CSS Modules

`.module.css` で終わる全ての CSS ファイルは全て [CSS modules file](https://github.com/css-modules/css-modules) とみなされます。 このようなファイルをインポートすると、対応するモジュールオブジェクトが返されます:

```css
/* example.module.css */
.red {
  color: red;
}
```

```js
import classes from './example.module.css'
document.getElementById('foo').className = classes.red
```

CSS モジュールの動作は [`css.modules` option](/config/#css-modules) を参考にしてください。

`css.modules.localsConvention` がキャメルケースローカルを有効にするように設定されている場合（例：`localsConvention: 'camelCaseOnly'`）、名前付きインポートを使用することもできます:

```js
// .apply-color -> applyColor
import { applyColor } from './example.module.css'
document.getElementById('foo').className = applyColor
```

### CSS プリプロセッサ


Vite は最新のブラウザのみを対象としているため、CSSWG ドラフト（[postcss-nesting](https://github.com/jonathantneal/postcss-nesting) など）を実装する PostCSS プラグインでネイティブ CSS 変数を使用し、将来の標準に準拠したプレーンなCSSを作成することをお勧めします。

とは言うものの、Vite は `.scss`、`.sass`、 `.less`、`.styl`、 `.stylus` ファイルの組み込みサポートを提供します。それら にVite 固有のプラグインをインストールする必要はありませんが、対応するプリプロセッサ自体をインストールする必要があります。

```bash
# .scss and .sass
npm install -D sass

# .less
npm install -D less

# .styl and .stylus
npm install -D stylus
```

もし Vue で単一ファイルコンポーネントを使用している場合、これにより、 `<style lang =" sass ">` なども自動的に有効になります。

Viteは、SassおよびLessの `@import` 解決を改善し、Vite エイリアスも尊重されるようにします。さらに、ルートファイルとは異なるディレクトリにあるインポートされた Sass / Less ファイル内の相対的な `url()` の参照も、正確性を確保するために自動的に結合されます。

`@import` エイリアスと URL の結合は、API の制約のため、Stylus ではサポートされていません。

ファイル拡張子の前に `.module` を付けることで、プリプロセッサと組み合わせて CSS モジュールを使用することもできます（例：`style.module.scss`）。

## 静的なアセット

静的アセットをインポートすると、提供時に解決されたパブリックURLが返されます:

```js
import imgUrl from './img.png'
document.getElementById('hero-img').src = imgUrl
```

特別なクエリにより、アセットの読み込み方法を変更できます:

```js
// アセットを URL として明示的にロードする
import assetAsURL from './asset.js?url'
```

```js
// アセットを文字列として明示的にロードする
import assetAsString from './shader.glsl?raw'
```

```js
// ウェブワーカーをロードする
import Worker from './worker.js?worker'
```

```js
// ビルド時に base64 文字列としてインライン化されたウェブワーカー
import InlineWorker from './worker.js?worker&inline'
```

詳しくは [Static Asset Handling](./assets) を見てください。

## JSON


JSON ファイルは直接インポートできます - また、名前付きインポートもサポートされています：:

```js
// オブジェクト全体をインポートする場合
import json from './example.json'
// 名前付きエクスポートとしてルートフィールドをインポートします - ツリーシェイクに役立ちます！
import { field } from './example.json'
```

## Glob のインポート

Viteは、特別な `import.meta.glob` 関数を介してファイルシステムから複数のモジュールをインポートすることをサポートしています:

```js
const modules = import.meta.glob('./dir/*.js')
```

上のコードは以下のように変換されます:

```js
// vite によって生成されたコード
const modules = {
  './dir/foo.js': () => import('./dir/foo.js'),
  './dir/bar.js': () => import('./dir/bar.js')
}
```

次に、 `modules` オブジェクトのキーを繰り返し処理して、対応するモジュールにアクセスできます:

```js
for (const path in modules) {
  modules[path]().then((mod) => {
    console.log(path, mod)
  })
}
```

一致したファイルはデフォルトで動的インポートを介して遅延ロードされ、ビルド中に個別のチャンクに分割されます。 もしあなたがすべてのモジュールを直接インポートする場合（たとえば、最初に適用されるこれらのモジュールの副作用に依存する場合）、代わりに `import.meta.globEager` を使用できます:

```js
const modules = import.meta.globEager('./dir/*.js')
```

上のコードは以下のように変換されます:

```js
// vite によって生成されたコード
import * as __glob__0_0 from './dir/foo.js'
import * as __glob__0_1 from './dir/bar.js'
const modules = {
  './dir/foo.js': __glob__0_0,
  './dir/bar.js': __glob__0_1
}
```

注意点:

- これは Vite のみの機能であり、Web または ES の標準ではありません。
- Glob パターンはインポート指定子のように扱われます。相対パス（`./`で始まる）または絶対パス（`/`で始まり、プロジェクトルートに対して解決される）のいずれかである必要があります。依存関係からの Glob はサポートされていません。
- The glob matching is done via `fast-glob` - check out its documentation for 
- Glob のマッチングは `fast-glob` を介して行われます。サポートされている Glob パターンについては、その[ドキュメント](https://github.com/mrmlnc/fast-glob#pattern-syntax)を確認してください。

## Web Assembly

Pre-compiled `.wasm` files can be directly imported - the default export will be an initialization function that returns a Promise of the exports object of the wasm instance:

```js
import init from './example.wasm'

init().then((exports) => {
  exports.test()
})
```

The init function can also take the `imports` object which is passed along to `WebAssembly.instantiate` as its second argument:

```js
init({
  imports: {
    someFunc: () => {
      /* ... */
    }
  }
}).then(() => {
  /* ... */
})
```

In the production build, `.wasm` files smaller than `assetInlineLimit` will be inlined as base64 strings. Otherwise, they will be copied to the dist directory as an asset and fetched on-demand.

## Web Workers

A web worker script can be directly imported by appending `?worker` to the import request. The default export will be a custom worker constructor:

```js
import MyWorker from './worker?worker'

const worker = new MyWorker()
```

The worker script can also use `import` statements instead of `importScripts()` - note during dev this relies on browser native support and currently only works in Chrome, but for the production build it is compiled away.

By default, the worker script will be emitted as a separate chunk in the production build. If you wish to inline the worker as base64 strings, add the `inline` query:

```js
import MyWorker from './worker?worker&inline'
```

## Build Optimizations

> Features listed below are automatically applied as part of the build process and there is no need for explicit configuration unless you want to disable them.

### Dynamic Import Polyfill

Vite uses ES dynamic import as code-splitting points. The generated code will also use dynamic imports to load the async chunks. However, native ESM dynamic imports support landed later than ESM via script tags and there is a browser support discrepancy between the two features. Vite automatically injects a light-weight [dynamic import polyfill](https://github.com/GoogleChromeLabs/dynamic-import-polyfill) to ease out that difference.

If you know you are only targeting browsers with native dynamic import support, you can explicitly disable this feature via [`build.polyfillDynamicImport`](/config/#build-polyfilldynamicimport).

### CSS Code Splitting

Vite automatically extracts the CSS used by modules in an async chunk and generate a separate file for it. The CSS file is automatically loaded via a `<link>` tag when the associated async chunk is loaded, and the async chunk is guaranteed to only be evaluated after the CSS is loaded to avoid [FOUC](https://en.wikipedia.org/wiki/Flash_of_unstyled_content#:~:text=A%20flash%20of%20unstyled%20content,before%20all%20information%20is%20retrieved.).

If you'd rather have all the CSS extracted into a single file, you can disable CSS code splitting by setting [`build.cssCodeSplit`](/config/#build-csscodesplit) to `false`.

### Preload Directives Generation

Vite automatically generates `<link rel="modulepreload">` directives for entry chunks and their direct imports in the built HTML.

### Async Chunk Loading Optimization

In real world applications, Rollup often generates "common" chunks - code that is shared between two or more other chunks. Combined with dynamic imports, it is quite common to have the following scenario:

![graph](/images/graph.png)

In the non-optimized scenarios, when async chunk `A` is imported, the browser will have to request and parse `A` before it can figure out that it also needs the common chunk `C`. This results in an extra network roundtrip:

```
Entry ---> A ---> C
```

Vite automatically rewrites code-split dynamic import calls with a preload step so that when `A` is requested, `C` is fetched **in parallel**:

```
Entry ---> (A + C)
```

It is possible for `C` to have further imports, which will result in even more roundtrips in the un-optimized scenario. Vite's optimization will trace all the direct imports to completely eliminate the roundtrips regardless of import depth.
