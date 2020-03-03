## 模块化

**模块化就是把系统分离成独立功能的方法，这样我们需要什么功能，就加载什么功能**

- 每个模块都是独立的，良好设计的模块会尽量与外部代码撇清关系，以便于独立对其改进和维护
- 可以重复利用，而不用经常复制自己之前写的代码

## 模块化主要解决两个问题，“命名冲突”、“文件依赖”

**命名冲突**

```javascript
// a.js
var a = 1;

// b.js
var a = 2;
```

**文件依赖**

```javascript
// b.js依赖a.js，标签的书写顺序必须是：
<script src='a.js' type='text/javascript'></script>
<script src='b.js' type='text/javascript'></script>
```

这样在很多人开发的时候很难协调啊，令人头疼的问题

### 结局冲突、依赖（按需加载）

**使用命名空间**

这样的写法会暴露所有模块内的成员，内部状态可以被外部改写

```javascript
let module = {
  name: 'likang xie',
  sayName() {
    console.log(this.name);
  }
}
```

**立即执行函数+闭包**

函数内部有自己独立的作用域，外部只能访问自己暴露的成员而不能读取内部的私有成员

```javascript
let module = (function () {
  let privateName = 'private'; // 私有变量
  let privateFn = function () {}; // 私有函数
  
  // 对外暴露的成员
  return {
    name: 'likang xie', // 公有属性
    sayName() { // 公有方法
      console.log(this.name);
    }
  }
})();

// 外部调用
module.sayName(); // likang xie 
```

**使用立即执行函数+类**

同上

```javascript
const People = (function () {

  let privateName = 'private'; // 私有变量
  let fn = function () {}; // 私有方法

  return class People {
    constructor () {
      this.name = 'likang xie'; // 公有变量
    }
  
    // 公有方法
    sayName() {
      console.log(this.name);
    }
   }

})()
```



## CommonJs（用于Node环境）

根据CommonJs规范，每个文件就是一个模块，有自己的作用域。在一个文件里面定义的变量、函数、类，都是私有的，对其他文件不可见，**CommonJS规范加载模块是同步的，也就是说，加载完成才可以执行后面的操作**，Node.js主要用于服务器编程，模块一般都是存在本地硬盘中，加载比较快，**所以Node.js采用CommonJS规范。**

### 定义模块

```javascript
// module.js
let name = 'liakng xie';
let sayName = function () {
  console.log(name);
};

module.exports = { name, sayName }

// 或者
exports.sayName = sayName;
```

### 加载模块

```javascript
// 通过 require 引入依赖
let module = require('./module.js');
module.sayName(); // likang xie
```

### 注意：module.export和exports的区别

1.module.exports方法还可以单独返回一个数据类型（String，Number，Object...），exports**只能返回一个Object对象**

2.所有的exports对象**最终都是通过module.exports传递执行**，因此可以更确切的说，exports是module.exports添加属性和方法

```javascript
exports.name = 'likang xie';
exports.age = 21;
console.log(module.exports); // 运行结果：{ name: 'likang xie', age: 21 }
```

3.同时用到module.exports和exports的时候

```javascript
// 情况1
module.exports = { a: 1 }
exports.b = 2;
 // { a:1 }
console.log(module.exports);

// 情况2
module.exports.a =1;
exports.b = 2;
 // { a:1, b:2 }
console.log(module.exports);
```



## AMD（用于浏览器环境RequireJs）

AMD是"Asynchronous Module Definition"的简写，也就是异步模块定义。**它采用异步方式加载模块**。通过define方法去定义模块，require方法去加载模块。

**定义模块**

```javascript
define(['module'], function() {
  let name = 'likang xie';

  function sayName() {
    console.log(name);
  }
  
  return { sayName }
})
```

**使用模块**

```javascript
// 通过 require 引入依赖
require(['module'], function(mod) {
   mod.sayName(); // likang xie
})
```



## CMD（用于浏览器环境）(SeaJs)

