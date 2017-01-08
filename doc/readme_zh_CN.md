# EasyTemplateJS 模板引擎使用手册


**EasyTemplateJS（EasyTemplate JavaScript）是一款小巧，纯粹，高性能的 JavaScript 模板引擎。**

JavaScript 模板引擎作为数据与界面分离工作中最重要一环。使用 JavaScript 模板函数能够避免在 JavaScript 中拼接 `HTML` 字符串带来的不便和低维护性的缺点，利用反向思路，在 `HTML` 中嵌入 JavaScript 脚本，就像利用 `JSP` 和 `ASP` 技术编程一样。EasyTemplate 能够提供超高性能的渲染引擎，在 JavaScript 中使用模板技术来简化操作，并增强程序设计的灵活性。

The latest version: `2.0.0-RELEASE`


## 特点

- 小巧，纯粹

- 预先静态编译，高性能

- 灵活自定义

- 支持转义输出表达式

- 支持 out 输出


## Performance test comparison/性能测试对比

从渲染性能上来说， **EasyTemplateJS** 和 artTemplate 都是使用预先静态编译原理，可以说已经接近的性能极限，是当前性能最高的模板引擎。一些实现较差的引擎不仅可能影响客户体验，还会会引起浏览器崩溃或异常终止，百度的引擎则性能较差，对浏览器渲染执行影响巨大。

![Performance test comparison](imgs/performance.png)


## 使用

### 1. 引入JS文件

```HTML
<script type="text/javascript" src="easy.templatejs.min-2.0.0.js"></script>
```

### 2. TemplateJS 模板表达式

TemplateJS 支持三类模板表达式，表达式不允许嵌套或交叉:

1. **脚本表达式**

  `%{ JavaScript Script }%`： 执行任意的 JavaScript 代码（作用JSP的`<% %>`小脚本相同），JS脚本中的 `<`、`>` 等特殊符号，也可使用对应字符实体代替。
  
2. **输出表达式**

  `{expression}`： 插入要输出的变量（作用与 JSP 的 `<%=expression%>` 相同）。

3. **转义输出表达式**
 
  `{-expression}`： 用法与`{expression}`相同，输出数据时会自动转义特殊字符为字符实体。
  

-  **为什么选择 %{}%， {} 作为闭合标签？**

 EasyTemplateJS 没有选择常用的  `<%%>` 或 `${}` 作为模板引擎的默认闭合标签，因为在 `JSP`，`ASP` 等动态页面中，`<%%>`，`${}` 都本身是动态特殊标记，当在 `JSP` 页面定义模板标签时，会对 `JSP` 解析造成影响，导致编译错误。所以 EasyTemplate 选择了尽量不会与其他语言冲突的 `%{}%` `{}`。

 尽管如此，但是如果您更喜欢使用 `<%%>` 或 `${}`，本身 EasyTemplate 模板标签是对外允许自定义的，您可以修改为 `<%%>`，以兼容你旧的模板代码。 (`参考 5. 模板表达式闭合标签自定义`)


### 3. 使用实例

EasyTemplateJS 向外暴露了一个名为**`Et`**的对象，用来完成模板操作。

```JS
// 1. 直接执行模板
var result = Et.template(tmplText, data);

// 2. 模板编译
var compiled = Et.template(tmplText);
// 编译后执行
var res2 = compiled(data);
var res3 = compiled(data2);
var res4 = compiled(data3);
```

- 模板基本示例

	```JS
	// Basic demo
	var compiled = Et.template("hello: { name }, {-name}");
	var res = compiled({
		name: 'MoMo'
	});
	console.info(res); // hello: MoMo, MoMo
	
	// Special label, test escape
	var res2 = compiled({
		name: '<MoMo>'
	}); 
	console.info(res2); // hello: <MoMo>, &lt;MoMo&gt;
	
	
	// Test escape (Special label)
	var compiled2 = Et.template("<u>{- value }</u>");
	var res3 = compiled2({
		value: '<script>'
	});
	console.info(res3); // <u>&lt;script&gt;</u>


	var res4 = Et.template("%{ out('Hello: '+name); }%", {
		name: "JACK"
	});
	console.info(res4); //Hello: JACK
	```


