1) Vue-Router如何响应路由参数变化：
由于两个路由都公用一个组件，比起销毁再创建复用显得更加高效。不过也就意味着，Vue的生命周期不再起作用，因此会出现内容不随着路由同步更新。
可以通过简单的watch来监听对路由变化做出响应
```javascript
const user = {
	templete: '...',
	watch: {
		'$route'(to, from) {
			// 对路有变化做出响应
		}
	}
}

// 或者使用beforeRouteUpdate导航守卫
const user = {
	templete: '...',
	beforeRouteUpdate(to, from, next) {
		// react to route changes...
        // don't forget to call next()
	}
}
```

还可以通过'动态的路径参数'达到效果`<router-view :key='$route.fullPath></router-view>`

2）Vue-Router有哪几种导航钩子：
**全局前置守卫（router.beforeEach注册）**
```javascript
const router = new VueRouter({ ... })
router.beforeEach((to, from, next) => {
	// ...
})

```
当一个导航触发时，全局前置守卫按照创建顺序调用。守卫是异步解析执行，此时导航在所有守卫resolve完之前一直处于**等待中**
**全局解析守卫（router.beforeResolve注册）**
在导航被确认之前，**同时在所有组件内守卫和异步路由组件被解析后**，解析守卫就被调用
**全局后置钩子**
你也可以注册全局后置钩子，然而和守卫不同的是，这些钩子不会接受**next**函数也不会改变导航本身
`router.afterEach((to, from) => {
  // ...
})`
路由独享的守卫（beforeEnter）守卫：
```javascript
const router = new VueRouter({
	routes: [
		{
			path: '/foo',
			component: Foo,
			beforeEnter: (to, from, next）=> {
				// ...
			}
		}
	]
})
```
**组件内的守卫（beforeRouteEnter beforeRouteUpdate beforeRouteLeave）**
```javascript
const Foo = {
	template: `...`,
	beforeRouteEnter(to, from, next) {
		// 在渲染该组件的对应路由被 confirm 调用
		// 不！能！获取组件实例上的`this`
		// 因为当守卫执行前，组件还未创建
	},
	beforeRouteUpdate(to, from, next) {
		 // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
   		 // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    	// 可以访问组件实例 `this`
	},
	beforeRouteLeave(to, from, next) {
		// 导航离开该组件时调用
		// 可以访问到`this`
	}
}
```
beforeRouteEnter守卫 **不能** 访问this，因为守卫在导航确认前被调用，因此即将登场的新组件还未被创建。
不过你可以传一个回调给 **next** 来访问组件实例。在导航被确认的时候执行回调，并且把组件实例作为回调方法的参数。
```javascript
beforeRouteEnter(to, from, next) {
	next(vm => {
		// 通过 `vm` 访问组件实例
	}
}
```

3) 完整的Vue-Router导航解析流程：
导航被触发 -> 在失活的组件里面调用离开守卫 -> 调用全局的beforeEach守卫 -> 在复用组件里调用beforeRouterUpdate守卫 -> 在路由配置里调用beforeEnter守卫 -> 解析异步路由组件 -> 在被激活的组件中调用beforeRouterEnter守卫 -> 调用全局的beforeRouterResolve守卫 -> 导航被确认 -> 调用全局的afterEach守卫 -> 触发DOM更新 -> 用创建好的实例调用beforeRouterEnter守卫中的next回调函数。
4）Vue-Router有几种实例方法以及参数传递：
[方法实例与参数传递](https://www.jianshu.com/p/2be6f131cec5?tdsourcetag=s_pcqq_aiomsg "方法实例与参数传递")
5）Vue-Router动态路由匹配以及使用：
我们经常需要把某种模式匹配到的路由映射到同一个组件。我们可以在**vue-router**的路由路径中使用"动态路径参数"来达到这个效果。
```javascript
const User = {
	template: '<div>User</div>
}
const router = new VueRouter({
	routes: [
		// 动态路径参数以冒号开头
		{ path: '/user/:id', component: User }
	]
})

// 一个"路径参数"使用冒泡 ：标记。当匹配到一个路由时, 参数值会被设置到 this.$route.params , 可以在每个组件中使用

const User = {
	template: `div>user {{ $route.parmas.id }} </div>`
}
```
6）Vue-Router如何定义嵌套路由：
需要在顶层出口添加一个`<router-view>`, 要在嵌套的出口中渲染组件, 需要在 VueRouter 添加children配置
```javascript
const User = {
	template: `
  <div class="user">
    <h2> {{$route.params.id}} </h2>
    <router-view></router-view>
  </div>
`
}

// children 配置
const router = new VueRouter({
   router: [
	   {
		   path: '/user/:id, component: User,
		   children: [
		      {
				  // 当 /user/:id/profile 匹配成功，
				  // UserProfile 会被渲染在 User 的 <router-view> 中
				  path: 'profile',
				  component: UserProfile
				},
				{
				  // 当 /user/:id/posts 匹配成功
				  // UserPosts 会被渲染在 User 的 <router-view> 中
				  path: 'posts',
				  component: UserPosts
				}
		   ]
	   }
   ]
})

// 基于上面的配置，当你访问/user/foo时，User出口不会匹配上任何子路由，所以无法渲染出任何东西，如果你想要渲染点什么，可以提供一个 空的 子路由：
const Router = new VueRouter({
	routes: [
		{
			path: '/user/:id', component: User,
			children: [
				 // 当 /user/:id 匹配成功，
				// UserHome 会被渲染在 User 的 <router-view> 中
				{ path: '', component: UserHome },

				// ...其他子路由
			]
		}
	]
})


```
7) `<router-link></router-link>`组件及其属性
8）Vue-router实现路由懒加载
9）Vue-router的两种模式：模式：hash模式和history模式
10）history路由模式与后台的配合