CMD与AMD用法很相似

**定义模块、使用模块**

```javascript
define(function(require, exports, module) {
  // 通过 require 引入依赖
   var otherModule = require('./otherModule');

   // 通过 exports 对外提供接口
   exports.myModule = function () {};

   // 或者通过 module.exports 提供整个接口
   module.exports = function () {};
})
```

### 注意：CMD和AMD

#### 不同点

1. **定位有差异**。RequireJS 想成为浏览器端的模块加载器，同时也想成为 Rhino / Node 等环境的模块加载器。Sea.js 则专注于 Web 浏览器端，同时通过 Node 扩展的方式可以很方便跑在 Node 环境中。
2. **遵循的规范不同**。RequireJS 遵循 AMD（异步模块定义）规范，Sea.js 遵循 CMD （通用模块定义）规范。规范的不同，导致了两者 API 不同。Sea.js 更贴近 CommonJS Modules/1.1 和 Node Modules 规范。
3. **推广理念有差异**。RequireJS 在尝试让第三方类库修改自身来支持 RequireJS，目前只有少数社区采纳。Sea.js 不强推，采用自主封装的方式来“海纳百川”，目前已有较成熟的封装策略。
4. **对开发调试的支持有差异**。Sea.js 非常关注代码的开发调试，有 nocache、debug 等用于调试的插件。RequireJS 无这方面的明显支持。
5. **插件机制不同**。RequireJS 采取的是在源码中预留接口的形式，插件类型比较单一。Sea.js 采取的是通用事件机制，插件类型更丰富。
6. 对于依赖的模块，AMD 是**提前执行**，CMD 是**延迟执行**。不过 RequireJS 从 2.0 开始，也改成可以延迟执行（根据写法不同，处理方式不同）。CMD 推崇 as lazy as possible。
7. CMD 推崇**依赖就近**，AMD 推崇**依赖前置**。看代码：

```javascript
// CMD
define(function(require, exports, module) {
   var a = require('./a')
   a.doSomething()
   // 此处略去 100 行
   var b = require('./b') // 依赖可以就近书写
   b.doSomething()
   // ... 
})

// AMD 默认推荐的是
define(['./a', './b'], function(a, b) {  // 依赖必须一开始就写好
    a.doSomething()
    // 此处略去 100 行
    b.doSomething()
    ...
})
```

虽然 AMD 也支持 CMD 的写法，同时还支持将 require 作为依赖项传递，但 RequireJS 的作者默认是最喜欢上面的写法，也是官方文档里默认的模块定义写法。

8. AMD 的 API 默认是**一个当多个用**，CMD 的 API 严格区分，推崇**职责单一**。比如 AMD 里，require 分全局 require 和局部 require，都叫 require。CMD 里，没有全局 require，而是根据模块系统的完备性，提供 seajs.use 来实现模块系统的加载启动。CMD 里，每个 API 都**简单纯粹**。

9. 等等。

#### 相同点

RequireJS 和 Sea.js 都是模块加载器，倡导模块化开发理念，核心价值是让 JavaScript 的模块化开发变得简单自然。



## ES6模块（用于浏览器环境）

模块功能主要由两个命令构成：**export和import**。export命令用于规定模块的对外接口，import命令用于输入其他模块提供的功能。

**定义模块、输出变量**

a.js

```javascript
export default { name: 'likang xie' }
```

b.js

```javascript
// 输出多个变量
export let name = 'likang xie';
export let sayName = () => console.log(name);
```

**使用模块**

```javascript
import people from 'a.js';
console.log(people); // { name: 'likang xie' }
```

```javascript
// 将所有获取到的变量存到people里
import * as people from 'b.js';
console.log(people); // 一个module对象 { name: 'likang xie', sayName: ....}

// 或者
import { name, sayName } from 'b.js';
```

### 引用：[ES6在浏览器中使用](https://segmentfault.com/a/1190000014342718)

