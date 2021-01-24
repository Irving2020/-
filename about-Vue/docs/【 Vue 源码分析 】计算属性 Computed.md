## 先理解计算属性

一提及到 Vue 的计算属性，想必大家都会想到其最大的优点——**缓存**。官方文档也有提到

> 对于任何复杂逻辑，你都应当使用**计算属性**。

如何理解？不是可以直接使用方法处理吗？话不多说，直接看一个栗子 🌰 ：

```javascript
// html
<div>
  <span>{{ allName }}</span>
	<span>{{ setAllName() }}</span>
	<span>{{ tag }}</span>
</div>

// js
const app = new Vue({
  data: {
    firstName: 'haha',
    lastName: 'hehe',
    tag: false
  },
  computed: {
    allName() {
      return firstName + ' ' + lastName
    }
  },
  methods: {
    setAllName() {
      return firstName + ' ' + lastName
    }
  }
})
```

相信很多同学对于上述的代码并不陌生，在界面上展示时，`allName`和`setAllName()`显示的效果是一样的，并没有什么区别。那真的就是没有任何区别了吗？

其实不是的，当我把变量`tag`的值设为 true 时，虽然展示的效果一样，但是两者背后渲染原理完全不同。其中就是上面提到的缓存功能。

为何这么说？你只要记住一点，**计算属性是基于它们的响应式依赖进行缓存的**。因此，当把变量`tag`的值设为 true 时，界面会重新渲染，由于计算属性`allName`是基于变量 firstName 和 lastName 的，而这两者并没有变化，所以会直接返回此前保存的计算值回来。相反，方法则不大一样了，因为重新渲染时，模板解析`setAllName`方法时，是无法辨别其有没有使用过该方法的，所以又会重新计算一遍并返回值回来。最终**在处理效率上，计算属性是完胜方法处理的**。

另一方面，既然计算属性是基于响应式依赖进行计算与否的，那么它也可以很好滴取代掉 Watch 侦听。什么意思？

如果你以前用过 Angular 的话，相信你会有一个很大的体会，那就是最喜欢滥用 watch 来监听某个属性是否变化，然后来进行相应的处理，那么使用计算属性的出现就很好滴避免了滥用 watch 行为，让代码在后期的维护和理解上都会有一个质的提升。官方文档中也有提到

> 通常更好的做法是使用计算属性而不是命令式的 `watch` 回调。

在这里，真的由不得感叹一下，尤大大可以发明出计算属性出来。👏🏽

既然上面都说到计算属性有那么牛，那究竟它是如何实现的呢？在这里，我们可以从两方面进行入手，分别是：

- 计算属性是如何进行计算的？
- 计算属性是如何进行缓存的？



## 从源码角度进行分析

让我们先从初始化阶段是如何处理计算属性的，上源码

```javascript
var computedWatcherOptions = { lazy: true }; // 关键中的关键，用于缓存作用的

function initState (vm) {
  // ...
  if (opts.computed) { initComputed(vm, opts.computed); } // 先判断是否已定义计算属性，若有则直接初始化计算属性
}

function initComputed (vm, computed) {
  var watchers = vm._computedWatchers = Object.create(null); // 创建一个纯对象，新建依赖所用
  // ...
  for (var key in computed) {
    var userDef = computed[key]; // 获取当前用户定义的计算属性
    var getter = typeof userDef === 'function' ? userDef : userDef.get; // 判断当前定义的计算属性是否为函数，若不是则直接获取其 Getter
    if (getter == null) { // 一旦 getter 不是函数时，就给出警告（蕴含着知识点，undefined == null）
      warn(
        ("Getter is missing for computed property \"" + key + "\"."),
        vm
      );
    }
    if (!isSSR) { // 不是服务端渲染时，就直接为每个计算属性创建一个相关依赖
      // create internal watcher for the computed property.
      watchers[key] = new Watcher(
        vm,
        getter || noop,
        noop,
        computedWatcherOptions // 关键中的关键，用于缓存作用的
      );
    }
    if (!(key in vm)) { // 判断是否为新创建的计算属性
      defineComputed(vm, key, userDef);	// 关键函数，主要负责计算属性响应式处理
    } 
  }
}
```

理解这段代码并不难，主要有以下几个知识点：

