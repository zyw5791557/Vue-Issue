

### 1、vue-router 路由跳转传值问题
```
<router-link :to="{ name: 'Names', params: { 'userId': '123' }}">User</router-link>
```
使用params传值，必须使用name而不能用path，不然值传不过去，如：
```
<router-link :to="{ path: 'Names', params: { 'userId': '123' }}">User</router-link>
```
而query则没问题
```
<router-link :to="{ path: '/Names', query: { plan: 'private' }}">Register</router-link>
```

### 2、watch 数组或者对象
1、普通的watch
```
data() {
    return {
        frontPoints: 0    
    }
},
watch: {
    frontPoints(newValue, oldValue) {
        console.log(newValue)
    }
}
```

2、数组的watch
```
data() {
    return {
        winChips: new Array(11).fill(0)   
    }
},
watch: {
　　winChips: {
　　　　handler(newValue, oldValue) {
　　　　　　for (let i = 0; i < newValue.length; i++) {
　　　　　　　　if (oldValue[i] != newValue[i]) {
　　　　　　　　　　console.log(newValue)
　　　　　　　　}
　　　　　　}
　　　　},
　　　　deep: true
　　}
}
```

3、对象的watch
```
data() {
　　return {
　　　　bet: {
　　　　　　pokerState: 53,
　　　　　　pokerHistory: 'local'
　　　　}   
    }
},
watch: {
　　bet: {
　　　　handler(newValue, oldValue) {
　　　　　　console.log(newValue)
　　　　},
　　　　deep: true
　　}
}
```

tips: 只要bet中的属性发生变化（可被监测到的），便会执行handler函数；
如果想监测具体的属性变化，如pokerHistory变化时，才执行handler函数，则可以利用计算属性computed做中间层。
事例如下：

 4、对象具体属性的watch［活用computed］
 ```
 data() {
　　return {
　　　　bet: {
　　　　　　pokerState: 53,
　　　　　　pokerHistory: 'local'
　　　　}   
    }
},
computed: {
　　pokerHistory() {
　　　　return this.bet.pokerHistory
　　}
},
watch: {
　　pokerHistory(newValue, oldValue) {
　　　　console.log(newValue)
　　}
}
 ```

### 3、Vue2 几种常见开局方式

Vue2 加了reader选项后, 再加上几种构建方式, 开局方式真是各种五花八门, 这里列几种常见的, 说说注意点

我们先建立一个 app.vue 来当入口组件, 即所有页面都会以这个组件为模板 (下面代码中无特别说明, App 即指下面这个组件)
```
<template>
  <div id="app">
    <img src="./assets/logo.png">
    <hello></hello>
    <!-- 这里还可以写点其他组件, 或者路由也可以 <router-view></router-view> -->
  </div>
</template>
<script>
import Hello from './components/Hello'
export default {
  name: 'app',
  components: {
    Hello
  }
}
</script>
<style>
</style>
```
 方式 1
 模板文件:
 ```
 <div id="app"></div>
 ```
注意: vue2 已经不支持直接绑定在 body 和 html 元素上, 所以我们需要在 body 里写个挂载元素

main.js 入口文件:
```
import App from './app.vue'
new Vue({
  el: '#app',
  render: creatElment => creatElment(App)
})
```
这里的采用 es6 的写法, 转出 es5 就是
```
render: funciton(creatElment) {
    return creatElment(App)
}
```
creatElment 的第一个参数可以是String(HTML 标签名称) | Object(组件对象) | Function(函数), 这里传的就是个组件对象
注意: 这种情况下, App 组件并不是根组件

方式 2

模板文件:
```
<div id="app"></div>
```
main.js 入口文件:
```
import App from './app.vue'
new Vue({
  el: '#app',
  render: creatElment => creatElment('App'),
  components: {
    App
  }
})
```
这个写法是不是和第一种很像? 只不过这里传的就是个App标签, 通过render渲染一个<App></App>元素, 然后把 App 当组件来用
注意: 这种情况下, App 组件并不是根组件

方式 3

