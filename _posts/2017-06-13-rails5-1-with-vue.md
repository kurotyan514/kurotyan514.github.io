---
layout: single
title: rails5.1 with vue
excerpt: 在 rails 5.1 上使用 vue
tag:
  - Rails
  - vue
  - wbepack
---
# install
```bash
# install yarn
brew install yarn
# install ruby
rvm install 2.4.1
# install rails
gem install rails -v 5.1.1
# create project
rails new PROJECT_NAME --webpack=vue
cd PROJECT_NAME
# install vue
rails rails webpacker:install:vue

```
在跑　`bundle install` 的時候會出現下面的 warning
> The dependency tzinfo-data (>= 0) will be unused by any of the platforms Bundler is installing for. Bundler is installing for ruby but the dependency is only for x86-mingw32, x86-mswin32, x64-mingw32, java. To add those platforms to the bundle, run 'bundle lock --add-platform x86-mingw32 x86-mswin32 x64-mingw32 java'.

如果想要解決的話可以跑，詳細可以看這邊
["The dependency tzinfo-data will be unused" on Ubuntu](https://github.com/tzinfo/tzinfo-data/issues/12)
```bash
bundle lock --add-platform mingw, mswin, x64_mingw, jruby
```

# webpack-dev-server
在每一次修改vue頁面後會需要在 console 跑 `bin/webpack` 這樣才會重新 compile vue的頁面，但這樣其實很麻煩，所以我們可以透過 `webpack-dev-server` 的功能來自動做這一件事情節省時間。

要用 webpack-dev-server 也很簡單，只要再另開一個 terminal 跑 `./bin/webpack-dev-server` 就可以了，這樣就會在檔案儲存的同時broswer就會自動refresh頁面。

但如果不想再開一個 terminal 來跑 webpack-dev-server 還有另一個方法是在同一個 terminal 視窗裡 同時跑 webpack-dev-server & Rails server。

要達到這個目的首先先在gemfile加入 `foreman`
```ruby
group :development do
   # ...
+  gem 'foreman'
 end
```
接下來在 root 下建立兩個檔案 `Procfile` & `Procfile.dev`
```bash
# Procfile
web: bundle exec puma -p $PORT
```
```bash
# Procfile.dev
web: bundle exec rails s
# watcher: ./bin/webpack-watcher
webpacker: ./bin/webpack-dev-server
```
之後在 `bin` 底下建立一個叫 `server` 的檔案
```bash
#!/bin/bash -i
# bundle install
bundle exec foreman start -f Procfile.dev
```
然後在下 CLI 讓檔案可以被執行
```bash
chmod 777 bin/server
```
這樣所有的設定都完成了，原本是用 `rails s` 來建立 server 的，現在改用 `bin/server`
來跑，這樣就會同時跑 webpack-dev-server & Rails server。在上面的設定檔中有幾個是被註解起來的，如果有需要的話可以自行打開使用。

另外需要注意的地方是因為用 foreman 的關係所以 default 的 port 為 `5000`，要記得打 `localhost:5000` 才看得到畫面。

如果還是習慣用 3000 port 的話可以修改 Procfile.dev，這樣就可以用 3000 port 了。
```bash
# Procfile.dev
web: PORT=3000 bundle exec rails s
# watcher: ./bin/webpack-watcher
webpacker: ./bin/webpack-dev-server
```

# Runtime + Compiler vs. Runtime-only
vue2 的 builder 是使用 Runtime-only ， 當使用 `vue-loader` 或 `vueify` 把 template 放在 `*.vue` 檔案裡的話（也就是 [Single-File-Components
](https://vuejs.org/v2/guide/single-file-components.html)）會先把 .vue 檔案裡的 template 用 pre-compiled 的方式打包成 javaScript at build time，這樣在最後就不需要 compiler 來打包，官方是說 runtime-only builds 比 full builder 還要來得小跟快
而且是用 SFC(Single-File-Components) 的方式才支援 SSR(server-side-render)這樣就不用透過DOM API來做 render。

參考: [Runtime-Compiler-vs-Runtime-only](https://vuejs.org/v2/guide/installation.html#Runtime-Compiler-vs-Runtime-only)

BUT ! ! ! !

我還是個 vue 超新手，我只是想先跟著官方的 Hello World ~~飯粒~~ 實作啊，什麼.vue 檔案把 js, css, html 都寫在同一個檔案裡的 ~~一條龍~~ 方式還不習慣啊，所以你如果還是把 html 跟 js 拆開來寫像這樣的方式↓↓↓↓↓

```erb
 <!-- index.html.erb -->
<%= javascript_pack_tag "hello" %>
<div id="app">
  {% raw %}{{ message }}{% endraw %}
</div>
```
```js
// javaScript/packs/hello.js
import Vue from 'vue'
document.addEventListener("DOMContentLoaded", () => {
  new Vue({
    el: "#app",
    data: {
      message: 'HELLOOOOOOOOO!'
    }
  })
})
```
就會噴以下的錯誤
> Vue warn: You are using the runtime-only build of Vue where the template compiler is not available. Either pre-compile the templates into render functions, or use the compiler-included build.
(found in <Root>)

原因的話官方這邊也有說
> If you need to compile templates on the fly (e.g. passing a string to the template option, or mounting to an element using its in-DOM HTML as the template), you will need the compiler and thus the full build:

所以就是要去把 build 設成 full build，以下是作法

修改/config/webpack/shared.js，這邊會用 `vue.esm.js'` 是因為官方說如果是用 webpack2 或 rollup 建議用 ES Module 版本，詳細可以參考這個圖表[Explanation-of-Different-Builds](https://vuejs.org/v2/guide/installation.html#Explanation-of-Different-Builds)
```javascript
resolve: {
    extensions: settings.extensions,
    modules: [
      resolve(settings.source_path),
      'node_modules'
    ],
+    alias: {
+      vue: 'vue/dist/vue.esm.js'
+    }
  }
```

# Single-File-Components 設定 css
這是用 rails5.1+vue 所建立的 Single-File-Components 範例
```html
<!-- app.vue -->
<template>
  <div id="app">
    <p>{% raw %}{{ message }}{% endraw %}</p>
  </div>
</template>

<script>
export default {
  data: function () {
    return {
      message: "Hello Vue!!!!!!"
    }
  }
}
</script>

<style scoped>
p {
  font-size: 2em;
  text-align: center;
}
</style>
```
實際跑起來後應該會發現吃不到 css， 但如果給 style 加上 `lang="scss"` 就會 work
```html
<style lang="scss" scoped>
p {
  font-size: 2em;
  text-align: center;
}
</style>
```
問題主要是發生在 `/config/webpack/loaders/vue.js` 這個檔案，在原始的檔案裡並沒有 `css` & `style` 這個選項，所以當 style tag 上沒有加上 lang 選項並且給有支援的格式的話是不會吃到 css 的，照下面修改之後即使 style tag 沒有加上 lang 選項，或是指定成 css 都可以作用了。另外一點小提醒是如果要把 `lang="sass"` 的時候，記得要把 css 的 syntax也改一下嘿，不要像我傻傻的一樣沒改然後還一直卡在會什麼會噴錯的泥沼啊～。
```js
module.exports = {
  test: /.vue$/,
  loader: 'vue-loader',
  options: {
    extractCSS: true,
    loaders: {
      js: 'babel-loader',
      file: 'file-loader',
      scss: 'vue-style-loader!css-loader!postcss-loader!sass-loader',
      sass: 'vue-style-loader!css-loader!postcss-loader!sass-loader?indentedSyntax',
+     css: 'vue-style-loader!css-loader',
+     style: 'vue-style-loader'
    }
  }
}
```

最後要用也很簡單就是在 view 裡面用 `javascript_pack_tag` 這個 helper，後面是指定你的 js 檔案，這樣就會把 vue page 給 render 出來了。

# Referance
[動画付き】Rails 5.1で作るVue.jsアプリケーション ～Herokuデプロイからシステムテストまで～](http://qiita.com/jnchito/items/30ab14ebf29b945559f6)

[Webpack dev server](https://github.com/rails/webpacker#webpack-dev-server)

[Introducing Webpacker](https://medium.com/statuscode/introducing-webpacker-7136d66cddfb)

[vue-loader說明 gitbook](https://vue-loader.vuejs.org/)

[vus-眼藥水 2.0](https://blog.hinablue.me/vuejs-yan-yao-shui-2-0/)

[Vue2.x系のハマりどころ templateとコンパイラを完全解説するよ](https://aloerina01.github.io/javascript/vue/2017/03/08/1.html)

[Railsのフォームビルダーで生成したform要素をVueコンポーネント化する](http://qiita.com/kuroda@github/items/940338b5f6f46da2c5f1)
