## 先聊聊观察者模式

观察者模式，又叫订阅-发布模式，定义了一种一对多的关系。让多个对象同时观察某一个主题对象的变化，从而执行相对应的回调函数。现在就来看一下观察者模式的实现思想：

- 发布者（某一主题对象）需包含一个数组属性用于存储订阅者对象；
- 订阅时：直接将订阅者`Push`到发布者数组属性中；
- 退订时：通过循环遍历，将需要退订的订阅者从发布者的数组属性中删除；
- 发布时：通过循环遍历，逐一执行发布者数组属性中每个元素的回调函数；

该模式思想大概了解了，那用 JavaScript 究竟如何实现呢？现在就来瞅瞅：

```javascript
// 发布者
function Publisher() {
  this.watcher = []
}
Publisher.prototype.addWatcher = obj => {
  this.watcher.push(obj)
}
Publisher.prototype.publish = obj => {
  this.watcher.forEach(item => { typeof item.update === 'function' && item.update() })
}

// 订阅者
function Subscriber(name) {
  this.name = name
}
Subscriber.prototype.update = function() {
  console.log(`${this.name} is updated.........`)
}
```

既然提到观察者模式，那肯定和本主题相关联。首先在 Vue 实现的依赖收集以及依赖更新中，其实就是用到了观察者模式，响应式属性就是发布者，而 Watcher 依赖就是订阅者。每当响应式属性发生更新时，都会直接通知到其 dep 中保存的所有订阅者进行响应的更新。

接下来，就让我们瞅瞅在 Vue 中是如何实现依赖更新的。



## 从源码角度分析

从前面的章节中，我有提及过，涉及到响应式属性的依赖更新，主要有两个地方，分别是 Setter 函数和全局`$set`方法。接下来我们就分别来看看 🤔 。

1. Setter 函数中依赖更新

   前面提到，只有 Vue 能监测到的更新才会主动触发 Setter 函数，先看下源码中的 Setter 函数是如何处理的：

   ```javascript
   function defineReactive$$1 (
   	obj,
    	key,
    	val,
    	customSetter,
    	shallow
   ) {
       // ...
       Object.defineProperty(obj, key, {
         enumerable: true,
         configurable: true,
         // ...
         set: function reactiveSetter (newVal) {
           // ...
           dep.notify();
         }
       })
     }
   
   Dep.prototype.notify = function notify () {
     var subs = this.subs.slice(); // 数组的浅复制，避免影响到元素组
     // ...
     for (var i = 0, l = subs.length; i < l; i++) {
       subs[i].update();
     }
   }
   ```

   上述代码很好理解，当响应式属性更新时，会直接触发到 Setter 函数，这时候就会调用 dep 对象上的 notify 方法，该方法做的事情就是遍历其保存的 Watcher 依赖，并逐个触发其更新回调函数。

   看到这里，也许你还会有一个疑问，那就是 Watcher 依赖的更新回调函数是如何产生的？

   其实在元素挂载时所创建的 Watcher 依赖就已经定义了其更新回调函数，现在就来看看

   ```javascript
   function mountComponent (
   	vm,
    	el,
    	hydrating
   ) {
       // ...
       var updateComponent;
       updateComponent = function () { // 更新回调函数的设置
         vm._update(vm._render(), hydrating);
       };
       new Watcher(vm, updateComponent, noop, {
         before: function before () {
           if (vm._isMounted && !vm._isDestroyed) {
             callHook(vm, 'beforeUpdate');
           }
         }
       }, true /* isRenderWatcher */);
       // ...
     }
   
   var Watcher = function Watcher (
   	vm,
    	expOrFn,
    	cb,
    	options,
    	isRenderWatcher
   ) {
       // ...
       if (typeof expOrFn === 'function') { // expOrFn 就是更新回调函数
         this.getter = expOrFn;
       }
       // ...
     }
   
   Watcher.prototype.update = function update () {
     /* istanbul ignore else */
   	// ...
     queueWatcher(this); // 利用队列形式进行更新每一个 Watcher 依赖，该方法中最后都会直接执行其更新回调函数
   };
   
   ```

   可以看到的是，组件在挂载的过程中，通过 Watcher 依赖本身的 getter 属性进行收集更新回调函数。另外，当执行 Watcher 依赖的 update 方法时，即相当于执行了 queueWatcher 方法。

    queueWatcher 方法需要探讨一下的是，会将 Watcher 依赖推进到一个队列中，然后再按顺序滴拿出来进行更新，而所谓的更新就是直接调用了其 Watcher 依赖本身的 getter 函数，从而更新依赖中响应式属性的值。

   

2. 全局`$set`方法中依赖更新

   在依赖收集的章节中，我有说过，官方提供的全局`$set`方法，其实就是利用`__ob__`中保存的依赖进行设置和更新的。现在我们就来看看源码究竟是如何设置和更新的：

   ```javascript
   Vue.prototype.$set = set;
   
   /**
    * Set a property on an object. Adds the new property and
    * triggers change notification if the property doesn't
    * already exist.
    */
   function set (target, key, val) {
     // ...
     var ob = (target).__ob__;
     // ...
     defineReactive$$1(ob.value, key, val); // 对新添加属性进行响应式处理
     ob.dep.notify(); // 初始化响应式处理后，同时通知更新
   }
   ```

   由代码中 set 方法注释可以看到，在响应式对象上新增属性时触发更新通知。在触发更新前，会先把新增的属性进行响应式处理 defineReactive$$1 ，紧接着就是根据此前存储的`__ob__`中依赖进行逐一通知更新，其中更新过程和上述调用 Watcher 依赖的 update 方法是一样的。



现在来总结一下：

- Vue 中实现依赖更新是使用了观察者模式，通过 Getter 函数和`__ob__`中收集的依赖进行逐一更新；
- 对响应式属性的处理会主动触发 Setter 函数，然后使用 Getter 函数中收集的依赖进行逐一更新。对非响应式属性的处理，则需要调用全局 $set 方法，使用`__ob__`中收集的依赖进行逐一更新并且把该**新增属性进行响应式处理**；
- 更新回调函数会在组件挂载时所创建的 Watcher 依赖进行设置，在通知更新时，会直接使用队列的方式进行更新，先到先处理；