模板文件:
```
<div id="app"></div>
```
main.js 入口文件:
```
import App from './app.vue'
new Vue({
  el: '#app',
  ...App
})
```
这种方法和方式1 基本一样, 区别就在于render: creatElment => creatElment(App)和...App
render: creatElment => creatElment(App)是把 App 当成一个组件对象, 给render解析, 而...App是将 App 这个组件对象和{el: '#app'}这个对象直接合并, 变成Vue的参数
注意: 这种情况下, this.$root 是 App 组件

注意: 这种写法, 需要在.babelrc里添加stage-3以上的presets, 如:
```
{
    "presets": ["es2015", "stage-2"]
}
```
方式 4

模板文件:
```
<div id="app">
    <App></App>
</div>
```
main.js 入口文件:
```
import App from './app.vue'
new Vue({
  el: '#app',
  components: {
      App
  }
})
```
这种写法就是完全把 App 当成一个组件使用, 所以我们需要在模板里添加<App></App>
注意: 上面这种写法需要在 webpack 配置别名, 这种情况下, App 组件并不是根组件
```
module.exports = {
  resolve: {
    alias: {
      'vue$': 'vue/dist/vue.common.js'
    }
  }
}
```
方式 5

模板文件:
```
<div id="app"></div>
```
main.js 入口文件:
```
import App from './app.vue'
new Vue({
  el: '#app',
  template: '<App/>',
  components: {
      App
  }
})
```
这种写法就也是完全把 App 当成一个组件使用, 不过模板直接写在了template选项里
注意: 上面这种写法需要在 webpack 配置别名, 这种情况下, App 组件并不是根组件
```
module.exports = {
  resolve: {
    alias: {
      'vue$': 'vue/dist/vue.common.js'
    }
  }
}
```


Vue实例

通过Vue开发单页面项目，一个项目就是一个Vue实例（暂且这么理解吧），也就是通过new Vue()语句创建的对象。那么既然有了一个实例，那么要显示出来，就得放入HTML文档中，这就是挂载。对应语法：
```
new Vue({
    // el是实例挂载点，会将根组件替换掉原文档中id为 app 标签
    el: '#app',
    // 通过render函数渲染
    render: h => {
        // 这里App是根组件
        h(App)
    }
})

// 第二种写法
new Vue({
    el: '#app',
    // 通过字符串模板
    template: '<App />',
    components: { App }
})

作者：红顏漠
链接：http://www.jianshu.com/p/56ba7488a4af
來源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
Vue扩展实例

严格来说，通过new Vue()方式创建的实例叫做根实例，而还有一种叫扩展实例。这里引用官方的一句话：所有的 Vue.js 组件其实都是被扩展的 Vue 实例（其实我也是刚刚理解）。

扩展实例创建方式：
```
const MyComponent = Vue.extend({
    // 该对象就相当于 单文件组件中 export 导出的对象
    // 这就是为什么说 所有的Vue组件都是被扩展的Vue实例
})
// 创建扩展实例
const component = new MyComponent()

作者：红顏漠
链接：http://www.jianshu.com/p/56ba7488a4af
來源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
同样的，Vue扩展实例也需要挂载，其语法与根实例挂载一样：
```
// 会替换掉 HTML 文档中 id 为 app 的标签
new MyComponent({el: '#app'})
// 但是这里不建议这么做，因为这样会与实例挂载点冲突
// 这里仅仅是为了说明扩展实例与 根实例的相似之处
```
扩展实例一个重要的用法就是在需要的时候才被插入到HTML文档中。比如点击一个按钮，弹出一个模态（modal）框。在我的Demo中，正是通过该方法实现一个加载等待的模态框。通过此方法有一个好处就是，可以将该功能封装成一个Vue插件，需要的时候通过一条指令就可以将组件插入到文档中，不需要预先将组件写入到HTML文档中。方法如下：
```
// 创建扩展实例
let component = new MyComponent()
// 挂载到虚拟DOM中
// 不传递参数，若传递参数会挂载到指定的地方
component  = component.$mount()
// 通过原生语法将 扩展实例添加到HTML文档中
document.body.appendChild(component.$el)

```

同样，Vue根实例也可以通过该方式挂载到HTML中。