- HTML 模板示例

	```HTML
	<!-- 普通模板 -->
	<script id="tmpl" type="text/tmpl">
		%{ for(var i in people){ }%
			<li>{i} = { people[i] }</li>
		%{ } }%
	</script>

	<!-- 使用HTML定义模板内容时，如果有<、>等特殊内容，可以使用对应字符实体代替 -->
	<script id="tmpl2" type="text/tmpl">
		%{ for(var i=0; i &lt; people.length; i++){ }%
			<li>{i} = { people[i] }</li>
		%{ } }%
	</script>

	<!-- 使用 out 输出 -->
	<script id="tmpl3" type="text/tmpl">
		%{ 
			for(var i=0; i < people.length; i++){ 
				out( "<li>"+i+ " = "+people[i]+ "</li> "); 
			} 
		}% 
	</script>
	
	
	<script type="text/javascript">
		//借助了jQuery
	    $(function(){
			<!-- 使用模板  -->
			// temp demo
			var tmpl = $("#tmpl").html();
			var res5 = Et.template(tmpl, {
				people: ['MoMo', 'Joy', 'Ray']
			});
			console.info(res5);
		
			// temp2 demo
			var tmpl2 = $("#tmpl2").html(); 
			var res6 = Et.template(tmpl2, {
				people: ['MoMo', 'Joy', 'Ray']
			});
			console.info(res6);
		
			// temp3 demo
			var tmpl3 = $("#tmpl3").html(); 
			var res7 = Et.template(tmpl3, {
				people: ['MoMo', 'Joy', 'Ray']
			});
			console.info(res7);
		});
	</script>
	```

	输出结果：
	
	```HTML
	<li>0 = MoMo</li>      
	<li>1 = Joy</li>    
	<li>2 = Ray</li>          
	```

### 4. 使用 out 输出信息

您也可以在JavaScript代码中使用 `out` 函数输出信息，这样不用断开您的代码块，有时候这会比使用  `{name}` 更方便清晰。

```HTML
<!-- 使用 out 输出 -->
<script id="tmpl3" type="text/tmpl">
	%{ 
		for(var i=0; i < people.length; i++){ 
			// out function
			out( "<li>"+i+ " = "+people[i]+ "</li> "); 
		} 
	}% 
</script>
```

```JS
var res4 = Et.template("%{ out('Hello: '+name); }%", {
		name: "JACK"
	});
console.info(res4); //Hello: JACK
```


### 5. 模板表达式闭合标签自定义

由于某些模板定义和执行块在某些动态页面（`JSP`, `ASP`）中具有特殊涵义，所以在某些页面中使用模板符号会引起错误。EasyTemplate允许改变模板设置, 使用别的符号来嵌入代码。

> **为什么选择 %{}%， {} 作为闭合标签？**
>
> EasyTemplateJS 没有选择常用的  `<%%>` 或 `${}` 作为模板引擎的默认闭合标签，因为在 `JSP`，`ASP` 等动态页面中，`<%%>`，`${}` 都本身是动态特殊标记，当在 `JSP` 页面定义模板标签时，会对 `JSP` 解析造成影响，导致编译错误。所以 EasyTemplate 选择了尽量不会与其他语言冲突的 `%{}%` `{}`。
> 
> 尽管如此，但是如果您更喜欢使用 `<%%>` 或 `${}`，本身 EasyTemplate 模板标签是对外允许自定义的，您可以修改为 `<%%>`，以兼容你旧的模板代码。 

**注意：** 如果您绝对修改模板表达式的闭合标签，您需要小心检查您的定义逻辑是否合理。

`Et.tmplSettings` 默认配置：

```JS
Et.tmplSettings={
	// 脚本表达式开始结束标记%{ JS script }%
	scriptBegin:"%{",
	scriptEnd:"}%",
	// 输出表达式开始结束标记 {name}
	outBegin:"{",
	outEnd:"}",
	// 转义输出表达式开始结束标记 {-name}
	escapeOutBegin:"{-",
	escapeOutEnd:"}"
}
```

重新定义示例：

