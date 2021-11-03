## 常用插件

### html-webpack-plugin

作用一是创建HTML页面文件到你的输出目录，作用二是将webpack打包后的chunk自动引入到这个HTML中

**用法**

```js
const HtmlPlugin = require('html-webpack-plugin')
...
// 单页
new HtmlPlugin({
    filename: 'index.html',
    template: 'src/index.html'
}
复制代码
```

template 是模板文件的地址，filename 是根据模板文件生成的html的文件名

如果是多个html页面的话，就需要多次实例化HtmlPlugin。比如现在有index.html和login.html两个页面：

```js
{
    entry: {
        index: './src/index.js',
        login: './src/login.js'
    },
    plugins: [
        new HtmlPlugin({
            filename: 'index.html',
            template: 'src/index.html',
            excludeChunks: ['login'],
            chunksSortMode: 'dependency'
        },
        new HtmlPlugin({
            filename: 'login.html',
            template: 'src/login.html',
            excludeChunks: ['index'],
            chunksSortMode: 'dependency'
        }
    ]
}
复制代码
```

### clean-webpack-plugin

清空文件夹插件,每次打包时先清空output文件夹。 使用

```js
//引入
const CleanWebpackPlugin = require('clean-webpack-plugin');
//清空dist文件夹
plugins: [
  new CleanWebpackPlugin(['dist'])
]
复制代码
```

### uglifyjs-webpack-plugin

通过UglifyES压缩JS代码

```js
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')
// ...
optimization: {
    minimizer: [
        new UglifyJsPlugin({
            // 开启多线程并行
            parallel: true
        })
    ]
},
复制代码
```

### terser-webpack-plugin

uglifyjs-webpack-plugin 不支持ES6语法，terser 支持对ES6进行压缩。

```js
import * as TerserPlugin from 'terser-webpack-plugin'
// const TerserPlugin = require('terser-webpack-plugin')
optimization: {
    minimizer: [
        new TerserPlugin()
    ],
    noEmitOnErrors: true
}
复制代码
```

### webpack.DefinePlugin

DefinePlugin 允许创建一个在编译时可以配置的全局常量

```js
new webpack.DefinePlugin({
    'process.env': JSON.stringify({
        NODE_ENV: process.env.NODE_ENV,
        ENV: 't1'
    })
}),
复制代码
```

### mini-css-extract-plugin

将CSS提取为独立的文件的插件，对每个包含css的js文件都会创建一个CSS文件，支持按需加载css和sourceMap。
 webpack 4.0以前，我们通过extract-text-webpack-plugin插件，把css样式从js文件中提取到单独的css文件中，webpack 4.0以后，官方推荐使用mini-css-extract-plugin插件来打包css文件。

```js
import * as MiniCssExtractPlugin from 'mini-css-extract-plugin'
module.exports = {
    plugins: [
        new MiniCssExtractPlugin({
            // 类似 webpackOptions.output里面的配置 可以忽略
            filename: '[name].css',
            chunkFilename: '[id].css',
        }),
    ],
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    {
                        loader: MiniCssExtractPlugin.loader,
                        options: {
                            // 这里可以指定一个 publicPath
                            // 默认使用 webpackOptions.output中的publicPath
                            publicPath: '../'
                        },
                    },
                    'css-loader',
                ],
            }
        ]
    }
}
复制代码
```

### optimize-css-assets-webpack-plugin

css压缩，主要使用 cssnano 压缩器. [github.com/cssnano/css…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fcssnano%2Fcssnano)

```js
import * as OptimizeCSSAssetsPlugin from 'optimize-css-assets-webpack-plugin'
optimization: {
    minimizer: [
        new TerserPlugin(),
        new OptimizeCSSAssetsPlugin()
    ],
    noEmitOnErrors: true
}
复制代码
```

### html-webpack-include-assets-plugin

将js css资源添加到html中 扩展html插件的功能。 先已改名为**html-webpack-tags-plugin**。一般可以结合webpack.DllReferencePlugin来把公共JS代码提前打包，提升打包速度。

