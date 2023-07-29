### 一、创建Vue3.0工程

#### 1、使用vue-cli创建

##查看@vue-cli版本
vue --version

##安装或者升级你的@vue/cli
npm install -g @vue/cli

##创建
vue create vue_test

##启动
cd vue_test
npm run serve

#### 2、使用vite创建

##创建工程
npm init vite-app <project-name>

##进入工程目录
cd <project-name>

##安装依赖
npm install

##运行
npm run dev

### 二、常见的Composition API

#### 1、拉开序幕的setup

1.理解：vue3.0中一个新的配置项，值为一个函数。

2.setup是所有Composition API（组合API）“表演的舞台”。

3.组件中所用到的：数据、方法等等，均要配置在setup中。

4.setup函数的两种返回值：

​		1.若返回一个对象，则对象中的属性、方法，在模板中均可以直接使用。（重点关注！）

​		2.若返回一个渲染函数：则可以自定义渲染内容。（了解）

5.注意点：

​		1.尽量不要与Vue2.x配置混用

​				Vue2.x配置（data、method、computed...）中可以访问到setup中的属性、方法。

​				但在setup中不能访问到Vue2.x配置（data、method、computed...）。

​		2.setup不能是一个async函数，因为返回值不再是return的对象，而是promise，模板看不到return对象中的属性。（后期也可以返回一个Promise实例，但需要Suspense和异步组件的配合）

~~~vue
<template>
	<h1>一个人的信息</h1>
	<h2>姓名：{{name}}</h2>
	<h2>年龄：{{age}}</h2>
	<h2>性别：{{sex}}</h2>
	<h2>a的值是：{{a}}</h2>
	<button @click="sayHello">说话(Vue3所配置的——sayHello)</button>
	<br>
	<br>
	<button @click="sayWelcome">说话(Vue2所配置的——sayWelcome)</button>
	<br>
	<br>
	<button @click="test1">测试一下在Vue2的配置中去读取Vue3中的数据、方法</button>
	<br>
	<br>
	<button @click="test2">测试一下在Vue3的setup配置中去读取Vue2中的数据、方法</button>

</template>

<script>
	// import {h} from 'vue'
	export default {
		name: 'App',
		data() {
			return {
				sex:'男',
				a:100
			}
		},
		methods: {
			sayWelcome(){
				alert('欢迎来到尚硅谷学习')
			},
			test1(){
				console.log(this.sex)
				console.log(this.name)
				console.log(this.age)
				console.log(this.sayHello)
			}
		},
		//此处只是测试一下setup，暂时不考虑响应式的问题。
		async setup(){
			//数据
			let name = '张三'
			let age = 18
			let a = 200

			//方法
			function sayHello(){
				alert(`我叫${name}，我${age}岁了，你好啊！`)
			}
			function test2(){
				console.log(name)
				console.log(age)
				console.log(sayHello)
				console.log(this.sex)
				console.log(this.sayWelcome)
			}

			//返回一个对象（常用）
			return {
				name,
				age,
				sayHello,
				test2,
				a
			}
			//返回一个函数（渲染函数）
			// return ()=> h('h1','尚硅谷')
		}
	}
</script>
~~~



#### 2.ref函数

作用：定义一个响应式的数据

语法：const xxx = ref(initValue)

​		创建一个包含响应式数据的引用对象（reference对象，简称ref对象）。

​		JS中操作数据：xxx.value

​		模板中读取数据：不需要.value，直接：<div>{{xxx}}</div>

备注：

​		接收的数据可以是：基本类型、也可以是对象类型。

​		基本类型的数据：响应式依然是靠Object.defineProperty()的get与set完成的。

​		对象类型的数据：内部“求助”了Vue3.0中的一个新函数——reactive函数。

~~~vue
<template>
	<h1>一个人的信息</h1>
	<h2>姓名：{{name}}</h2>
	<h2>年龄：{{age}}</h2>
	<h3>工作种类：{{job.type}}</h3>
	<h3>工作薪水：{{job.salary}}</h3>
	<button @click="changeInfo">修改人的信息</button>
</template>

<script>
	import {ref} from 'vue'
	export default {
		name: 'App',
		setup(){
			//数据
			let name = ref('张三')
			let age = ref(18)
			let job = ref({
				type:'前端工程师',
				salary:'30K'
			})

			//方法
			function changeInfo(){
				// name.value = '李四'
				// age.value = 48
				console.log(job.value)
				// job.value.type = 'UI设计师'
				// job.value.salary = '60K'
				// console.log(name,age)
			}

			//返回一个对象（常用）
			return {
				name,
				age,
				job,
				changeInfo
			}
		}
	}
</script>

~~~



#### 3.reactive函数

