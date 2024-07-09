---
title: 移动端开发
author: 王雨晴
date: '2023-12-22'
---
补：

在HbuilderX中新建页面很简洁，直接在文件目录单击右键选择新建页面，可以选择创建同名目录，也可以勾选自动在pages.json里添加路径，这样就已经可以通过页面的url进行访问了。

在vue2中，template 的二级节点只能有一个节点一般是在一个根 view 下继续写页面组件，要不然会报错，这里使用的就是vue2，而vue3可以有多个二级节点，可以在 manifest 中切换使用 Vue2 还是 Vue3。

ref个人感觉跟id差不多，ref是vue里用来标识组件的

每一个页面的样式就写在当前 page下面
scoped 就只会作用在当前页面组件内，避免造成全局的样式污染
```html
<style lang="scss" scoped>
    page {
		background-color: #ffffff;
	}
</style>
```



## 路由
在uni-app中有两种页面路由跳转方式：

使用navigator组件跳转
```html
 <navigator url="/pages/tabBar/extUI/extUI" hover-class="navigator-hover">
    <button type="default">跳转到新页面</button>
</navigator>
```
调用api跳转
```javascript
uni.navigateTo({
	url:'../logmanage/logdetal'
})
```
Ruoyi中的$tab对象也可用于路由跳转，它定义在plugins/tab.js文件中
```javascript
this.$tab.navigateTo("../logmanage/logdetal");
this.$tab.navigateTo("../logmanage/logdetal").then(() => {
  // 执行结束的逻辑
})
```
>常用的跳转方法：
>* navigateTo保留当前页面，跳转到应用内的某个页面
>* redirectTo关闭当前页面，跳转到应用内的某个页面
>* navigateBack关闭当前页面，返回上一页面或多级页面（系统返回就用的这个，除了这个返回其他都有参数即跳转的url）
>* reLaunch关闭所有页面，打开到应用内的某个页面
>* switchTab跳转到tabBar页面，并关闭其他所有非tabBar页面

>ps：redirectTo换言之在新页面点击系统返回键返回不了之前的界面

路由传参

带参数传递
```javascript
uni.navigateTo({
  url:'/pages/login?id=1&name=ry'
})
```
获取参数
```javascript
export default {
  onLoad(option) {
    console.log(option.id);
    console.log(option.name);
  }
}
```
比如下面这个实战例子，点击当前text跳转页面，将id传给新页面

原页面
```javascript
<view v-for="item in logDataTuo">
    <text @click="openDetal" :data-logid="item.id">{{item.logname}}</text>
</view>

openDetal(e) {
	var logid = e.currentTarget.dataset.logid
	uni.navigateTo({
		url:'../logmanage/logdetal?logid='+logid
	})
},
```
跳转到的新页面，新页面获取参数
```javascript
export default{
    onload(e){
        //这里的e就获取到了上个页面传过来的logid
    }
}
```

navigateBack如何传参给上一个页面？

首先在第一个页面中创建一个变量ziDingYiChongFuGuiZe
```js
data() {
	return {
		ziDingYiChongFuGuiZe:'',
    }
}
```
然后在onshow里，用来接收第二个页面返回携带的参数
```js
onShow() {
	let pages = getCurrentPages();
	if(pages[pages.length-1].$vm.ziDingYiChongFuGuiZe){
		this.ziDingYiChongFuGuiZe=pages[pages.length-1].$vm.ziDingYiChongFuGuiZe
	}
	console.log(this.ziDingYiChongFuGuiZe)
},
```
第二个页面在返回的时候，写如下代码
```js
navBarRightClick(){
	let pages = getCurrentPages();
	let prevPage = pages[pages.length-2]
    let zidingyi='每'+this.repeattime+'天'
	prevPage.$vm.ziDingYiChongFuGuiZe=zidingyi
	uni.navigateBack({
		delta:1
	})
},
```


## 组件
子组件代码
```javascript
<template>
	<view>
		<view type="default" @click.stop="open" style="height: 30px;">{{age}}</view>
		<button type="default" @click.stop="seeme">{{age}}</button>
	</view>
</template>

<script>
	export default {
		name:"childcom",
    //父向子传值
		props:{
			age:{
				type:Number,
				default:0,
				required:true,
				validator:function(value){
					return value >=0
				}
			},
		},
		methods:{
			seeme(){
				this.$emit('childToFather',{name:'wk','sex':'male'})
				this.$emit('update:age',23)  
        //props里的值只能这样更新不能像在data里直接用this.访问
			},
			open(){
				console.log("open")
			},
		},
	}
}
</script>
```
父组件调用代码
```javascript
<childcom :age.sync="agechild" @childToFather="childToFather" @click.native="clickChild"></childcom> 

import childcom from "../logmanage/childComponent.vue"
export default {
	components:{
		childcom
	},
  data(){
    return{
      agechild:34
    }
  },
  methods:{
    clickChild(){
				console.log("click原生事件")
			},
		childToFather(obj){
			console.log(obj)  //子向父传递的参数放在obj里
		},
  }
}
```

