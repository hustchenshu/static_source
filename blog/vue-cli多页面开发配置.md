# vue-cli多页面开发配置
vue的脚手架工具为我们提供了不少的方便，特别是对于单页面开发，可以无视什么webpack、eslint等等对新手不友好的配置文件，但是如果要想在你的网站开发中使用两个或者多个页面，比如搭建一个博客平台，自然需要有前台展示和后台管理系统，那么至少需要两个页面，这个时候就需要好好好研究一番这些配置文件了。

##　1  . vue脚手架webpack模板架构
通过vue脚手架可以很简单的建立起一个demo，这对于新手是十分友好的，不用去管什么配置，直接**npm run dev**就可看到效果，我们要做的就只收在**src/components**下添加自己的组件，然后稍加改动app.vue文件，使得入口文件渲染自己编写的组件即可。
既然要开发多页面，自然要先弄清楚单页面的流程：
### 1.1 单页面详解
我们知道vue脚手架当中是使用webpack进行vue的解析编译打包管理工作，那么自然有webpack的配置文件，这些文件就是**build**文件夹下的3个文件：**webpack.base.conf.js、webpack.dev.conf.js、webpack.prod.conf.js**,

+ webpack.base.conf.js

webpack.base.conf.js文件当中是基本配置，代码如下：
```javascript
var path = require('path')
var utils = require('./utils')
var config = require('../config')
var vueLoaderConfig = require('./vue-loader.conf')

function resolve (dir) {
  return path.join(__dirname, '..', dir)
}

module.exports = {
   // 入口文件
  entry: {
    app: './src/main.js',
  },
  // 输出文件
  output: {
    path: config.build.assetsRoot,
    filename: '[name].js',
    publicPath: process.env.NODE_ENV === 'production'
      ? config.build.assetsPublicPath
      : config.dev.assetsPublicPath
  },
  resolve: {
    extensions: ['.js', '.vue', '.json'],
    alias: {
      'vue$': 'vue/dist/vue.esm.js',
      '@': resolve('src')
    }
  },
  // 解析配置
  module: {
    rules: [
      {
        test: /\.(js|vue)$/,
        loader: 'eslint-loader',
        enforce: 'pre',
        include: [resolve('src'), resolve('test')],
        options: {
          formatter: require('eslint-friendly-formatter')
        }
      },
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: vueLoaderConfig
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        include: [resolve('src'), resolve('test')]
      },
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('img/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
    ]
  }
}
```
**可以看出这个文件主要配置了工程的入口文件、输出文件和相关文件的loader规则**

+ webpack.dev.conf.js

再看webpack.dev.conf.js文件,顾名思义，这个文件配置了开发环境下的相关信息如下：
```javascript
var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')

// add hot-reload related code to entry chunks
Object.keys(baseWebpackConfig.entry).forEach(function (name) {
  baseWebpackConfig.entry[name] = ['./build/dev-client'].concat(baseWebpackConfig.entry[name])
})

module.exports = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({ sourceMap: config.dev.cssSourceMap })
  },
  // cheap-module-eval-source-map is faster for development
  devtool: '#cheap-module-eval-source-map',
  plugins: [
    new webpack.DefinePlugin({
      'process.env': config.dev.env
    }),
    // https://github.com/glenjamin/webpack-hot-middleware#installation--usage
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoEmitOnErrorsPlugin(),
    // https://github.com/ampedandwired/html-webpack-plugin
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      chunks: ['app'],
      inject: true
    })
    new FriendlyErrorsPlugin()
  ]
})

```

**上面的代码中首先使用的webpack-merge包将基本配置文件webpack.base.conf.js和当前文件配置文件进行了合并，然后使用了打包插件HtmlWebpackPlugin来对工程打包,如下代码：**
```JavaScript
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      chunks: ['app'],
      inject: true
    })
```
**当中分别配置了html打包的输出的文件名，使用的模板，其中重要的是chunks这一个配置项，对应于webpack.base.conf.js当中的entry名称，在单页面时可以省略，而在多页面情况下必须指定。**

+ webpack.prod.conf.js

最后看webpack.base.conf.js配置文件，对应的，这个文件应该就是生成静态资源时使用的配置项，也就是运行命令**npm run build**时使用的配置，截取重要代码如下：
```javascript
var webpackConfig = merge(baseWebpackConfig, {
    module: {
        ...
    },

    ....
    plugins: [
        
        ...

        new HtmlWebpackPlugin({
          filename: config.build.index,
          template: 'index.html',
          chunks: ['manifest','vendor','app'],
          inject: true,
          minify: {
            removeComments: true,
            collapseWhitespace: true,
            removeAttributeQuotes: true
            // more options:
            // https://github.com/kangax/html-minifier#options-quick-reference
          },
          // necessary to consistently work with multiple chunks via CommonsChunkPlugin
          chunksSortMode: 'dependency'
        }),

        ...
    ],
    ...
}
```

