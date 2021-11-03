重点： 如何优化，优化的哈需要配置哪些，工作原理，



# 常见的Loader

- `file-loader`：把⽂件输出到⼀个⽂件夹中，在代码中通过相对 URL 去引⽤输出的⽂件
- `url-loader`：和 file-loader 类似，但是能在⽂件很⼩的情况下以 base64 的⽅式把⽂件内容注⼊到代码中去
- `source-map-loader`：加载额外的 Source Map ⽂件，以⽅便断点调试
- `image-loader`：加载并且压缩图⽚⽂件
- `babel-loader`：把 ES6 转换成 ES5
- `css-loader`：加载 CSS，⽀持模块化、压缩、⽂件导⼊等特性
- `style-loader`：把 CSS 代码注⼊到 JavaScript 中，通过 DOM 操作去加载 CSS。
- `eslint-loader`：通过 ESLint 检查 JavaScript 代码



## file-loader和url-loader的区别


url-loader 会将引入的文件进行编码，生成 DataURL，相当于把文件翻译成了一串字符串，再把这个字符串打包到 JavaScript。

使用 base64 来加载图片也是有两面性的：

- 优点：节省请求，提高页面性能
- 缺点：增大本地文件大小，降低加载性能

所以我们得有取舍，只对部分小 size 的图片进行 base64 编码，其它的大图片还是发请求吧。

### 什么是 file-loader

在css文件中定义background的属性或者在html中引入image的src，我们知道在webpack打包后这些图片会打包至定义好的一个文件夹下，和开发时候的相对路径会不一样，这就会导致导入图片路径的错误。而file-loader正是为了解决此类问题而产生的，他修改打包后图片的储存路径，再根据配置修改我们引用的路径，使之对应引入

### 之间有什么联系呢？

url-loader内部封装了file-loader。url-loader不依赖于file-loader，即使用url-loader时，只需要安装url-loader即可，不需要安装file-loader。

通过上面的介绍，我们可以看到，url-loader工作分两种情况：

1. 文件大小小于limit参数，url-loader将会把文件转为DataURL；
2. 文件大小大于limit，url-loader会调用file-loader进行处理，参数也会直接传给file-loader。因此我们只需要安装url-loader即可



## style-loader

**用途：** 用于将`css`编译完成的样式，挂载到页面`style`标签上。**需要注意loader执行顺序，style-loader放到第一位，因为loader都是从下往上执行，最后全部编译完成挂载到style上**

**安装**

```javascript
cnpm i style-loader -D
复制代码
```

**配置**

**webpack.config.js**

```javascript
module.exports = {
    module: {
        rules: [
            {
                test: /\.css/,
                use: ["style-loader"]
            }
        ]
    }
}
复制代码
```

## css-loader

**用途：** 用于识别`.css`文件, 处理`css`必须配合`style-loader`共同使用，只安装`css-loader`样式不会生效。

**安装**

```javascript
cnpm i css-loader style-loader -D
复制代码
```

**配置**

**webpack.config.js**

```javascript
module.exports = {
    module: {
        rules: [
            {
                test: /\.css/,
                use: [
                    "style-loader",
                    "css-loader"
                ]
            }
        ]
    }
}
复制代码
```

## sass-loader

**用途：**`css`预处理器，我们在项目开发中经常会使用到的，编写`css`非常便捷，一个字 "棒"。

**安装**

```javascript
cnpm i sass-loader@5.0.0 node-sass -D
复制代码
```

**配置**

**index.js**

```javascript
import "index.scss"
复制代码
```

**index.scss**

```javascript
body {
    font-size: 18px;
    background: red;
}
复制代码
```

**webpack.config.js**

```javascript
module.exports = {
    module: {
        rules: [
            {
                test: /\.scss$/,
                use: [
                    "style-loader",
                    "css-loader",
                    "sass-loader"
                ],
                include: /src/, 
            },
        ]
    }
}
复制代码
```

## postcss-loader

**用途：** 用于补充css样式各种浏览器内核前缀，太方便了，不用我们手动写啦。

**安装**

```javascript
cnpm i postcss-loader autoprefixer -D
复制代码
```

**配置**

**postcss.config.js**

如果不写在该文件呢，也可以写在`postcss-loader`的`options`里面，写在该文件跟写在那里是同等的

```javascript
module.exports = {
    plugins: [
        require("autoprefixer")({
            overrideBrowserslist: ["> 1%", "last 3 versions", "ie 8"]
        })
    ]
}
复制代码
```