作用：定义一个对象类型的响应式数据（基本类型别用它，用ref函数）

语法：const 代理对象 = reactive(被代理对象)接收一个对象（或数组），返回一个代理器对象（Proxy的实例对象，简称proxy对象）

reactive定义的响应式数据是“深层次的”。

内部基于ES6的Proxy实现，通过代理对象操作源对象内部数据都是响应式的。

~~~vue
<template>
	<h1>一个人的信息</h1>
	<h2>姓名：{{person.name}}</h2>
	<h2>年龄：{{person.age}}</h2>
	<h3>工作种类：{{person.job.type}}</h3>
	<h3>工作薪水：{{person.job.salary}}</h3>
	<h3>爱好：{{person.hobby}}</h3>
	<h3>测试的数据c：{{person.job.a.b.c}}</h3>
	<button @click="changeInfo">修改人的信息</button>
</template>

<script>
	import {reactive} from 'vue'
	export default {
		name: 'App',
		setup(){
			//数据
			let person = reactive({
				name:'张三',
				age:18,
				job:{
					type:'前端工程师',
					salary:'30K',
					a:{
						b:{
							c:666
						}
					}
				},
				hobby:['抽烟','喝酒','烫头']
			})

			//方法
			function changeInfo(){
				person.name = '李四'
				person.age = 48
				person.job.type = 'UI设计师'
				person.job.salary = '60K'
				person.job.a.b.c = 999
				person.hobby[0] = '学习'
			}

			//返回一个对象（常用）
			return {
				person,
				changeInfo
			}
		}
	}
</script>


~~~



#### 4、Vue3.0中的响应式原理

**vue2.x的响应式**

​		实现原理：

​				。对象类型：通过Object.defineproperty()对属性的读取、修改进行拦截（数据劫持）。

​				。数组类型：通过重写更新数组的一系列方法来实现拦截。（对数组的变更方法进行了包裹）。

~~~vue
<script type="text/javascript" >
			//源数据
			let person = {
				name:'张三',
				age:18
			}

			//模拟Vue2中实现响应式
			let p = {}
			Object.defineProperty(p,'name',{
				configurable:true,
				get(){ //有人读取name时调用
					return person.name
				},
				set(value){ //有人修改name时调用
					console.log('有人修改了name属性，我发现了，我要去更新界面！')
					person.name = value
				}
			})
			Object.defineProperty(p,'age',{
				get(){ //有人读取age时调用
					return person.age
				},
				set(value){ //有人修改age时调用
					console.log('有人修改了age属性，我发现了，我要去更新界面！')
					person.age = value
				}
			}) 
</script>
~~~

​		存在问题：

​				。新增属性、删除属性，界面不会更新。

​				。直接通过下标修改数组，界面不会自动更新。

**Vue3.0的响应式**

​		实现原理：

​				。通过Proxy（代理）：拦截对象中任意属性的变化，包括：属性值的读写、属性的添加、属性的删除等。

​				。通过Reflect（反射）：对源对象（或称被代理对象）的属性进行操作。

​				。MDN文档中描述的Proxy与Reflect：

​                		Proxy：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy

​						Reflect：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect

~~~vue
<script>
//模拟Vue3中实现响应式
			//#region 
			const p = new Proxy(person,{
				//有人读取p的某个属性时调用（target其实就是person，proname是读的值，比如p.name 读的是name）
				get(target,propName){
					console.log(`有人读取了p身上的${propName}属性`)
					return 9ct.get(target,propName)
				},
				//有人修改p的某个属性、或给p追加某个属性时调用
				set(target,propName,value){
					console.log(`有人修改了p身上的${propName}属性，我要去更新界面了！`)
					Reflect.set(target,propName,value)
				},
				//有人删除p的某个属性时调用
				deleteProperty(target,propName){
					console.log(`有人删除了p身上的${propName}属性，我要去更新界面了！`)
					return Reflect.deleteProperty(target,propName)
				}
			})
			//#endregion

			let obj = {a:1,b:2}
			//通过Object.defineProperty去操作
			//#region 
			/* try {
				Object.defineProperty(obj,'c',{
					get(){
						return 3
					}
				})
				Object.defineProperty(obj,'c',{
					get(){
						return 4
					}
				})
			} catch (error) {
				console.log(error)
			} */
			//#endregion

			//通过Reflect.defineProperty去操作
			//#region 
			/* const x1 = Reflect.defineProperty(obj,'c',{
				get(){
					return 3
				}
			})
			console.log(x1)
			
			const x2 = Reflect.defineProperty(obj,'c',{
				get(){
					return 4
				}
			}) 
			if(x2){
				console.log('某某某操作成功了！')
			}else{
				console.log('某某某操作失败了！')
			} */
			//#endregion

			// console.log('@@@')

