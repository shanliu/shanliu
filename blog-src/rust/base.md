# 一些概念



```
泛型 
	不可继承
	可限定:trait
	可泛型实现trait
		泛型实现trait的泛型可带trait限定

生命周期 
  存在生命周期生命跟泛型声明时,生命周期参数放前面
	可继承[子长过父]
	函数带引用参数
		输入必须大于输出
	结构带引用
	'static
		局部生命周期可转为'static 如 Box::leak
	方法参数中&self的生命周期
		不能跟自身相同 否则只能调用一次

方法
	静态方法
		无类型实现无法调用
	对象方法[参数 self||&self||mut self||&mut self ] mut self不能再trait的fn中
		无指定类型实现可调用[可根据self推导]
	Self 为类型自身
	可带泛型
	可带生命周期
		引用参数标识生命周期

trait
	可以继承
	无属性
	可带方法
	使用需实现到类型或泛型
	可带泛型
	可带生命周期
	dyn trait [注意是类型不是trait,所以可以 impl traitname for dyn name]
		可将实现类型转为dyn trait 对象[可非安全转回]
		可给dyn trait 增加方法
	返回分发 Box<dyn triat> 和 impl trait
	操作符也是trait
	内部泛型[内部type]
	
	
struct||enum [实体]
	不可继承
	可带属性
	可带方法
	实现trait
		不可多次和远程实现
		可实现多个
		重名方法调用指定trait
		可判断是否实现某类型
	可带泛型
		内嵌泛型实现两[实体]联合 xx<xx> [一级继承]
	可带生命周期
		字段引用标识生命周期
		泛型+生命周期且字段未使用 PhantomData
	Drop trait 实现
```
