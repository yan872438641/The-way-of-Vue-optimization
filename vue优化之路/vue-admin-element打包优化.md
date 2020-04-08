# 一、配置CND加速

先找到 `vue.config.js`， 添加 `externals` 让 `webpack` 不打包 `vue` 和 `element`

```js
  configureWebpack: config => {
        config.externals = {
          'vue': 'Vue',
          'element-ui': 'ELEMENT'
        }
    }
```

然后在 chainWebpack 中配置第三方资源的`CDN`

```js
const cdn = {
  css: [
    'https://unpkg.com/element-ui/lib/theme-chalk/index.css'
  ],
  js: [
    'https://unpkg.com/vue/dist/vue.js',
    'https://unpkg.com/element-ui/lib/index.js'
  ]
}

config.plugin('html').tap(args => {
  args[0].cdn = cdn
  return args
})
```

再然后在 public/index.html 依次注入 css 和 js

```vue
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <meta name="renderer" content="webkit">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
    <link rel="icon" href="<%= BASE_URL %>favicon.ico">
    <title><%= webpackConfig.name %></title>
      <!-- 引入样式 -->
      <% for(var css of htmlWebpackPlugin.options.cdn.css) { %>
        <link rel="stylesheet" href="<%=css%>">
      <% } %>
  </head>
  <body>
    <!-- 引入JS -->
    <% for(var js of htmlWebpackPlugin.options.cdn.js) { %>
      <script src="<%=js%>"></script>
    <% } %>
    <div id="app"></div>
    <!-- built files will be auto injected -->
  </body>
</html>
```

# 二、配置gzip压缩

先安装插件

```vue
npm install compression-webpack-plugin -D
```

打开`vue.config.js` 文件找到 configureWebpack

```js
/**********  配置参数详解  *******/
# asset： 目标资源名称。 [file] 会被替换成原始资源。[path] 会被替换成原始资源的路径， [query] 会被替换成查询字符串。默认值是 "[path].gz[query]"。
# algorithm： 可以是 function(buf, callback) 或者字符串。对于字符串来说依照 zlib 的算法(或者 zopfli 的算法)。默认值是 "gzip"。
# test： 所有匹配该正则的资源都会被处理。默认值是全部资源。
# threshold： 只有大小大于该值的资源会被处理。单位是 bytes。默认值是 0。
# minRatio： 只有压缩率小于这个值的资源才会被处理。默认值是 0.8。

configureWebpack: config => {
    if (process.env.NODE_ENV === 'production') {
          const plugins = [];
          plugins.push(
              new CompressionWebpackPlugin({
                  filename: '[path].gz[query]',
                  algorithm: 'gzip',
                  test: new RegExp('\\.(' + productionGzipExtensions.join('|') + ')$'),
                  threshold: 0,
                  minRatio: 0.8
              })
          );
          config.plugins = [
              ...config.plugins,
              ...plugins
          ];
 	 }
  },
```