如上面的例子，子组件编写完成之后，想在父组件使用子组件，局部注册使用的步骤如下
```javascript
1、引用组件
import 组件名称 from "../../components/组件名.vue";
2、注册组件
export default{
  components:{
    组件名称
  },
}
3、在试图模板中使用组件
<组件名称 组件属性="对应的值"></组件名称>
```
### 父组件向子组件传参（props）
子组件的props可以是数组或对象，用于接收来自父组件的数据，比如上面的例子从父组件接收age这个参数，如果父组件没有传参，默认值为0，父组件在使用子组件时给相应的组件属性赋值就行了，比如直接写 :age="10" ，这样子组件就从父组件接收到了age的值为10（props这里很像自定义的组件属性）
### 子组件向父组件传参（$emit）
$emit类的操作还有$on、$off等，常用于跨页面、跨组件通讯，子组件想要向父组件传递参数，那么子组件需要用到$emit，比如上面这个例子中点击子组件中的一个组件，然后调用相应的点击事件监听方法，向父组件传递参数，这里传递了name和sex两个参数
```javascript
this.$emit('childToFather',{name:'wk','sex':'male'})
```
然后父组件调用时注册事件childToFather，这样传递的参数就存放在了obj里
### 客户端调用组件引用
还有一种客户端调用组件引用（ref）方式（这里我没有太理解，ref是vue里组件的标识，没太看懂这里是怎么操作的）

### 自定义事件
可以使用 @ 事件的.native修饰符来在一个组件的根元素上直接监听一个原生事件，接上面的例子这里以点击事件click为例，@click.native原生事件监听的是所有的子节点
>* 如果子组件没有自定义点击事件，会默认调用父组件定义的方法
>* 如果子组件自己使用@click或者@tap定义了点击事件，那么点击该子组件，父子组件定义的方法都会调用
>* 如果子组件使用@click.stop或者@tap.stop，点击只会调用子组件的方法

>ps:在app、小程序端和h5端表现不一致，h5端获取到的是浏览器原生事件

### 将子组件prop值的变化同步到父组件（.sync）
当一个子组件改变了一个prop的值时，这个变化也会同步到父组件中所绑定。比如上面这个例子，子组件改变了prop值
```javascript
this.$emit('update:age',23) 
```
父组件得加一个.sync，变成:age.sync，要不然值不更新。
.sync它会被扩展为一个自动更新父组件属性的v-on监听器，.sync的作用就是子组件的prop里的值变了将变化同步到父组件。感觉用处在于在子组件内部比如有很多组件，其中某个组件的变化使得值发生了改变。父组件好像无法细致到对子组件具体某个组件的监听，就算有也没有必要，子组件的目的就是为了精简父组件，一些内部操作就应该放在子组件里。

### 两个子组件之间通信
通过$emit、$on、$off来完成

父组件代码
```js
<template>
	<view>
        <page-head title="组件通讯示例"></page-head>
        <view class="uni-padding-wrap">
            <view class="uni-btn-v">
                <reciver></reciver>
                <sender></sender>
            </view>
        </view>
	</view>
</template>
<script>
    import reciver from './reciver.vue'
    import sender from './sender.vue'
	export default {
        components:{
           reciver,
           sender
        },
		data() {
			return {

			}
		},
		methods: {

		}
	}
</script>
```
sender
```js
<template>
    <view class="sender-container">
        <button type="primary" @click="send">点击发送消息</button>
    </view>
</template>

<script>
    export default {
        methods: {
            send() {
                let num = parseInt(Math.random() * 10000)
                uni.$emit('cc', {
                    msg: 'From uni.$emit -> ' + num
                })
            }
        }
    }
</script>

<style>
    .sender-container{
        padding: 20px;
    }
</style>

```
reciver
```js
<template>
    <view>
        <view class="reciver">
            {{msg===''?'等待发送':'收到消息：'}}{{msg}}
        </view>
    </view>
</template>

<script>
    export default {
        data() {
            return {
                msg: ''
            }
        },
        created() {
            uni.$on('cc', this.recive)
        },
        beforeDestroy() {
            uni.$off('cc',this.recive)
        },
        methods: {
            recive(e) {
                this.msg = e.msg
            }
        }
    }
</script>

<style>
    .reciver {
        padding: 40px 0px;
        text-align: center;
        line-height: 40px;
    }
</style>

```
效果
![](http://10.2.0.128:8000/mty8105/picgo-images/-/raw/master/pictures/2023/12/28_13_12_41_20231228131239.png)
		