</script>
~~~

#### 5.reactive对比ref

从定义数据角度对比：

​		。ref用来定义：基本数据类型。

​		。reactive用来定义：对象（或数组）类型数据。

​		。备注：ref也可以用来定义对象（或数组）类型数据，它内部会自动通过reactive转为代理对象。

从原理角度对比：

​		。ref通过Object.defineproperty()的get与set来实现响应式（数据劫持）。

​		。reactive通过使用Proxy来实现响应式（数据劫持），并通过Reflect操作源对象内部的数据。

从使用角度对比：

​		。ref定义的数据：操作数据需要.value，读取数据时模板中直接读取不需要.value。

​		。reactive定义的数据：操作数据与读取数据：均不需要.value。

#### 6.setup的两个注意点

setup执行的时机

​		。在beforeCreate之前执行一次，this是undefined。

setup的参数

​		。props：值为对象，包含：组件外部传递过来，且组件内部生命接收了的属性。

​		。context：上下文对象

​				attrs：值为对象，包含：组件外部传递过来，但没有在props配置中声明的属性，相当于this.$attrs。

​				slots：收到的插槽内容，相当于this.$slots。

​				emit：分发自定义事件的函数，相当于this.$emit。

Demo.vue
~~~vue
<template>
	<h1>一个人的信息</h1>
	<h2>姓名：{{person.name}}</h2>
	<h2>年龄：{{person.age}}</h2>
	<button @click="test">测试触发一下Demo组件的Hello事件</button>
</template>

<script>
	import {reactive} from 'vue'
	export default {
		name: 'Demo',
		props:['msg','school'],
		emits:['hello'],
		setup(props,context){
			// console.log('---setup---',props)
			// console.log('---setup---',context)
			// console.log('---setup---',context.attrs) //相当与Vue2中的$attrs
			// console.log('---setup---',context.emit) //触发自定义事件的。
			console.log('---setup---',context.slots) //插槽
			//数据
			let person = reactive({
				name:'张三',
				age:18
			})

			//方法
			function test(){
				context.emit('hello',666)
			}

			//返回一个对象（常用）
			return {
				person,
				test
			}
		}
	}
</script>

~~~

app.vue
~~~VUE
<template>
	<Demo @hello="showHelloMsg" msg="你好啊" school="尚硅谷">
		<template v-slot:qwe>
			<span>尚硅谷</span>
		</template>
		<template v-slot:asd>
			<span>尚硅谷</span>
		</template>
	</Demo>
</template>

<script>
	import Demo from './components/Demo'
	export default {
		name: 'App',
		components:{Demo},
		setup(){
			function showHelloMsg(value){
				alert(`你好啊，你触发了hello事件，我收到的参数是:${value}！`)
			}
			return {
				showHelloMsg
			}
		}
	}
</script>

~~~

#### 7.计算属性与监视

**computed函数**

​		定义：要用的属性不存在，需要通过已有属性计算得来。

​		原理：底层借助了 objcet.defineproperty() 万法提供的 getter 和 setter

​		get 函数什么时候执行?

​		a：初次读取时会执行一次

​		b：当依赖的数据发生改变时会被再次调用

​		优势:与 methods 实现相比，内部有缓存机制 (复用)，效率更高，调试方便

​		备注:

​		a：计算属性最终会出现在 vm 上，直接读取使用即可

​		b：如果计算属性要被修改，那必须写 set 函数去响应修改，且 set 中要引起计算时依赖的数据发生改变

​		c：如果计算属性确定不考虑修改，可以使用计算属性的简写形式

vue3.0与vue2.x中的computed配置功能一致

写法

~~~vue
<template>
	<h1>一个人的信息</h1>
	姓：<input type="text" v-model="person.firstName">
	<br>
	名：<input type="text" v-model="person.lastName">
	<br>
	<span>全名：{{person.fullName}}</span>
	<br>
	全名：<input type="text" v-model="person.fullName">
</template>

<script>
	import {reactive,computed} from 'vue'
	export default {
		name: 'Demo',
		setup(){
			//数据
			let person = reactive({
				firstName:'张',
				lastName:'三'
			})
			//计算属性——简写（没有考虑计算属性被修改的情况）
			/* person.fullName = computed(()=>{
				return person.firstName + '-' + person.lastName
			}) */

			//计算属性——完整写法（考虑读和写）
			person.fullName = computed({
				get(){
					return person.firstName + '-' + person.lastName
				},
				set(value){
					const nameArr = value.split('-')
					person.firstName = nameArr[0]
					person.lastName = nameArr[1]
				}
			})

			//返回一个对象（常用）
			return {
				person
			}
		}
	}
</script>

~~~

**watch函数**

