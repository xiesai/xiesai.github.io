---
layout: blog
program: true
background-image: https://gss0.bdstatic.com/94o3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=911e0d8af403738dca470470d272db34/5882b2b7d0a20cf419b28cf574094b36acaf99fd.jpg
title:  "1024程序员节随笔之--angularjs中ng-repeat元素渲染监听"
date:   2018-10-24
background-image: https://gss0.bdstatic.com/94o3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=911e0d8af403738dca470470d272db34/5882b2b7d0a20cf419b28cf574094b36acaf99fd.jpg
category: 前端
tags:
- AngularJs
- 前端
- $animation
---

- 今天10月24日，传说中的程序员节。其实作为猿类，我也不太明白为啥选择1024，有个比较合理的解释是1024为二进制中2的10次方，程序员对这个数据比较敏感，所以就定义为程序员节了。具体为啥定在这一天就不纠结了，既然日子已经确定，那么目前作为节日群体中的一员，总感觉今天不写点东西有点不像话^_^
- 那就写下前段时间在需求开发过程中遇到并解决的问题吧。

## 问题描述

<p>在使用angularjs进行开发的过程中，ng-repeat作为基础指令经常被用在作为对数组元素的遍历，然后html在根据遍历的数据对页面进行渲染。如果数据绑定过程中不涉及对遍历出的dom元素进行操作，那么ng-repeat只需正常使用即可。但是如果在数据绑定的过程中需要操作渲染出的dom元素,那么这时就会遇到一个问题，我们需要监听要操作的dom元素是否已经渲染完成。否则，将取不到对应的元素</p>

<img src="http://img0.ph.126.net/NMzFC0I0UL3zDD5vZQKXfw==/6631461390866443161.jpeg"/>
## 来吧，直接上代码
<p>比如下面有个Html代码片段，代码中使用了ng-repeat，并且将遍历对象中的service属性作为id赋给要渲染出的dom元素</p>
```html
  我们定义Arr=[{service:'xs',id:'01'},{service:'xs1',id:'02'}];
  <div ng-repeat="obj in Arr">
    <div id="obj.service"></div>
  </div>
 ``` 
 <p>现在有个需求是，需要在页面加载完成后将图表绑定到对应id的dom元素中。所以要想解决这个问题，我们就需要能够监听到什么时候ng-repeat中所有的元素都渲染完成了，这个时候js再根据绑定的id取到对应的dom元素绑定图表，否则，如果直接不经过监听就去绑定，那么根据id获取的元素很可能就是null.</p>
## 废话不多说，直接解决问题
<p>解决问题的思路有很多种，我这里面提供的解决方法是写一个自定义指令，去监听ng-repeat中元素的渲染情况。提出用这种思路来解决问题的，网上有很多，但是我大概扫了一眼，基本没有能做到从根本上解决问题的，大部分都是在人云亦云解决很片面的相关问题。</p>
<p>为了突出指令的派系，通常情况下，我会给我的指令命名以‘xs’开头。那么今天的这个指令名称我给定义为'xs-ng-repeat-finish-watch'</p>
```javascript
function xsRepeatFinishWatch($,$animate){
  return{
    replace:true,
    restrict:'AE',
    template:'',
    scope:{
    actmethod:'&'
    }
    link:link
  };
  function link(scope,ele,attrs){
     //判断是否遍历到最后一组
     if(scope.$parent.$last == true){
     //$animate为该指令控件功能实现的核心,该方法返回deserver对象，可以动态给元素添加样式，当添加样式完成后则会执行then里面成功对应的回调函数
     //所以这块我们通过判断最后遍历标签上的样式是否动态添加完成来达到判断整个ng-repeat中dom元素是否渲染完成的目的
      var deserve = $animate.addClass(ele,attrs.initDomClass);
      $.when(deserve).then(function(){
        scope.$parent.$watch('$viewContentLoaded',funcion(){
          scope.actmethod();
        })
      })
   }
  }
}
```
<p>定义完指令，使用方法如下,其中自定义标签中init-dom-class为要添加的样式，actmethod为渲染完成后待执行的方法。</p>
```html
  <div ng-repeat="obj in Arr">
    <div id="obj.service"></div>
    <xs-ng-repeat-finish-watch init-dom-class-"domfinish" actmethod="doPrinter()"></xs-ng-repeat-finish-watch>
  </div>
```
## 总结
<p>问题的整个解决过程描述完了，解决方法还是比较巧妙的。如果你看不懂，那需要首先了解angulajs中最核心的功能点自定义指令。后面应该也会有博文和大家分享相关内容。102节日快乐，大家晚安！</p>