**同样使用了merge来合并了基础配置，在plugins当中使用了HtmlWebpackPlugin打包，只是配置项略微有些不同，具体配置项可以查阅[html-webpack-plugin](https://github.com/ampedandwired/html-webpack-plugin),当然，这里的输出文件是从文件/config
/index.js当中读取获得的，这部分以后再说。**

## 2.修改配置文件，生成多页面

### 2.1 准备所需文件
首先根据之前的单页面配置可以知道，每个页面需要有自己的入口文件，模板、输入输出文件配置等，所以第一步：创建所需文件，在/src文件夹下面建立文件，分别为admin-main.js和Admin.vue分别对应的是后台管理页面的入口文件和该页面的最高组件，还需要在/src/router文件夹下面建立admin页面的路由文件admin.js[内容参照/src/router.index.js],最后在根目录下新建admin页面的html模板admin.html【内容参照/index.html】

完成后的diamante结构如下：

![avatar](https://raw.githubusercontent.com/hustchenshu/static_source/master/blog/images/muti-1.png)

### 2.2 修改配置文件

首先是 webpack.base.conf.js文件当中增加入口文件：
```javascript 
...
module.exports = {
  entry: {
    app: './src/main.js',
    //新增admin入口文件
    admin: './src/admin-main.js'
  },
  ...
}
...
```
然后需要去webpack.dev.conf.js 中添加打包插件
```javascript
...
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      chunks: ['app'],
      inject: true
    }),
    // 新添加的admin打包插件
    new HtmlWebpackPlugin({
      filename: 'admin.html',
      template: 'admin.html',
      chunks: ['admin'],// 这里对应webpack.base.conf.js当中的entry定义的名称
      inject: true
    }),
...
```
同样的修改webpack.prod.conf.js当中的插件
```javascript   
...
    new HtmlWebpackPlugin({
      filename: config.build.index,
      template: 'index.html',
      chunks: ['manifest','vendor','app'],
      inject: true,
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
        // more options:
        // https://github.com/kangax/html-minifier#options-quick-reference
      },
      // necessary to consistently work with multiple chunks via CommonsChunkPlugin
      chunksSortMode: 'dependency'
    }),

    // 新添加的admin打包插件
    new HtmlWebpackPlugin({
      filename: config.build.admin,
      template: 'admin.html',
      inject: true,
      chunks: ['manifest','vendor','admin'],
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
        // more options:
        // https://github.com/kangax/html-minifier#options-quick-reference
      },
      // necessary to consistently work with multiple chunks via CommonsChunkPlugin
      chunksSortMode: 'dependency'
    }),
...
```
由于这里的输出文件由/config/index.js来定义，因此还需要在/config/index.js文件当中加入输出文件路径，如下：

```javascript   
...
    env: require('./prod.env'),
    // index: path.resolve(__dirname, '../dist/index.html'),
    // assetsRoot: path.resolve(__dirname, '../dist'),
    index: path.resolve(__dirname, '../../myserver/public/pages/index.html'),
    //  新添加的admin输出文件
    admin: path.resolve(__dirname, '../../myserver/public/pages/admin.html'),
    assetsRoot: path.resolve(__dirname, '../../myserver/public/pages'),
    assetsSubDirectory: 'static',
    assetsPublicPath: '/',
...
```
此外，为了admin.html能够从当前已有的默认界面上成功跳转，我们还需要修改build/dev-server.js当中的代码，这里假设我们需要在用户请求localhost:port/admin的时候返回生成的admin.html页面，那么配置如下：
```javascript
...
// handle fallback for HTML5 history API
// 这里是原有代码
// app.use(require('connect-history-api-fallback')())
// 修改成如下
var history = require('connect-history-api-fallback')
app.use(history ({
  // index: '/admin.html',
  rewrites: [
    { 
      from: '/admin', 
      to: function(context) {
        if(context.parsedUrl.pathname === '/admin' || context.parsedUrl.pathname === '/admin/'){
          return '/admin.html'
        }
        return context.parsedUrl.pathname;
      }
      // 将/admin的请求都基于'/admin.html'，而不是默认的index.html
    }
  ]
}))
...
```
这样，如果我们在原有默认界面当中的组件当中的template当中加入a标签
```
      <a href="/admin/#/addBlog">添加博客</a>
```
那么该链接将跳转的admin.html页面，

效果查看[HustChenShu](http://www.yyxyzclass.cn/),

[项目源代码](https://github.com/hustchenshu/personal_website)




