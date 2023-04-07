#es6

> 异步函数

```
function t(){}
function t1(){}
export {t,t1}
//import {t,t1} from './test';
```

> 语法糖

```
//方式1
var t={
  ['todos/get']() {}
};
//等价于
var t={
  'todos/get':function todosGet(){}//todosGet 函数名
};
//方式2
var t={
  'todosget'() {}
};
//等价于
var t={
  'todosget':function todosget() {}
};
//方式3
var todosget=function(){};
var t={todosget};
//等价于
var t={
  'todosget':function todosget() {}
};
//其他1
const { type1:at1, type2:at2 }={at1:1,at2:1};
```