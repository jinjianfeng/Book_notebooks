### 方法一：

````js
var getRandomColor = function(){

  return  '#' +

    (function(color){

    return (color +=  '0123456789abcdef'[Math.floor(Math.random()*16)])

      && (color.length == 6) ?  color : arguments.callee(color);

  })('');

}
````

随机生成6个字符然后再串到一起，闭包调用自身与三元运算符让程序变得内敛。
 
 
### 方法二： 

````js
var getRandomColor = function(){

  return (function(m,s,c){

    return (c ? arguments.callee(m,s,c-1) : '#') +

      s[m.floor(m.random() * 16)]

  })(Math,'0123456789abcdef',5)

}
````

把Math对象，用于生成hex颜色值的字符串提取出来，并利用第三个参数来判断是否还继续调用自身。
 
 
### 方法三：

````js
Array.prototype.map = function(fn, thisObj) {

  var scope = thisObj || window;

  var a = [];

  for ( var i=0, j=this.length; i < j; ++i ) {

    a.push(fn.call(scope, this[i], i, this));

  }

  return a;

};

var getRandomColor = function(){

  return '#'+'0123456789abcdef'.split('').map(function(v,i,a){

    return i>5 ? null : a[Math.floor(Math.random()*16)] }).join('');

}
````

这个要求我们对数组做些扩展，map将返回一个数组，然后我们再用join把它的元素串成字符。
 
 
### 方法四：

````js
var getRandomColor = function(){

  return '#'+Math.floor(Math.random()*16777215).toString(16);

}
````

这个实现非常逆天，虽然有点小bug。我们知道hex颜色值是从#000000到#ffffff，后面那六位数是16进制数，相当于"0x000000"到"0xffffff"。这实现的思路是将hex的最大值ffffff先转换为10进制，进行random后再转换回16进制。我们看一下，如何得到16777215 这个数值的。
 
 
### 方法五：

````js
var getRandomColor = function(){

  return '#'+(Math.random()*0xffffff<<0).toString(16);

}
````

基本实现4的改进，利用左移运算符把0xffffff转化为整型。这样就不用记16777215了。由于左移运算符的优先级比不上乘号，因此随机后再左移，连Math.floor也不用了。
 
 
### 方法六：

````js
var getRandomColor = function(){

  return '#'+(function(h){

    return new Array(7-h.length).join("0")+h

  })((Math.random()*0x1000000<<0).toString(16))

}
````

修正上面版本的bug（无法生成纯白色与hex位数不足问题）。0x1000000相当0xffffff+1，确保会抽选到0xffffff。在闭包里我们处理hex值不足6位的问题，直接在未位补零。
 
 
### 方法七：

````js
var getRandomColor = function(){

  return '#'+('00000'+(Math.random()*0x1000000<<0).toString(16)).slice(-6);

}
````

这次在前面补零，连递归检测也省了。
上面版本生成颜色的范围算是大而全，但随之而来的问题是颜色不好看，于是实现8搞出来了。它生成的颜色相当鲜艳。
 
 
### 方法八：

````js

var getRandomColor = function(){
    return "hsb(" + Math.random()  + ", 1, 1)";
 }
 
````

