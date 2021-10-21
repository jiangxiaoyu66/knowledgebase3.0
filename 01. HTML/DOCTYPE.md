## DOCTYPE

### DOCTYPE是什么

`DOCTYPE`是`document type`(文档类型)的简写，在`web`设计中用来说明使用的`XHTML`或者`HTML`是什么版本。
(`HTML5` 不基于 `SGML`，所以不需要引用 `DTD`。)



### 怎么用

`<!DOCTYPE html>`声明必须是 HTML 文档的第一行，位于 `<html>` 标签之前。

```html
<!DOCTYPE html>
<html>
    <head>
        <title></title>
    </head>
    <body>

    </body>
</html>
```





### 扩展

#### DTD

`DTD`叫文档类型定义，里面包含了文档的规则，浏览器就根据定义的`DTD`来解释你页面的标识，并展现出来。

`DTD`规定了标记语言的规则，这样浏览器才能正确地呈现内容

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Frameset//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-frameset.dtd">
```



### 参考

[**DOCTYPE**](https://github.com/WindrunnerMax/EveryDay/blob/master/HTML/DOCTYPE.md)

[w3c-DTD](https://www.w3school.com.cn/dtd/dtd_intro.asp)