```js
import * as HtmlWebpackIncludeAssetsPlugin from 'html-webpack-include-assets-plugin'
plugins: [
    new webpack.DllReferencePlugin({
        context: appPath,
        manifest: require(`${outputLibPath}/react-manifest.json`)
    }),
    new webpack.DllReferencePlugin({
        context: appPath,
        manifest: require(`${outputLibPath}/common-manifest.json`)
    }),
    new HtmlWebpackIncludeAssetsPlugin({
        assets: [{ path: 'lib', glob: '*.dll.js', globPath: outputLibPath }],
        append: false
    })
]
复制代码
```

### webpack-bundle-analyzer

编译模块分析插件，生成代码分析报告，帮助提升代码质量和网站性能。

```js
import { BundleAnalyzerPlugin } from 'webpack-bundle-analyzer'
module.exports={
    plugins: [
        new BundleAnalyzerPlugin()  // 使用默认配置
        // 默认配置的具体配置项
        // new BundleAnalyzerPlugin({
        //   analyzerMode: 'server',
        //   analyzerHost: '127.0.0.1',
        //   analyzerPort: '8888',
        //   reportFilename: 'report.html',
        //   defaultSizes: 'parsed',
        //   openAnalyzer: true,
        //   generateStatsFile: false,
        //   statsFilename: 'stats.json',
        //   statsOptions: null,
        //   excludeAssets: null,
        //   logLevel: info
        // })
    ]
}
复制代码
```

### DllPlugin && DllReferencePlugin && autodll-webpack-plugin

编译速度优化。

- dllPlugin将模块预先编译，DllReferencePlugin 将预先编译好的模块关联到当前编译中，当 webpack 解析到这些模块时，会直接使用预先编译好的模块。
- autodll-webpack-plugin 相当于 dllPlugin 和 DllReferencePlugin 的简化版，其实本质也是使用 dllPlugin && DllReferencePlugin，它会在第一次编译的时候将配置好的需要预先编译的模块编译在缓存中，第二次编译的时候，解析到这些模块就直接使用缓存，而不是去编译这些模块

#### dllPlugin 基本用法：

```js
...
entry: {
    react: [
        'react',
        'react-dom',
        'react-router-dom',
        'prop-types',
        'classnames'
    ],
    common: ['axios', 'redux']
},

output: {
    path: path.join(__dirname, 'dist'),
    filename: 'lib/[name]_[hash:4].dll.js',
    library: '[name]_[hash:4]'
},
...
plugins: [
    new CleanWebpackPlugin(),
    new webpack.DllPlugin({
        context: __dirname,
        path: path.join(__dirname, 'dist/lib', '[name]-manifest.json'),
        name: '[name]_[hash:4]'
    })
]
复制代码
```

#### DllReferencePlugin 基本用法：

```js
import * as HtmlWebpackIncludeAssetsPlugin from 'html-webpack-include-assets-plugin'
new webpack.DllReferencePlugin({
    context: __dirname,
    manifest: require(`${outputLibPath}/react-manifest.json`)
}),
new webpack.DllReferencePlugin({
    context: __dirname,
    manifest: require(`${outputLibPath}/common-manifest.json`)
}),
new HtmlWebpackIncludeAssetsPlugin({
    assets: [{ path: 'lib', glob: '*.dll.js', globPath: outputLibPath }],
    append: false
})
复制代码
```

#### autodll-webpack-plugin 基本用法：

```js
import * as AutoDllPlugin from 'autodll-webpack-plugin'
plugins: [
    new AutoDllPlugin({
        inject: true, // 与 html-webpack-plugin 结合使用，注入html中
        filename: '[name].js',
        entry: {
            vendor: [
                'react',
                'react-dom'
            ]
        }
    })
]
```


作者：JoeyZ50872
链接：https://juejin.cn/post/6875510964062519304
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。