- 使用`Object.create(null)`创建一个纯对象（相关链接，可看其他话题中[Object.create(null) 与 {} 究竟有什么不一样](https://github.com/Andraw-lin/about-Vue/blob/master/docs/%E3%80%90%20Vue%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E3%80%91%E8%AE%A1%E7%AE%97%E5%B1%9E%E6%80%A7%20Computed.md)）
- 判断计算属性定义时的类型
- 为每个计算属性创建各自的依赖
- 新定义的计算属性，使用`defineComputed`方法进行响应式处理

可以看到，每个计算属性本身就会被作为一个 Watcher 依赖，现在就来看看`defineComputed`方法是如何进行响应式处理的

```javascript
var sharedPropertyDefinition = {
  enumerable: true,
  configurable: true,
  get: noop,
  set: noop
};

function defineComputed (
	target,
 	key,
 	userDef
) {
    var shouldCache = !isServerRendering(); // 是否为服务端渲染
    if (typeof userDef === 'function') { // 针对计算属性直接定义为函数的形式
      sharedPropertyDefinition.get = shouldCache
        ? createComputedGetter(key) // 关键函数，主要负责计算属性的缓存以及与 $data 建立依赖收集
        : createGetterInvoker(userDef);
      sharedPropertyDefinition.set = noop;
    } else { // 一旦定义的不是函数类型，则直接从 Getter 里面去拿
      sharedPropertyDefinition.get = userDef.get
        ? shouldCache && userDef.cache !== false
          ? createComputedGetter(key) // 关键函数，主要负责计算属性的缓存以及与 $data 建立依赖收集
          : createGetterInvoker(userDef.get)
        : noop;
      sharedPropertyDefinition.set = userDef.set || noop;
    }
    if (sharedPropertyDefinition.set === noop) { // 同时针对 Setter 未设置，默认配置一个警告函数
      sharedPropertyDefinition.set = function () {
        warn(
          ("Computed property \"" + key + "\" was assigned to but it has no setter."),
          this
        );
      };
    }
    Object.defineProperty(target, key, sharedPropertyDefinition); // 进行响应式的处理
  }
```

由此可以看到，每个计算属性除了被定义为 Watcher 外，还会作一个响应式处理。便于建立与页面调用时的依赖，一旦页面更新后，同时判断是否需要重新计算，由于计算属性拥有着强大的缓存功能，因此若依赖的响应式属性 data 没有变化时，则会直接返回原来计算的值，避免重新计算。

接下来我们来看看方法`createComputedGetter`是如何处理缓存以及与 $data 建立依赖收集的

```javascript
function createComputedGetter (key) {
  return function computedGetter () {
    var watcher = this._computedWatchers && this._computedWatchers[key]; // 拿到每一个计算属性已经定义好的 Watcher
    if (watcher) {
      if (watcher.dirty) { // 缓存的关键，只有 dirty 为 true 时才会进行重新计算，否则直接获取已经计算好的值
        watcher.evaluate();
      }
      if (Dep.target) { // 建立依赖的关键，建立与页面之间的依赖
        watcher.depend();
      }
      return watcher.value
    }
  }
}
```

也许有同学会有疑惑，究竟 dirty 是什么？以及 Dep.target 又是啥？

我们先来重新回顾一下计算属性在创建 Watcher 时是如何传递值的，以及 Wacther 类中是如何处理的。

```javascript
// ...
// create internal watcher for the computed property.
watchers[key] = new Watcher(
	vm,
  getter || noop,
  noop,
  computedWatcherOptions // 关键中的关键，用于缓存作用的
);
// ...
var Watcher = function Watcher (
	vm,
 	expOrFn, // 更新回调函数
 	cb,
 	options, // 选项配置
 	isRenderWatcher
) {
    // ...
    if (options) {
 			// ...
      this.lazy = !!options.lazy;
      // ...
    }
    // ...
    this.dirty = this.lazy;
  }
```

计算属性在创建 Watcher 依赖时，会传递一个 lazy 为 true 的属性，该属性不是懒惰的意思，可以理解成缓慢变化的意思即不变。将 lazy 直接赋予到 Watcher 的 dirty 属性中（至于 dirty ，可以理解为是否为脏数据）。因此一开始，计算属性在页面中都会被计算一遍。另一方面，由于会把计算属性的 Getter 函数传入到 Wacther 作为更新回调函数使用，一旦依赖的响应式属性变化时，就会调用更新回调函数进行重新计算计算属性的值。

当页面第一次引用了计算属性时，dirty 的值肯定会为 true ，就会调用 Watcher 的 evaluate 方法。接下来，我们来看看 evaluate 方法

```javascript
/**
 * Evaluate the value of the watcher.
 * This only gets called for lazy watchers.
 */
Watcher.prototype.evaluate = function evaluate () {
  this.value = this.get();
  this.dirty = false;
};

/**
 * Evaluate the getter, and re-collect dependencies.
 */
Watcher.prototype.get = function get () {
  pushTarget(this);
  var value;
  var vm = this.vm;
  try { 
    value = this.getter.call(vm, vm); // 引用响应式属性，建立与响应式属性之间的依赖
  // ...
  } finally {
    // ...
    popTarget();
    // ...
  }
  return value
};
```

好明显，evaluate 方法就是开始计算计算属性的值，然后再设置其 dirty 属性为 false，这时候就会被认为是已经更新的值了，下一次在调用时就不会进行重新计算。另一方面，由于在调用 this.getter 函数（即更新回调函数）时，会引用响应式属性并在响应式属性中收集到计算属性的依赖。

那么问题来了，什么时候 dirty 才会被设回 true ？还有就是为什么引用响应式属性就会在响应式属性中收集到计算属性的依赖？现在我来简单解析一下：

- 什么时候 dirty 才会被设回 true ？

  计算属性的 Watcher 只会被收集到相对应的响应式属性中，因此在响应式属性更改后，通知到相对应的 Watcher 进行更新，其实就会在 update 函数中进行设置 dirty 为 false，我们来看看

  ```javascript
  /**
   * Subscriber interface.
   * Will be called when a dependency changes.
   */
  Watcher.prototype.update = function update () {
    /* istanbul ignore else */
    if (this.lazy) {
      this.dirty = true;
    }
    // ...
  };
  ```

  可以看到，由于计算属性的 lazy 会 true，因此在响应式属性更新时，你有没有发现仅仅只是把 dirty 设回为 true ，并没有调用更新回调函数？

  这也是一个**优化点**，就是**要等到页面调用的地方更新时，才会通知计算属性重新计算值**（类似于我们的按需引入一样）。

- 为什么引用响应式属性就会在响应式属性中收集到计算属性的依赖？

  相信看到我之前的文章，应该都知道，响应式属性都会设置 Getter 和 Setter ，因此当计算属性计算值时，会引用到响应式属性，这样机会将计算属性的 Watcher 收集到了响应式属性中。



好了，接下来我们再看看如下代码，究竟是什么用呢？

```javascript
// ...
if (Dep.target) { // 建立依赖的关键，建立响应式属性与页面之间的依赖
  watcher.depend();
}
// ...
```

前面可以看到，当将计算属性被收集到响应式属性中后，会调用`popTarget`方法将`Dep.target`重新指回到页面的 Watcher，这时候调用

`watcher.depend()`后就会将页面的 Watcher 收集到了响应式属性中。这也就说明，当页面调用计算属性时，响应式更改后，设置计算属性的 dirty 为 true 后，并不是由计算属性去通知页面更新的，而是以让有响应式属性通知页面更新后，再去重新计算计算属性的值。

看到这里，是否会迷惑了？😄 其实尤大大是真的很厉害，利用一个标记位来实现计算属性的缓存功能，而且只有在调用到其计算属性时才去开始计算其值，实现了一个按需调用的思想。

我们现在来总结一下：

- 定义计算属性时，会依次为每一个计算属性创建相应的 Watcher，并设置成响应式；
- 当模板中引用到计算属性时，会先将计算属性的 Watcher 收集到相应的响应式属性中，并且计算到值后会把`dirty`缓存标记位设置为 false，这样一来，下次模板更新后，发现`dirty`标记位为 false，就不会重新计算。同时响应式属性会收集到模板的 Wather；
- 当响应式属性更改时，会先设置计算属性的`dirty`缓存标记位为 true，然后通知模板进行更新。模板在更新的过程中发现计算属性的`dirty`缓存标记位为 true，这时候就会重新计算其值并返回；









