```JS
// 修改脚本表达式开始结束标记
var userSettings=
{
	// 脚本表达式开始结束标记 <% JS script %>
	scriptBegin:"<%",
	scriptEnd:"%>",
	// 输出表达式开始结束标记 <%=name %>
	outBegin:"<%=",
	outEnd:"%>",
	// 转义输出表达式开始结束标记 <%-name %>
	escapeOutBegin:"<%-",
	escapeOutEnd:"%>"
}
```

```JS
// 全局修改
Et.tmplSettings=userSettings;

// 测试
console.info(
	Et.template(
		"hello: <%= name %>, <%- name %>",  // templateText
		{name: '<MoMo>'} // data
	)
);			
```

```JS
// 局部修改测试
console.info(
	Et.template(
		"hello: <%= name %>, <%- name %>",  // templateText
		{name: '<MoMo>'}, 					// data
		userSettings 						//settings
	)
);	
```


### 6. API

`Et` 暴露了有限的几个 API:

- 核心函数

	```JS
	/**
	 * 模板引擎编译核心函数
	 * @param {String} tmplText 模板代码
	 * @param {Object} data 模板数据；可选
	 * @param {Et.tmplSettings} settings 引擎动态控制参数；可选
	 * @return {String|function} 如果 data 不为空，则返回 String 渲染结果；否则，返回一个编译后的 function 渲染函数
	 */
	Et.template(tmplText, data, settings);
	```

- 模板闭合标签自定义(`5. 模板表达式闭合标签自定义`)

	```JS
	Et.tmplSettings ={...}
	```
  
- 将字符串中的特殊字符转义为字符实体，并返回

	```JS
	/**
	 * 将字符串中的特殊字符转义为字符实体，并返回
	 * @param {String} text 字符串
	 * @return {String} 转意后的字符串
	 */
	Et.escape(text);
	```  

- escape的反向操作函数，将字符串中的字符实体转移为字符，并返回

	```JS
	/**
	 * escape的反向操作函数，将字符串中的字符实体转移为字符，并返回
	 * @param {text} string 字符串
	 * @return {String} 转意后的字符串
	 */
	Et.unescape(text);
	``` 
  
- 遍历集合或对象
	
	```JS
	/**
	 * 遍历集合或对象。
	 * @param {Object} obj 要遍历的集合或对象
	 * @param {function} iterator 迭代函数，包括值、索引、集合对象三个参数
	 * @param {Object} context 将iterator绑定到context上执行，可用来向iteraotr传递一些其他元素
	 */
	 Et.each(list, iterator, [context]);
	```

- 将 `Et` 引用的对象还原为原始对象，并返回 `Et` 对象。

	```JS
	/**
	 * 排除冲突
	 * @return {Et} Et 对象
	 */
	Et.noConflict();
	```
   


## 结束

### [官方主页](http://www.easyproject.cn/easytemplate/zh-cn/index.jsp '官方主页')

[留言评论](http://www.easyproject.cn/easytemplate/zh-cn/index.jsp#donation '留言评论')

如果您有更好意见，建议或想法，请联系我。

### [The official home page](http://www.easyproject.cn/easytemplate/en/index.jsp 'The official home page')

[Comments](http://www.easyproject.cn/easytemplate/en/index.jsp#donation 'Comments')

If you have more comments, suggestions or ideas, please contact me.



Email：<inthinkcolor@gmail.com>

[http://www.easyproject.cn](http://www.easyproject.cn "EasyProject Home")



**支付宝钱包扫一扫捐助：**

我们相信，每个人的点滴贡献，都将是推动产生更多、更好免费开源产品的一大步。

**感谢慷慨捐助，以支持服务器运行和鼓励更多社区成员。**

<img alt="支付宝钱包扫一扫捐助" src="http://www.easyproject.cn/images/s.png"  title="支付宝钱包扫一扫捐助"  height="256" width="256"></img>



We believe that the contribution of each bit by bit, will be driven to produce more and better free and open source products a big step.

**Thank you donation to support the server running and encourage more community members.**

[![PayPal](http://www.easyproject.cn/images/paypaldonation5.jpg)](https://www.paypal.me/easyproject/10 "Make payments with PayPal - it's fast, free and secure!")
