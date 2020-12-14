---
title: this.$root任意控制父子兄弟组件
date: 2020-06-13 21:06:51
tags:
- 通信传递
- vue
---
一个朋友在开发组件时候，需求是集合各种状态信息的安全检测，分若干个版面显示状态，   
因为各个版本各自牵涉接口和业务逻辑比较复杂，所以她是用父子组件嵌套的框架：父组件嵌套多个子组件来实现业务分离，形式如下：     
```html
<parent><child1></child1>
	<child2></child2>
	<child3></child3>
	<child4></child4>
</parent>
```

虽然分离业务的出发点是好的，但是在组件之间数据的控制就比较麻烦，她只是简单的用了emit和on的父子组件，但是子组件的的数据初始化时机难以把握。     
查阅了相关资料后，发现不必用vuex，用根组件this.$root就可以轻松解决。引用如下：   
```javascript
//注册方法
Vue.component('parent',{
	props:["parent-prop"],
	template:` ...`
	})
	
Vue.component('child',{
	template:` ...`
	})	
```

1. 不唯一，可以多个子组件设定相同的 ref 值）   
2. 父组件通过 refs 访问子组件/子元素   

```html
<parent>
	<child ref="the-child">
	</child>
</parent>
```   

a .可以在父组件 parent 里面使用 this.refs.the-child 来访问子组件里面的数据/ 方法   
b. 可以访问父组件里面子元素（不是组件，是父组件中的dom 元素）的数据/ 方法       
甚至可以通过其父级组件定义方法：    

```javascript
methods: {
  // 用来从父级组件聚焦输入框
  focus: ()=> {
    this.$refs.input.focus();
  }
}
```   
这样就允许父级组件通过下面的代码聚焦 <base-input> 里的输入框：   

this.$refs.usernameInput.focus()
另一个例子：（访问父组件里面子元素的数据）
在子组件：这个例子是给 "welcome"  这个组件设置了一个this.$refs.input.mes

在根实例：用 this.$refs.input.mes 可以访问到子组件的 data 数据。这里访问到了子组件的 "mes" 数据    
```html
<div id="div">
   <form>
   <input is="welcome" ref="input" >
   </input>
   </form>
</div>
```    

```javascript
Vue.component("welcome",{
  template:"<form><input>点击通过emit( )触发事件</input></form>",
  data:function(){
    return {mes:"good good study"}}
})
 
new Vue({
  el:"#div",
  mounted:
    function(){
      alert(this.$refs.input.mes) 
   }
  })  
```   

c. 不唯一，可以多个子组件设定相同的 ref 值）    
当 ref 和 v-for 一起使用的时候，你得到的引用将会是一个包含了对应数据源的这些子组件的数组