### 4、vue-router 路由跳转传值问题
vue如何将点击登录返回的导航栏数据传给导航栏组件 vuex
```javascript
//index.js
import Vue from 'vue'
import Vuex from 'vuex'

import ma from './ma/index'
import mb from './mb/index'


Vue.use(Vuex);

export default new Vuex.Store({
	state: {
		list: [{"id":1, "name": "001"}]
	},
	mutations: {
		ADDITEM: function(argState, item){
	        argState.list.push(item);
	    }
	},
	getters: {
		getList:function(argState){
	        return argState.list;
	    }
	},
	actions: {
		addItem:function(dis,item){
	        dis.commit('ADDITEM',item);
	    }
	},
    modules: {
        ma,
        mb
    }
});
```
```javascript
//ma.js
export default {
	state: {
		count: 12
	},
	mutations: {
		increat(state, i){
			 return state.count++
		}
	},
	getters: {
		getCount(state){
			return state.count
		}
	},
	actions: {
		aaa(){

		}
	}
}
```
```html
hello.vue
<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <h1 @click="abcd">{{getCount}}</h1>
    <!-- <h2 @click="abcd">Hello Vue{{dd}}</h2> -->
    <router-view></router-view>
    <EeLe></EeLe>
    <EeLe></EeLe>
    <button @click="goto">goto</button>
    <ul>
      <li><a href="http://localhost:8080/#/scrollBehavior">Core Docs</a></li>

    </ul>
    <br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>
    <h2>Ecosystem</h2>
    <ul>
      <li><a href="http://router.vuejs.org/" target="_blank">vue-router</a></li>
      <li><a href="http://vuex.vuejs.org/" target="_blank">vuex</a></li>
      <li><a href="http://vue-loader.vuejs.org/" target="_blank">vue-loader</a></li>
      <li><a href="https://github.com/vuejs/awesome-vue" target="_blank">awesome-vue</a></li>
    </ul>
    <ul>
      <li><a href="http://router.vuejs.org/" target="_blank">vue-router</a></li>
      <li><a href="http://vuex.vuejs.org/" target="_blank">vuex</a></li>
      <li><a href="http://vue-loader.vuejs.org/" target="_blank">vue-loader</a></li>
      <li><a href="https://github.com/vuejs/awesome-vue" target="_blank">awesome-vue</a></li>
    </ul>
    <ul>
      <li><a href="http://router.vuejs.org/" target="_blank">vue-router</a></li>
      <li><a href="http://vuex.vuejs.org/" target="_blank">vuex</a></li>
      <li><a href="http://vue-loader.vuejs.org/" target="_blank">vue-loader</a></li>
      <li><a href="https://github.com/vuejs/awesome-vue" target="_blank">awesome-vue</a></li>
    </ul>
    <ul>
      <li><a href="http://router.vuejs.org/" target="_blank">vue-router</a></li>
      <li><a href="http://vuex.vuejs.org/" target="_blank">vuex</a></li>
      <li><a href="http://vue-loader.vuejs.org/" target="_blank">vue-loader</a></li>
      <li><a href="https://github.com/vuejs/awesome-vue" target="_blank">awesome-vue</a></li>
      localStorage.getItem("token");
      localStorage.setItem("token", JSON.stringify(token));
    </ul>
    -------------------------------v-if="item.length!=0"
    <div >
      <div v-for="(item, index) in arr" >
        <div v-if="item.length != 1">{{item}}</div>
      </div>
    </div>
    <!-- <div >
      <div v-for="(item, index) in arr" >
        <div v-for="i in item">{{i}}</div>
      </div>
    </div> -->
  </div>
</template>

<script>

// import {s} from '../js/common'
import a from '../js/common.js'
import EeLe from './ele.vue'
import {mapGetters} from 'vuex'

export default {
  name: 'hello',
  data () {
    return {
      msg: 'Hello Vue',
      arr: ['1','2',['3','a','s','d'],'4','5','6'],
      dd: this.$store.getters.getCount
      // dd: getCount()


      // dd: this.$store.getters.getList
    }
  },
  methods: {
    goto(){
      this.$router.push({name: "Names"})
    },
    abcd(){
      // return this.$store.commit('incre')
      // this.$store.commit('ADDITEM',{"id":2,"name": '不不不'});
    this.$store.commit('increat')
    console.log(this.$store.getters.getCount)
    }
  },
  created() {
    // console.log(this.$store.commit);
    // console.log(s());
    // console.log(this.$store.getters)
    a('aaa');
  },
  computed: {
    // ...mapGetters({
    //     getCount: state => state.ma.getCount
    //   })
    getCount () {
      return this.$store.getters.getCount
    }
  },
  components: {
    EeLe
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>

```
```javascript
// The Vue build version to load with the `import` command
// (runtime-only or standalone) has been set in webpack.base.conf with an alias.
import Vue from 'vue'
import Vuex from 'vuex'
import App from './App'
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-default/index.css'
import router from './router'
import store from './store/index'

Vue.config.productionTip = false

Vue.use(ElementUI)
Vue.use(Vuex)

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  store,
  template: '<App/>',
  components: { App }
})

```
### 5、js深拷贝浅拷贝
[https://zhuanlan.zhihu.com/p/26282765](https://zhuanlan.zhihu.com/p/26282765)

[http://www.cnblogs.com/y8932809/p/5863764.html](http://www.cnblogs.com/y8932809/p/5863764.html)

### 6、使用toString()方法来检测对象类型
```javascript
var toString = Object.prototype.toString;

toString.call(new Date); // [object Date]
toString.call(new String); // [object String]
toString.call(Math); // [object Math]

//Since JavaScript 1.8.5
toString.call(undefined); // [object Undefined]
toString.call(null); // [object Null]
```
参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/toString#Using_toString_to_detect_object_type)

只读属性
<b>Element.clientWidth </b>属性表示元素的内部宽度，以像素计。该属性包括内边距，但不包括垂直滚动条（如果有）、边框和外边距。

jQuery：

width() - 返回元素的宽度。 
height() - 返回元素的高度。 
innerWidth() 方法返回元素的宽度（包括内边距）。 
innerHeight() 方法返回元素的高度（包括内边距）。                    
outerWidth() 方法返回元素的宽度（包括内边距和边框）。               
outerHeight() 方法返回元素的高度（包括内边距和边框）。              
outerWidth(true) 方法返回元素的宽度（包括内边距、边框和外边距）。   
outerHeight(true) 方法返回元素的高度（包括内边距、边框和外边距）。   
返回文档（HTML 文档）$(document).height()的高度 
返回窗口（浏览器视口）$(window).height()的高度

JavaScript 深拷贝（有问题）
```javascript

var obj = {
	a: 1,
	b: "2",
	c: function(){
		console.log("3")
	},
	d: {
		e: 4,
		f: function(){
			console.log("5")
		}
	},
	g: [
		{h: 1},
		{i: function(){console.log("6")}}
	]
};

var newObj = {};

function copy(newObj, obj){
	for(var i in obj){
		if(Object.prototype.toString.call(obj[i]) === "[object Object]"){
			newObj[i] = {};
			copy(newObj[i], obj[i]);
		}if(Object.prototype.toString.call(obj[i]) === "[object Array]"){
			newObj[i] = [];
			copy(newObj[i], obj[i]);
		}else {
			newObj[i] = obj[i];
		}
	}
	console.log(newObj);
	// return newObj;
}

copy(newObj, obj);

newObj.d.e = 22;
console.log( newObj.d.e );        // 22
console.log( obj.d.e );           // 22

```



```
Pip3 install beautifulsoup4
```
python 爬虫 爬取信息后编码问题

错误信息： UnicodeEncodeError: 'gbk' codec can't encode character '\xbb' in position 13698: illegal multibyte sequence

[CSDN](http://blog.csdn.net/jim7424994/article/details/22675759)

### vue上拉加载
```html
<template lang="html">
  <div class="yo-scroll"
  :class="{'down':(state===0),'up':(state==1),refresh:(state===2),touch:touching}"
  @touchstart="touchStart($event)"
  @touchmove="touchMove($event)"
  @touchend="touchEnd($event)"
  @scroll="(onInfinite || infiniteLoading) ? onScroll($event) : undefined">
    <section class="inner" :style="{ transform: 'translate3d(0, ' + top + 'px, 0)' }">
      <header class="pull-refresh">
        <slot name="pull-refresh">
           <span class="down-tip">下拉更新</span>
           <span class="up-tip">松开更新</span>
           <span class="refresh-tip">更新中</span>
        </slot>
      </header>
      <slot></slot>
      <footer class="load-more">
        <slot name="load-more">
          <span>加载中……</span>
        </slot>
      </footer>
    </section>
  </div>
</template>

<script>
export default {
  props: {
    offset: {
      type: Number,
      default: 40
    },
    enableInfinite: {
      type: Boolean,
      default: true
    },
    enableRefresh: {
      type: Boolean,
      default: true
    },
    onRefresh: {
      type: Function,
      default: undefined,
      required: false
    },
    onInfinite: {
      type: Function,
      default: undefined,
      require: false
    }
  },
  data () {
    return {
      top: 0,
      state: 0,
      startY: 0,
      touching: false,
      infiniteLoading: false
    }
  },
  methods: {
    touchStart (e) {
      this.startY = e.targetTouches[0].pageY
      this.startScroll = this.$el.scrollTop || 0
      this.touching = true
    },
    touchMove (e) {
      if (!this.enableRefresh || this.$el.scrollTop > 0 || !this.touching) {
        return
      }
      let diff = e.targetTouches[0].pageY - this.startY - this.startScroll
      if (diff > 0) e.preventDefault()
      this.top = Math.pow(diff, 0.8) + (this.state === 2 ? this.offset : 0)

      if (this.state === 2) { // in refreshing
        return
      }
      if (this.top >= this.offset) {
        this.state = 1
      } else {
        this.state = 0
      }
    },
    touchEnd (e) {
      if (!this.enableRefresh) return
      this.touching = false
      if (this.state === 2) { // in refreshing
        this.state = 2
        this.top = this.offset
        return
      }
      if (this.top >= this.offset) { // do refresh
        this.refresh()
      } else { // cancel refresh
        this.state = 0
        this.top = 0
      }
    },
    refresh () {
      this.state = 2
      this.top = this.offset
      this.onRefresh(this.refreshDone)
    },
    refreshDone () {
      this.state = 0
      this.top = 0
    },

    infinite () {
      this.infiniteLoading = true
      this.onInfinite(this.infiniteDone)
    },

    infiniteDone () {
      this.infiniteLoading = false
    },

    onScroll (e) {
      if (!this.enableInfinite || this.infiniteLoading) {
        return
      }
      let outerHeight = this.$el.clientHeight
      let innerHeight = this.$el.querySelector('.inner').clientHeight
      let scrollTop = this.$el.scrollTop
      let ptrHeight = this.onRefresh ? this.$el.querySelector('.pull-refresh').clientHeight : 0
      let infiniteHeight = this.$el.querySelector('.load-more').clientHeight
      let bottom = innerHeight - outerHeight - scrollTop - ptrHeight
      if (bottom < infiniteHeight) this.infinite()
    }
  }
}
</script>
<style>
.yo-scroll {
  position: absolute;
  top: 2.5rem;
  right: 0;
  bottom: 0;
  left: 0;
  overflow: auto;
  -webkit-overflow-scrolling: touch;
  background-color: #ddd
}
.yo-scroll .inner {
  position: absolute;
  top: -2rem;
  width: 100%;
  transition-duration: 300ms;
}
.yo-scroll .pull-refresh {
  position: relative;
  left: 0;
  top: 0;
  width: 100%;
  height: 2rem;
  display: flex;
  align-items: center;
  justify-content: center;
}
.yo-scroll.touch .inner {
  transition-duration: 0ms;
}
.yo-scroll.down .down-tip {
  display: block;
}
.yo-scroll.up .up-tip {
  display: block;
}
.yo-scroll.refresh .refresh-tip {
  display: block;
}
.yo-scroll .down-tip,
.yo-scroll .refresh-tip,
.yo-scroll .up-tip {
  display: none;
}
.yo-scroll .load-more {
  height: 3rem;
  display: flex;
  align-items: center;
  justify-content: center;
}  
</style>
```
```html
// 要点：touch事件、要阻止事件冒泡、下拉刷新state有状态1（下拉）2（松开更新）3（正在更新）4（更新完成）0（默认）、this.$el获取顶级元素、获取scrollTop再进行事件操作
<template>
        <div class="wrapper" ref="menuWrapper">
        <ul 
          @touchstart="touchStart($event)"
          @touchmove="touchMove($event)"
          @touchend="touchEnd($event)"
          :style="{ transform: 'translate3d(0, ' + top + 'px, 0)',
          transition: ms+'ms'
          }"
         class="content">
          <li>...</li>
          <li>...</li>
          <li><span>123</span></li>
          <li>...</li>
          <li>...</li>
          <li>...</li>
          <li>...</li>
          <li>...</li>
          <li>...</li>
          <li>...</li>
          <li>...</li>
          <li>...</li>
          <li>...</li>
          <li>...</li>
          <li>...</li>
          <li>...</li>
          <li>...</li>
          <li>...</li>
          <li>...</li>
          <li>...</li>
          <li>...</li>
          <li>...</li>
        </ul>
      </div>
</template>
<style scoped>
  .wrapper{
    position: relative;
    width: 200px;
    height: 200px;
    overflow: scroll;
    background: #ccc;
  }
  .content li{
    line-height: 20px;
  }
</style>
<script>
  // import BScroll from 'better-scroll'


  export default{
    props: {

    },
    data () {
      return {
        startY: 0,
        top: 0,
        ms: 0
      }
    },
    computed: {

    },
    mounted () {
    },
    created () {
    },
    methods: {
      touchStart(e){
        this.ms = 0
        this.startY = e.targetTouches[0].pageY
      },
      touchMove (e) {
        this.top = (e.targetTouches[0].pageY - this.startY) *0.75
        console.log(this.$el.scrollTop)
        e.preventDefault()
      },
      touchEnd (e) {
        this.top = 0
        this.ms = 300
        e.preventDefault()
      }
    }
  }
</script>
```
### vue中使用les、sass
```
npm install less less-loader --save
```
```
npm install sass-loader node-sass --save-dev 
```
### v-for 问题
获取某一个<b>1</b>
```html
<div v-if="i==index" v-for="(v, i) in items">{{v}}</div>
```

### vue 轮播图 vue-awesome-swiper
```html
<template>
    <swiper :options="swiperOption" class="swiper-box" ref="mySwiper">
        <swiper-slide class="swiper-item">Slide 1</swiper-slide>
        <swiper-slide class="swiper-item">Slide 2</swiper-slide>
        <swiper-slide class="swiper-item">Slide 3</swiper-slide>
        <div class="swiper-pagination" slot="pagination"></div>
    </swiper>
</template>

<script>
import { swiper, swiperSlide } from 'vue-awesome-swiper'

require('swiper/dist/css/swiper.css')
export default {
    components: {
        swiper,
        swiperSlide
    },
    name: 'hello',
    data() {
        return {
            swiperOption: {
                notNextTick: true,
                pagination: '.swiper-pagination',
                slidesPerView: 'auto',
                centeredSlides: true,
                paginationClickable: true,
                spaceBetween: 30,
                onSlideChangeEnd: swiper => {
                    //这个位置放swiper的回调方法
                    this.page = swiper.realIndex + 1;
                    this.index = swiper.realIndex;
                }
            }
        }
    }, computed: {
      swiper() {
        return this.$refs.mySwiper.swiper
      }
    },
    mounted() {
        //这边就可以使用swiper这个对象去使用swiper官网中的那些方法
        this.swiper.slideTo(0, 0, false);
    }
}
</script>
```

### vue-cli 代理跨域
```
 proxyTable: {
        '/api': {
            target: 'http://api.douban.com',
            changeOrigin: true,
            pathRewrite: {
              '^/api': '/'
            }
          }
    },
```






