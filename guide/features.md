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

Vite provides first-class Vue support:

- Vue 3 SFC support via [@vitejs/plugin-vue](https://github.com/vitejs/vite/tree/main/packages/plugin-vue)
- Vue 3 JSX support via [@vitejs/plugin-vue-jsx](https://github.com/vitejs/vite/tree/main/packages/plugin-vue-jsx)
- Vue 2 support via [underfin/vite-plugin-vue2](https://github.com/underfin/vite-plugin-vue2)

## JSX

`.jsx` and `.tsx` files are also supported out of the box. JSX transpilation is also handled via [ESBuild](https://esbuild.github.io), and defaults to the React 16 flavor. React 17 style JSX support in ESBuild is tracked [here](https://github.com/evanw/esbuild/issues/334).

Vue users should use the official [@vitejs/plugin-vue-jsx](https://github.com/vitejs/vite/tree/main/packages/plugin-vue-jsx) plugin, which provides Vue 3 specific features including HMR, global component resolving, directives and slots.

If not using JSX with React or Vue, custom `jsxFactory` and `jsxFragment` can be configured using the [`esbuild` option](/config/#esbuild). For example for Preact:

```js
// vite.config.js
export default {
  esbuild: {
    jsxFactory: 'h',
    jsxFragment: 'Fragment'
  }
}
```

More details in [ESBuild docs](https://esbuild.github.io/content-types/#jsx).

You can inject the JSX helpers using `jsxInject` (which is a Vite-only option) to avoid manual imports:

```js
// vite.config.js
export default {
  esbuild: {
    jsxInject: `import React from 'react'`
  }
}
```

## CSS

Importing `.css` files will inject its content to the page via a `<style>` tag with HMR support. You can also retrieve the processed CSS as a string as the module's default export.

### `@import` Inlining and Rebasing

Vite is pre-configured to support CSS `@import` inlining via `postcss-import`. Vite aliases are also respected for CSS `@import`. In addition, all CSS `url()` references, even if the imported files are in different directories, are always automatically rebased to ensure correctness.

`@import` aliases and URL rebasing are also supported for Sass and Less files (see [CSS Pre-processors](#css-pre-processors)).

### PostCSS

If the project contains valid PostCSS config (any format supported by [postcss-load-config](https://github.com/postcss/postcss-load-config), e.g. `postcss.config.js`), it will be automatically applied to all imported CSS.

### CSS Modules

Any CSS file ending with `.module.css` is considered a [CSS modules file](https://github.com/css-modules/css-modules). Importing such a file will return the corresponding module object:

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

CSS modules behavior can be configured via the [`css.modules` option](/config/#css-modules).

If `css.modules.localsConvention` is set to enable camelCase locals (e.g. `localsConvention: 'camelCaseOnly'`), you can also use named imports:

```js
// .apply-color -> applyColor
import { applyColor } from './example.module.css'
document.getElementById('foo').className = applyColor
```

### CSS Pre-processors

Because Vite targets modern browsers only, it is recommended to use native CSS variables with PostCSS plugins that implement CSSWG drafts (e.g. [postcss-nesting](https://github.com/jonathantneal/postcss-nesting)) and author plain, future-standards-compliant CSS.

That said, Vite does provide built-in support for `.scss`, `.sass`, `.less`, `.styl` and `.stylus` files. There is no need to install Vite-specific plugins for them, but the corresponding pre-processor itself must be installed:

```bash
# .scss and .sass
npm install -D sass

# .less
npm install -D less

# .styl and .stylus
npm install -D stylus
```

If using Vue single file components, this also automatically enables `<style lang="sass">` et al.

Vite improves `@import` resolving for Sass and Less so that Vite aliases are also respected. In addition, relative `url()` references inside imported Sass/Less files that are in different directories from the root file are also automatically rebased to ensure correctness.

`@import` alias and url rebasing are not supported for Stylus due to its API constraints.

You can also use CSS modules combined with pre-processors by prepending `.module` to the file extension, for example `style.module.scss`.

## Static Assets

Importing a static asset will return the resolved public URL when it is served:

```js
import imgUrl from './img.png'
document.getElementById('hero-img').src = imgUrl
```

Special queries can modify how assets are loaded:

```js
// Explicitly load assets as URL
import assetAsURL from './asset.js?url'
```

```js
// Load assets as strings
import assetAsString from './shader.glsl?raw'
```

```js
// Load Web Workers
import Worker from './worker.js?worker'
```

```js
// Web Workers inlined as base64 strings at build time
import InlineWorker from './worker.js?worker&inline'
```

More details in [Static Asset Handling](./assets).

## JSON

JSON files can be directly imported - named imports are also supported:

```js
// import the entire object
import json from './example.json'
// import a root field as named exports - helps with treeshaking!
import { field } from './example.json'
```

## Glob Import

Vite supports importing multiple modules from the file system via the special `import.meta.glob` function:

```js
const modules = import.meta.glob('./dir/*.js')
```

The above will be transformed into the following:

```js
// code produced by vite
const modules = {
  './dir/foo.js': () => import('./dir/foo.js'),
  './dir/bar.js': () => import('./dir/bar.js')
}
```

You can then iterate over the keys of the `modules` object to access the corresponding modules:

```js
for (const path in modules) {
  modules[path]().then((mod) => {
    console.log(path, mod)
  })
}
```

Matched files are by default lazy loaded via dynamic import and will be split into separate chunks during build. If you'd rather import all the modules directly (e.g. relying on side-effects in these modules to be applied first), you can use `import.meta.globEager` instead:

```js
const modules = import.meta.globEager('./dir/*.js')
```

The above will be transformed into the following:

```js
// code produced by vite
import * as __glob__0_0 from './dir/foo.js'
import * as __glob__0_1 from './dir/bar.js'
const modules = {
  './dir/foo.js': __glob__0_0,
  './dir/bar.js': __glob__0_1
}
```

Note that:

- This is a Vite-only feature and is not a web or ES standard.
- The glob patterns are treated like import specifiers: they must be either relative (start with `./`) or absolute (start with `/`, resolved relative to project root). Globbing from dependencies is not supported.
- The glob matching is done via `fast-glob` - check out its documentation for [supported glob patterns](https://github.com/mrmlnc/fast-glob#pattern-syntax).

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