​		watch 监视属性

​		a:当被监视的属性变化时，回调函数自动调用，进行相关操作

​		b:监视的属性必须存在，才能进行监视，既可以监视 data ，也可以监视计算属性

​		c:配置项属性 immediate:false ，改为 true，则初始化时调用一次 handler(newValue,oldValue)

​		监视有两种写法.

​		a：创建 Vue 时传入 watch: {}配置

​		b：通过 vm.$watch() 监视

​		深度侦听

​		Vue 中的 watch 默认不监测对象内部值的改变 (一层)

​		2.在 watch 中配置deep:true 可以监测对象内部值的改变 (多层)

​		注意

​		自身可以监测对象内部值的改变，但 vue 提供的 watch 默认不可以Vue

​		使用 watch 时根据监视数据的具体结构，决定是否采用深度监视

vue3.0与vue2.x中的watch配置功能一致

两个小“坑”：

​		。监视reactive定义的响应式数据时：oldValue无法正确获取、强制开启了深度监视（deep配置失效）。

​		。监视reactive定义的响应式数据中某个属性时：deep配置有效。

~~~vue
<template>
	<h2>当前求和为：{{sum}}</h2>
	<button @click="sum++">点我+1</button>
	<hr>
	<h2>当前的信息为：{{msg}}</h2>
	<button @click="msg+='！'">修改信息</button>
	<hr>
	<h2>姓名：{{person.name}}</h2>
	<h2>年龄：{{person.age}}</h2>
	<h2>薪资：{{person.job.j1.salary}}K</h2>
	<button @click="person.name+='~'">修改姓名</button>
	<button @click="person.age++">增长年龄</button>
	<button @click="person.job.j1.salary++">涨薪</button>
</template>

<script>
	import {ref,reactive,watch} from 'vue'
	export default {
		name: 'Demo',
		setup(){
			//数据
			let sum = ref(0)
			let msg = ref('你好啊')
			let person = reactive({
				name:'张三',
				age:18,
				job:{
					j1:{
						salary:20
					}
				}
			})

			//情况一：监视ref所定义的一个响应式数据
			/* watch(sum,(newValue,oldValue)=>{
				console.log('sum变了',newValue,oldValue)
			},{immediate:true}) */

			//情况二：监视ref所定义的多个响应式数据
			/* watch([sum,msg],(newValue,oldValue)=>{
				console.log('sum或msg变了',newValue,oldValue)
			},{immediate:true}) */

			/* 
				情况三：监视reactive所定义的一个响应式数据的全部属性
						1.注意：此处无法正确的获取oldValue
						2.注意：强制开启了深度监视（deep配置无效）
			*/
			/* watch(person,(newValue,oldValue)=>{
				console.log('person变化了',newValue,oldValue)
			},{deep:false}) //此处的deep配置无效 */

			//情况四：监视reactive所定义的一个响应式数据中的某个属性
			/* watch(()=>person.name,(newValue,oldValue)=>{
				console.log('person的name变化了',newValue,oldValue)
			})  */

			//情况五：监视reactive所定义的一个响应式数据中的某些属性
			/* watch([()=>person.name,()=>person.age],(newValue,oldValue)=>{
				console.log('person的name或age变化了',newValue,oldValue)
			})  */

			//特殊情况
			/* watch(()=>person.job,(newValue,oldValue)=>{
				console.log('person的job变化了',newValue,oldValue)
			},{deep:true}) //此处由于监视的是reactive素定义的对象中的某个属性，所以deep配置有效 */


			//返回一个对象（常用）
			return {
				sum,
				msg,
				person
			}
		}
	}
</script>

~~~

watch监视ref数据的说明