| 属性            | 描述                                                  |
| --------------- | ----------------------------------------------------- |
| > 1%            | 全球超过1%人使用的浏览器                              |
| > 5% in CN      | 指定国家使用率覆盖                                    |
| last 2 versions | 所有浏览器兼容到最后两个版本根据CanIUse.com追踪的版本 |
| Firefox ESR     | 火狐最新版本                                          |
| Firefox > 20    | 指定浏览器的版本范围                                  |
| not ie <=8      | 方向排除部分版本                                      |
| Firefox 12.1    | 指定浏览器的兼容到指定版本                            |

**webpack.config.js**

```javascript
module.exports = {
    module: {
        rules: [
            {
                test: /\.scss$/,
                use: [
                    "style-loader",
                    "css-loader",
                    "sass-loader",
            		"postcss-loader"
                ],
                include: /src/, 
            },
        ]
    }
}
复制代码
```

## babel-loader

**用途**： 将Es6+ 语法转换为Es5语法。

**安装**

```javascript
cnpm i babel-loader @babel/core @babel/preset-env -D
复制代码
```

- babel-loader 这是使babel和webpack协同工作的模块
- @bable/core 这是babel编译器核心模块
- @babel/preset-env 这是babel官方推荐的预置器，可根据用户的环境自动添加所需的插件和补丁来编译Es6代码

**配置**

**webpack.config.js**

```javascript
module.exports = {
    module: {
        rules: [
            {
                test: /\.js$/,
                use: {
                    loader: "babel-loader",
                    options: {
                        presets: [
                            ['@babel/preset-env', { targets: "defaults"}]
                        ]
                    }
                }
            },
        ]
    }
}
复制代码
```

## ts-loader

**用途：** 用于配置项目typescript

**安装**

```javascript
cnpm i ts-loader typescript -D
复制代码
```

**配置**

**webpack.config.js**

当前配置`ts-loader`不会生效，只是会编译识别`.ts`文件, 主要配置文件在`tsconfig.json`里

```javascript
module.exports = {
    module: {
        rules: [
            {
                test: /\.ts$/,
                use: "ts-loader"
            },
        ]
    }
}
复制代码
```

**tsconfig.json**

```javascript
{
    "compilerOptions": {
      "declaration": true,
      "declarationMap": true, // 开启map文件调试使用
      "sourceMap": true,
      "target": "es5", // 转换为Es5语法
    }
}  
复制代码
```

**webpack.config.js**

```javascript
module.exports = {
    entry: "./src/index.ts",
    output: {
        path: __dirname + "/dist",
        filename: "index.js",
    },
    module: {
        rules: [
            {
                {
                	test: /\.ts$/,
                	use: "ts-loader",
            	}
            }
        ]
    }
}
复制代码
```

## html-loader

**用途：** 我们有时候想引入一个`html`页面代码片段赋值给`DOM`元素内容使用，这时就用到`html-loader`

**安装**

```javascript
cnpm i html-loader@0.5.5 -D
复制代码
```

建议安装低版本，高版本可能会不兼容导致报错。我这里安装的是0.5.5版本

**配置**

**index.js**

```javascript
import Content from "../template.html"

document.body.innerHTML = Content
复制代码
```

**webpack.config.js**

```javascript
module.exports = {
    module: {
        rules: [
            {
                test: /\.html$/,
                use: "html-loader"
            }
        ]
    }
}
复制代码
```

## file-loader

**用途：** 用于处理文件类型资源，如`jpg`，`png`等图片。返回值为`publicPath`为准。

**安装**

```javascript
cnpm i file-loader -D
复制代码
```

**配置**

**index.js**

```javascript
import img from "./pic.png"
console.log(img) // https://www.baidu.com/pic_600eca23.png
复制代码
```

**webpack.config.js**

```javascript
module.exports = {
    module: {
        rules: [
            {
                test: /\.(png|jpg|jpeg)$/,
                use: [
                    {
                        loader: "file-loader",
                        options: {
                            name: "[name]_[hash:8].[ext]",
                            publicPath: "https://www.baidu.com" 
                        }
                    }
                ]
            }
        ]
    }
}
复制代码
```

## url-loader

**用途：** `url-loader`也是处理图片类型资源，只不过它与`file-loader`有一点不同，`url-loader`可以设置一个根据图片大小进行不同的操作，如果该图片大小大于指定的大小，则将图片进行打包资源，否则将图片转换为`base64`字符串合并到`js`文件里

