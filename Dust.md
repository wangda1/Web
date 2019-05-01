# Dust

## 介绍

Dust出现的原因：

随着AJAX的发展，不刷新页面而动态更新内容的需求越来越多，而在浏览器与服务器的交互过程中用的较多的是JSON数据，
前端接收到JSON数据后，需要经过处理才能够按要求显示，而传统的方式是采用Javascript的方法，这种方式比较繁琐，
且代码可读性差，因此出现了Dust.js作为前端的模板。

```json
{
    "title": "Famous People",
    "names": [{"name": "Larry"},{"name": "Curly"},{"name": "Moe"}]
}

```
要把它渲染成一个HTML列表如下：
```html
Famous People
<ul>
　　<li>Larry</li>
　　<li>Curly</li>
　　<li>Moe</li>
</ul>
```
使用纯粹的Javascript是这样的，
```javascript
var result = people.title + '\n';
result += '<ul>' + '\n';
for (var i = 0; i < people.names.length; i++) {
	result += '<li>' + people.names[i].name + '</li>' + '\n';
}
result += '</ul>'
```
使用Dust是这样的，只需要一个模板：
```dust
{title}
<ul>
{#names}
　　<li>{name}</li>{~n}
{/names}
</ul>
```
然后将`people`传给编译好的模板即可生成所需要的结果。

## 应用
Dust是编译型模板，意味着要应用模板则需要先将代码可执行化，即编译成可执行的代码，这种编译型的方式很明显会带来解析速度的提升，大概和正则表达式的编译有相同点吧。

dust库有两种发行版本：
- dust-core-2.0.2.js 核心版本，只包括解析模板相关的代码
- dust-full-2.0.2.js 完全版本，包括编译器

```dust
var compiled = dust.compile("Hello {name}!","intro");
dust.loadsource(compiled);
dust.render("intro",{"name" : "Fred"},function(err,out) {
    console.log(out);
});

```
- dust.compile()，编译；para1: 模板字符串，para2: 模板名；return: 编译完成的可执行代码字符串；
- dust.loadsource(),注册；para1: 可执行代码字符串，将模板代码注册到dust中；
- dust.render(),渲染：para1: 模板名，para2: JSON对象，para3: 回调函数，其中回调函数包含：para1: 错误信息，para2: 渲染结果字符串。


## 语法简介
[Dust在线测试器](http://www.dustjs.com/test/test.html)

### tag
使用curly bracket包裹，
```dust
{name}
```
### comment
使用两个感叹号之间
```dust
{! Comment syntax !}
```

### key 
键，一般Dust的标签只有两种形式：键和区段，键则对应于curly bracket之间，对应于JSON对象的属性名，对应的属性值一般为简单类型：如字符串
```dust
{name}
```

```
在键名后面可以跟随过滤器，使用竖线分隔，一般用于选择处理“<”，“>”等特殊符号的转义：

{name|s} 禁用自动转码
{name|h} 强制使用HTML转码
{name|j} 强制使用Javascript转码
{name|u} 使用encodeURI编码
{name|uc} 使用encodeURIComponent编码
{name|js} 将JSON对象转换为字符串
{name|jp} 将JSON 字符串转换为JSON对象
还可以输出一些特殊字符：
{~n} 换行
{~r} CR换行
{~lb} 左花括号
{~rb} 右花括号
{~s} 空格
```
### 区段
区段一般用于循环，如：需要对一个数组内的数据进行批量展示成表格

参考上文中的例子
"#" 为开始标签
"/" 为结束标签
键值对应于JSON对象的属性名，对应的键值则对应JSON对象的属性值（如数组或单个对象）。
模板会按照下标对数组中的每个元素调用一次区段包裹着的模板。
```dust
{#names}...{/names}
```
在区段中还可以使用两个特殊的键：
{$idx} 表示当前迭代的序号
{$len} 表示数组的长度

### 上下文 context
上下文这里其实对应于作用域范围

以例子
```json
{
	"name": "root",
	"anotherName": "root2",
	"A":{
		"name":"Albert",
		"B":{
			"name":"Bob"
		}
	}
}
```
使用区段索值
```dust
{#A}{name}{/name}
```
结果： `Albert`

但这种用法：
```dust
{#A}{anotherName}{/name}
```
结果：`root2`

因为dust会从A的上下文中向上查找anotherName和作用域的上下文是类似的；


Ref: [blog](http://blog.sprabbit.com/2013/08/17/introduction-dustjs-2/)