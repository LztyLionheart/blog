---
title: Zepto
date: 2016-12-04 06:34:22
tags: 
- Zepto
categories: 
- Javascript
---

```
var Zepto =(function(){
    var $
    var zepto = {}
    /*省略一万行*/
    zepto.Z = function(dom, selector) {
           //将dom 隐式原型强制改为$.fn
           // 这玩意就是个构造函数 ( zepto.Z.prototype = $.fn  所以 dom.__proto__ = $fn 等于
zepto.Z.prototype ) 
          dom = dom || []
          dom.__proto__ = $.fn
          dom.selector = selector || ''
          return dom  
    }
   $ = function(selector, context){
          return zepto.init(selector,context);
   }

   /*你使用的api 全部都在fn里*/ 
   $.fn = {
         push: emptyArray.push,
         indexOf: emptyArray.indexOf,
         ..........
   }
 zepto.init = function(selector, context) {
        var dom
        /*此处省略xxx行*/
        //这儿最终传入dom就是数组  最后创建对象是通过zepto.Z这个构造函数
        return zepto.Z(dom, selector)
  }
 // $被匿名函数返回 并复制给全局window.Zepto 变量
 //  全局的zepto变量暴露给了window  并起到别名$
   return $;
})()

window.Zepto = Zepto;
window.$ === undefind && (window.$ = Zepto)
```