**安装**

```javascript
cnpm i url-loader -D
复制代码
```

**配置**

**index.js**

```javascript
import img from "./pic.png"
复制代码
```

**webpack.config.js**

```javascript
module.exports = {
    module: {
        rules: [
            {
                test: /\.(png|jpg|jpeg)$/,
                use: [
                    {
                        loader: "url-loader",
                        options: {
                            name: "[name]_[hash:8].[ext]",
                            limit: 10240, // 这里单位为(b) 10240 => 10kb
                            // 这里如果小于10kb则转换为base64打包进js文件，如果大于10kb则打包到dist目录
                        }
                    }
                ]
            }
        ]
    }
}
复制代码
```

## html-withimg-loader

**用途：** 我们在编译图片时，都是使用`file-loader`和`url-loader`，这两个`loader`都是查找`js`文件里的相关图片资源，但是`html`里面的文件不会查找所以我们`html`里的图片也想打包进去，这时使用`html-withimg-loader`

**安装**

```javascript
cnpm i html-withimg-loader -D
复制代码
```

**配置**

**index.html**

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>首页</title>
</head>
<body>
    <h4>我蛙人</h4>
    <img src="./src/img/pic.jpg" alt="">
</body>
</html>
复制代码
```

**webpack.config.js**

如果打包出现img的src路径为`[Object Module]`，解决方案有

- 将file-loader降级到4.2.0
- 修改options参数esModule为false

```javascript
module.exports = {
    module: {
        rules: [
            {
                test: /\.(png|jpg|jpeg)$/,
                use: {
                    loader: "file-loader",
                    options: {
                        name: "[name]_[hash:8].[ext]",
                        publicPath: "http://www.baidu.com",
                        esModule: false
                    }
                }
            },
            {
                test: /\.(png|jpeg|jpg)/,
                use: "html-withimg-loader"
            }
        ]
    }
}
复制代码
```

## vue-loader

**用途：** 用于编译`.vue`文件，如我们自己搭建`vue`项目就可以使用`vue-loader`, 以下简单了解一下，这里就不多赘述啦。

**安装**

```javascript
cnpm i vue-loader@15.7.0 vue vue-template-compiler -D
复制代码
```

- vue-loader 用于识别.vue文件
- vue 不用多说，识别支持vue语法
- vue-template-compiler  语法模板渲染引擎 {{}} `template`、 `script`、 `style`

**配置**

**main.js**

```javascript
import App from "./index.vue"
import Vue from 'Vue'
new Vue({
    el: "#app",
    render: h => h(App) 
})
复制代码
```

**index.vue**

```javascript
<template>
  <div class="index">
    {{ msg }}
  </div>
</template>

<script>
export default {
 name: 'index',
  data() {
    return {
        msg: "hello 蛙人"
    }
  },
  created() {},
  components: {},
  watch: {},
  methods: {}
}
</script>
<style scoped>

</style>
复制代码
```

**webpack.config.js**

```javascript
const VueLoaderPlugin = require('vue-loader/lib/plugin')
module.exports = {
    entry: "./src/main.js",
    output: {
        path: __dirname + "/dist",
        filename: "index.js",
    },
    module: {
        rules: [
            {
                test: /\.vue$/,
                use: "vue-loader"
            }
        ]
    },
    plugins: [
        new VueLoaderPlugin()
    ]
}
复制代码
```

## eslint-loader

**用途：** 用于检查代码是否符合规范，是否存在语法错误。

**安装**

```javascript
cnpm i eslint-loader eslint -D
复制代码
```

**配置**

**index.ts**

```javascript
var abc:any = 123;
console.log(abc)
复制代码
```

**.eslintrc.js**

这里简单写几个规则

```javascript
module.exports = {
    root: true,   
    env: {
        browser: true,
    },
    rules: {
        "no-alert": 0, // 禁止使用alert
        "indent": [2, 4], // 缩进风格
        "no-unused-vars": "error" // 变量声明必须使用
    }
}
复制代码
```

**webpack.config.js**

```javascript
module.exports = {
    module: {
        rules: [
            {
                test: /\.ts$/,
                use: ["eslint-loader", "ts-loader"],
                enforce: "pre",
                exclude: /node_modules/
            },
            {
                test: /\.ts$/,
                use: "ts-loader",
                exclude: /node_modules/
            }
        ]
    }
}
```