~~~vue
setup(){
			//数据
			let sum = ref(0)
			let msg = ref('你好啊')
			let person = ref({
				name:'张三',
				age:18,
				job:{
					j1:{
						salary:20
					}
				}
			})

			console.log(person)
            //sum不能写sum.value，因为写sum.value的值是0，
			watch(sum,(newValue,oldValue)=>{
				console.log('sum的值变化了',newValue,oldValue)
			})
            //person可以写成person.value，这样ref内部会自动通过reactive转为代理对象。或者直接用deep：true。
			watch(person,(newValue,oldValue)=>{
				console.log('person的值变化了',newValue,oldValue)
			},{deep:true})


~~~

**watchEffect**

watch的套路是：既要指明监视的属性，也要指明监视的回调。

watchEffect的套路是：不用指明监视哪个属性，监视的回调中用到哪个属性，那就监视哪个属性。

watchEffect有点像computed：

​		。但computed注重的计算出来的值（回调函数的返回值），所以必须写返回值。

​		。而watchEffect更注重的是过程（回调函数的函数体），所以不用写返回值。

~~~vue
<script>
	import {ref,reactive,watch,watchEffect} from 'vue'
	export default {
		name: 'Demo',
		setup(){
			//数据
			let sum = ref(0)
			let msg = ref('你好啊')
			let person = reactive({
				name:'张三',
				age:18,
				job:{
					j1:{
						salary:20
					}
				}
			})

			//监视
			/* watch(sum,(newValue,oldValue)=>{
				console.log('sum的值变化了',newValue,oldValue)
			},{immediate:true}) */

			watchEffect(()=>{
				const x1 = sum.value
				const x2 = person.job.j1.salary
				console.log('watchEffect所指定的回调执行了')
			})

			//返回一个对象（常用）
			return {
				sum,
				msg,
				person
			}
		}
	}
</script>


~~~

#### 8.vue的生命周期

vue2.x与vue3.0的生命周期图

| ![img](https://img-blog.csdnimg.cn/20200228160557153.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTA2MDgwNg==,size_16,color_FFFFFF,t_70) | <img src="https://img-blog.csdnimg.cn/a591b3781c3e4bc3be19b3383bc745b6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeGlhb0xpYW5nIG8=,size_15,color_FFFFFF,t_70,g_se,x_16" alt="img" style="zoom: 200%;" /> |
| :----------------------------------------------------------- | ------------------------------------------------------------ |

Vue3.0中可以继续使用Vue2.x中的生命周期钩子，但是有两个被更名：

​		。beforeDestroy改名为beforeUnmount

​		。destroyed改名为unmounted

Vue3.0也提供了Composition API形式的生命周期钩子，与Vue2.x中钩子对应关系如下：

​		beforeCreate======>setup()

​		created===========>setup()

​		beforeMount======>onBeforeMount

​		mounted=========>onMounted

​		beforeUpdate=====>onBeforeUpdate

​		updated=========>onUpdated

​		beforeUnmount===>onBeforeUnmount

​		unmounted=======>onUnmounted

~~~vue
<template>
	<h2>当前求和为：{{sum}}</h2>
	<button @click="sum++">点我+1</button>
</template>

<script>
	import {ref,onBeforeMount,onMounted,onBeforeUpdate,onUpdated,onBeforeUnmount,onUnmounted} from 'vue'
	export default {
		name: 'Demo',
		
		setup(){
			console.log('---setup---')
			//数据
			let sum = ref(0)

			//通过组合式API的形式去使用生命周期钩子
			onBeforeMount(()=>{
				console.log('---onBeforeMount---')
			})
			onMounted(()=>{
				console.log('---onMounted---')
			})
			onBeforeUpdate(()=>{
				console.log('---onBeforeUpdate---')
			})
			onUpdated(()=>{
				console.log('---onUpdated---')
			})
			onBeforeUnmount(()=>{
				console.log('---onBeforeUnmount---')
			})
			onUnmounted(()=>{
				console.log('---onUnmounted---')
			})

			//返回一个对象（常用）
			return {sum}
		},
		//通过配置项的形式使用生命周期钩子
		//#region 
		beforeCreate() {
			console.log('---beforeCreate---')
		},
		created() {
			console.log('---created---')
		},
		beforeMount() {
			console.log('---beforeMount---')
		},
		mounted() {
			console.log('---mounted---')
		},
		beforeUpdate(){
			console.log('---beforeUpdate---')
		},
		updated() {
			console.log('---updated---')
		},
		beforeUnmount() {
			console.log('---beforeUnmount---')
		},
		unmounted() {
			console.log('---unmounted---')
		},
		//#endregion
	}
</script>


~~~

注意：通过组合式API的形式去使用生命周期钩子和通过配置项的形式使用生命周期钩子一起使用，优先级是API（不推荐这样使用，非常乱。）要么用组合式API的形式去使用生命周期钩子，要么用配置项的形式使用生命周期钩子。

#### 9.自定义hook函数

什么是hook？

本质是一个函数，把setup函数中使用的Composition API进行了封装。

类似于vue2.x中的mixin。

自定义hook的优势：复用代码，让setup中的逻辑更清楚易懂。

hooks文件的usePoint.js

~~~JavaScript
import {reactive,onMounted,onBeforeUnmount} from 'vue'
export default function (){
	//实现鼠标“打点”相关的数据
	let point = reactive({
		x:0,
		y:0
	})

	//实现鼠标“打点”相关的方法
	function savePoint(event){
		point.x = event.pageX
		point.y = event.pageY
		console.log(event.pageX,event.pageY)
	}

	//实现鼠标“打点”相关的生命周期钩子
	onMounted(()=>{
		window.addEventListener('click',savePoint)
	})

	onBeforeUnmount(()=>{
		window.removeEventListener('click',savePoint)
	})

	return point
}

~~~

demo.vue

~~~vue
<template>
	<h2>当前求和为：{{sum}}</h2>
	<button @click="sum++">点我+1</button>
	<hr>
	<h2>当前点击时鼠标的坐标为：x：{{point.x}}，y：{{point.y}}</h2>
</template>

<script>
	import {ref} from 'vue'
	import usePoint from '../hooks/usePoint'
	export default {
		name: 'Demo',
		setup(){
			//数据
			let sum = ref(0)
			let point = usePoint()
			

			//返回一个对象（常用）
			return {sum,point}
		}
	}
</script>


~~~

#### 10.toRef

作用：创建一个ref对象，其value值指向另一个对象中的某个属性。

语法：const name = toRef(person,'name')

应用：要将响应式对象中的某个属性单独提供给外部使用时。

扩展：toRefs与toRef功能一致，但可以批量创建多个ref对象，语法：toRefs(person)

### 三、其它Composition API

#### 1.shallowReactive与shallowRef

shallowReactive：只处理对象最外层属性的响应式（浅响应式）。（说明只考虑第一次响应式）

shallowRef：只处理基本数据类型的响应式，不进行对象的响应式处理。

什么时候使用？

​		。如果有一个对象数据，结构比较深，但变化时只是外层属性变化===>shallowReactive。

​		。如果有一个对象数据，后续功能不会修改该对象中的属性，而是生新的对象来替换===>shallowRef。

~~~vue
<template>
	<h4>当前的x.y值是：{{x.y}}</h4>
	<button @click="x={y:888}">点我替换x</button>
	<button @click="x.y++">点我x.y++</button>
	<hr>
	<h4>{{person}}</h4>
	<h2>姓名：{{name}}</h2>
	<h2>年龄：{{age}}</h2>
	<h2>薪资：{{job.j1.salary}}K</h2>
	<button @click="name+='~'">修改姓名</button>
	<button @click="age++">增长年龄</button>
	<button @click="job.j1.salary++">涨薪</button>
</template>

<script>
	import {ref,reactive,toRef,toRefs,shallowReactive,shallowRef} from 'vue'
	export default {
		name: 'Demo',
		setup(){
			//数据
			// let person = shallowReactive({ //只考虑第一层数据的响应式
			let person = reactive({
				name:'张三',
				age:18,
				job:{
					j1:{
						salary:20
					}
				}
			})
            //返回的是object，不是响应式的proxy
			let x = shallowRef({
				y:0
			})
			console.log('******',x)

			//返回一个对象（常用）
			return {
				x,
				person,
				...toRefs(person)
			}
		}
	}
</script>


~~~

#### 2.readonly与shallowReadonly

readonly：让一个响应式数据变为只读的（深只读）。

shallowReadonly：让一个响应式数据变为只读的（浅只读）。

应用场景：不希望数据被修改时。

~~~vue
<template>
	<h4>当前求和为：{{sum}}</h4>
	<button @click="sum++">点我++</button>
	<hr>
	<h2>姓名：{{name}}</h2>
	<h2>年龄：{{age}}</h2>
	<h2>薪资：{{job.j1.salary}}K</h2>
	<button @click="name+='~'">修改姓名</button>
	<button @click="age++">增长年龄</button>
	<button @click="job.j1.salary++">涨薪</button>
</template>

<script>
	import {ref,reactive,toRefs,readonly,shallowReadonly} from 'vue'
	export default {
		name: 'Demo',
		setup(){
			//数据
			let sum = ref(0)
			let person = reactive({
				name:'张三',
				age:18,
				job:{
					j1:{
						salary:20
					}
				}
			})

			person = readonly(person)
			// person = shallowReadonly(person)
			// sum = readonly(sum)
			// sum = shallowReadonly(sum)

			//返回一个对象（常用）
			return {
				sum,
				...toRefs(person)
			}
		}
	}
</script>


~~~

#### 3.toRaw与markRaw

toRaw：

​		。作用：将一个由reactive生成的响应式对象转为普通对象。

​		。使用场景：用于读取响应式对象的普通对象，对这个普通对象的所有操作，不会引起页面更新。

markRaw：

​		。作用：标记一个对象，使其永远不会再成为响应式对象。

​		。应用场景：

​				1.有些值不应被设置为响应式的，例如复杂的第三方类库等。

​				2.当渲染具有不可变数据源的大列表时，跳过响应式转换可以提高性能。

~~~vue
<template>
	<h4>当前求和为：{{sum}}</h4>
	<button @click="sum++">点我++</button>
	<hr>
	<h2>姓名：{{name}}</h2>
	<h2>年龄：{{age}}</h2>
	<h2>薪资：{{job.j1.salary}}K</h2>
	<h3 v-show="person.car">座驾信息：{{person.car}}</h3>
	<button @click="name+='~'">修改姓名</button>
	<button @click="age++">增长年龄</button>
	<button @click="job.j1.salary++">涨薪</button>
	<button @click="showRawPerson">输出最原始的person</button>
	<button @click="addCar">给人添加一台车</button>
	<button @click="person.car.name+='!'">换车名</button>
	<button @click="changePrice">换价格</button>
</template>

<script>
	import {ref,reactive,toRefs,toRaw,markRaw} from 'vue'
	export default {
		name: 'Demo',
		setup(){
			//数据
			let sum = ref(0)
			let person = reactive({
				name:'张三',
				age:18,
				job:{
					j1:{
						salary:20
					}
				}
			})

			function showRawPerson(){
				const p = toRaw(person)
                //数据改了，页面不刷新，没有显示出来。
				p.age++  
				console.log(p)
			}

			function addCar(){
				let car = {name:'奔驰',price:40}
				person.car = markRaw(car)
			}

			function changePrice(){
				person.car.price++
				console.log(person.car.price)
			}

			//返回一个对象（常用）
			return {
				sum,
				person,
				...toRefs(person),
				showRawPerson,
				addCar,
				changePrice
			}
		}
	}
</script>


~~~

#### 4.customRef

作用：创建一个自定义的ref，并对其依赖项跟踪和更新触发进行显式控制。

实现防抖效果：

~~~vue
<template>
	<input type="text" v-model="keyWord">
	<h3>{{keyWord}}</h3>
</template>

<script>
	import {ref,customRef} from 'vue'
	export default {
		name: 'App',
		setup() {
			//自定义一个ref——名为：myRef
			function myRef(value,delay){
				let timer
				return customRef((track,trigger)=>{
					return {
						get(){
							console.log(`有人从myRef这个容器中读取数据了，我把${value}给他了`)
							track() //通知Vue追踪value的变化（提前和get商量一下，让他认为这个value是有用的）
							return value
						},
						set(newValue){
							console.log(`有人把myRef这个容器中数据改为了：${newValue}`)
							clearTimeout(timer)
							timer = setTimeout(()=>{
								value = newValue
								trigger() //通知Vue去重新解析模板
							},delay)
						},
					}
				})
			}

			// let keyWord = ref('hello') //使用Vue提供的ref
			let keyWord = myRef('hello',500) //使用程序员自定义的ref
			
			return {keyWord}
		}
	}
</script>


~~~

#### 5.provide与inject

![image-20230728143950266](C:\Users\24499\AppData\Roaming\Typora\typora-user-images\image-20230728143950266.png)

作用：实现祖与后代组件间的通信。

套路：父组件有一个provide选项来提供数据，后代组件有一个inject选项来开始使用这些数据。

具体写法：

​		1.祖组件中：

~~~vue
<template>
	<div class="app">
		<h3>我是App组件（祖），{{name}}--{{price}}</h3>
		<Child/>
	</div>
</template>

<script>
	import { reactive,toRefs,provide } from 'vue'
	import Child from './components/Child.vue'
	export default {
		name:'App',
		components:{Child},
		setup(){
			let car = reactive({name:'奔驰',price:'40W'})
			provide('car',car) //给自己的后代组件传递数据
			return {...toRefs(car)}
		}
	}
</script>

<style>
	.app{
		background-color: gray;
		padding: 10px;
	}
</style>

~~~

​		2.后代组件中：

~~~vue
<template>
	<div class="son">
		<h3>我是Son组件（孙），{{car.name}}--{{car.price}}</h3>
	</div>
</template>

<script>
	import {inject} from 'vue'
	export default {
		name:'Son',
		setup(){
			let car = inject('car')
			return {car}
		}
	}
</script>

<style>
	.son{
		background-color: orange;
		padding: 10px;
	}
</style>
~~~

#### 6.响应式数据的判断

isRef:检查一个值是否为一个ref对象。

isReactive：检查一个对象是否是由reactive创建的响应式代理。

isReadonly：检查一个对象是否是由readonly创建的只读代理。

isProxy：检查一个对象是否是由reactive或者readonly方法创建的代理。

~~~vue
<template>
	<h3>我是App组件</h3>
</template>

<script>
	import {ref, reactive,toRefs,readonly,isRef,isReactive,isReadonly,isProxy } from 'vue'
	export default {
		name:'App',
		setup(){
			let car = reactive({name:'奔驰',price:'40W'})
			let sum = ref(0)
			let car2 = readonly(car)

			console.log(isRef(sum))
			console.log(isReactive(car))
			console.log(isReadonly(car2))
			console.log(isProxy(car))
			console.log(isProxy(sum))

			
			return {...toRefs(car)}
		}
	}
</script>

<style>
	.app{
		background-color: gray;
		padding: 10px;
	}
</style>
~~~

### 四、Composition API的优势

#### 1.Options API存在的问题

使用传统Options API中，新增或者修改一个需求，就需要分别在data，methods，computed里修改。

![image-20230728145258067](C:\Users\24499\AppData\Roaming\Typora\typora-user-images\image-20230728145258067.png)

#### 2.Composition API 的优势

我们可以更加优雅的组织我们的代码，函数。让相关共能的代码更加有序的组织在一起。

![image-20230728145516240](C:\Users\24499\AppData\Roaming\Typora\typora-user-images\image-20230728145516240.png)

### 5、新的组件

#### 1.Fragment

在vue2中：组件必须有一个根标签

在vue3中：组件可以没有根标签，内部会将多个标签包含在一个Fragment虚拟元素中

好处：减少标签层级，减小内存占用。

#### 2.Teleport

什么是Teleport？

Teleport是一种能够将我们的组件html结构移动到指定位置的技术。

~~~vue
<template>
	<div>
		<button @click="isShow = true">点我弹个窗</button>
        //to="移动位置"
		<teleport to="body">
			<div v-if="isShow" class="mask">
				<div class="dialog">
					<h3>我是一个弹窗</h3>
					<h4>一些内容</h4>
					<h4>一些内容</h4>
					<h4>一些内容</h4>
					<button @click="isShow = false">关闭弹窗</button>
				</div>
			</div>
		</teleport>
	</div>
</template>

<script>
	import {ref} from 'vue'
	export default {
		name:'Dialog',
		setup(){
			let isShow = ref(false)
			return {isShow}
		}
	}
</script>

<style>
	.mask{
		position: absolute;
		top: 0;bottom: 0;left: 0;right: 0;
		background-color: rgba(0, 0, 0, 0.5);
	}
	.dialog{
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%,-50%);
		text-align: center;
		width: 300px;
		height: 300px;
		background-color: green;
	}
</style>
~~~

#### 3.Suspense

等待异步组件时渲染一些额外内容，让应用有更好的用户体验。

使用步骤：

​		。异步引入组件

~~~vue
import {defineAsyncComponent} from 'vue' 
const Child = defineAsyncComponent(()=>import('./components/Child')) //异步引入
~~~

​		。使用Suspense包裹组件，并配置好default与fallback

~~~vue
<Suspense>
		<template v-slot:default>
			<Child/>
		</template>
		<template v-slot:fallback>
			<h3>稍等，加载中...</h3>
		</template>
</Suspense>
~~~

### 六、其他

#### 1.全局API的转移

**Vue2.x有许多全局API和配置**

​		。例如：注册全局指令等。

~~~vue
(1).局部指令：
	directives:{指令名:配置对象}   或   		directives{指令名:回调函数}
(2).全局指令：
    Vue.directive(指令名,配置对象) 或   Vue.directive(指令名,回调函数)

~~~

**Vue3.0中对这些API作出了调整：**

​		。将全局的API，即：Vue.xxx调整到应用实例（app）上。

| 2.x全局API（vue）        | 3.实例API（app）            |
| ------------------------ | --------------------------- |
| Vue.config.xxx           | app.config.xxx              |
| Vue.config.productionTip | 移除                        |
| Vue.component            | app.component               |
| Vue.directive            | app.directive               |
| Vue.mixin                | app.mixin                   |
| Vue.use                  | app.use                     |
| Vue.prototype            | app.config.globalProperties |

#### 2.其他改变

data选项应始终被声明为一个函数。

**过度类名的更改：**

​		。vue2.x写法

~~~vue
.v-enter,
.v-leave-to{
	opacity:0;
}
.v-leave,
.v-enter-to{
	opacity:1;
}
~~~

​		。vue3.x写法

~~~vue
.v-enter-from,
.v-leave-to{
	opacity:0;
}
.v-leave-from,
.v-enter-to{
	opacity:1;
}
~~~

**移除**keyCode作为v-on的修饰符，同时也不再支持config.keyCodes

**移除**v-on.native修饰符

​		。父组件中绑定事件

~~~vue
<my-component
              v-on:close="handleComponentEvent"
              v-on:click="handleNativeClickEvent"
/>
~~~

​		。子组件中声明自定义事件

~~~vue
<script>
	export default{
        emits:['close']
    }
</script>
~~~

**移除过滤器（filter）**

​		过滤器虽然这看起来很方便，但它需要一个自定义语法，打破大括号内表达式是“只是JavaScript”的假设，这不仅有学习成本，而且有实现成本！建议用方法调用或计算属性去替换过滤器。



......
