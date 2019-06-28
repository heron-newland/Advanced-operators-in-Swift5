# Advanced-operators-in-Swift5

Swift相比OC在运算符上做了一些改变, 但是基本的运算符,比如 `+,-,*,/,%`, 逻辑运算符,比如 `&&,||,!`, 还有三目运算符等也基本没有变. 只是取消了自增`++`, 自减`--`.还有就是针对swift的可选类型的特点, 推出了` ?? `运算符.

下面介绍一下Swift中的高级云算符, [demo请点击](https://github.com/heron-newland/Advanced-operators-in-Swift5)

### 溢出操作符 &+, &-, &*
溢出操作符在我们进行运算时,如果结果超出了容器范围,会自行做容错处理.例如:

	var a :UInt8 = UInt8.max
	var b = a
	a &+ 1 //a为UInt8容器的最大值, 溢出后再从最小值开始, 所以结果为0
	b += 1//编译直接报错,因为会溢出
	
### 操作符方法
很多时候我们需要给自定义的数据结构或者类来定义一个操作符方法.比如向量的加减法

我们先定义如下一个结构体, 后文中的所有自定义操作符都是根据这个结构体来实现的.

	struct Vector2D{
	    var x = 0.0, y = 0.0
	}
	
自定义向量加法运算

	extension Vector2D{
	    static func + (left:Vector2D, right:Vector2D) -> Vector2D {
	        return Vector2D(x: left.x + right.x, y: left.y + right.y)
	    }
	}
	
	let vec1 = Vector2D(x: 1, y: 2)
	let vec2 = Vector2D(x: 3, y: 4)
	//调用方式
	vec1 + vec2
	
自定义向量的反向运算(只有一个操作数), 那么我们可以根据情况定义前置或者后置操作符(Prefix and postfix operators), 案例如下

	//前置操作符
	extension Vector2D {
	    static prefix func - (vector:Vector2D) -> Vector2D {
	        return Vector2D(x: -vector.x, y: -vector.y)
	    }
	}
	//调用方式
	-vec1

>以上是前置操作符(prefix)的定义和使用方式, 后置操作符(postfix)同理

### 联合赋值操作符
联合赋值操作符常见的就是 `+=, -= `等.比如我们做向量的的` += `运算.

	//赋值操作符+=
	extension Vector2D {
	    static func += (left: inout Vector2D, right: Vector2D) {
	    //此处的` + `操作符是上面定义的
	        left = left + right
	    }
	}
	//调用方式
	vec1 += vec2
	
### 等同性操作符
等同性操作符最常见的就是` ==, >=, <= `等, 通常系统很多类都能直接使用这些等同性操作符, 那是因为其遵循了 `Equatable` 协议, 所以是实现这个功能有两种方式, 第一: 让自定义的类或者结构体遵循`Equatable` 协议; 第二: 自定义操作符.我们今天讨论的是自定义操作符, 实现方法如下:

	//等同性操作符
	extension Vector2D {
	    static func == (left: Vector2D, right: Vector2D) -> Bool {
	        //此处调用的 == 是Float类型遵循 Equatable 协议中的函数, 不是此处自定义的
	        return left.x == right.x && left.y == right.y
	    }
	}
	//调用方式
	vec2 == vec1


### 自定义操作符
以上这些操作符都是系统实现过的, 而且这些操作符的关键字都是全局的, 我们虽然能自定义其行为, 但是也不是我们完全自定义的操作符.例如我们要定义一个` ++ `(在Swift新版本中已经移除了++, 和--, 要使用必须由开发者自定义)来实现自增运算.我们可以使用`prefix, infix, postfix`几个修饰符来限定操作数有几个, 操作数在左边还是在右边.

自定义自增运算案例如下:

	//告诉系统+++是一个后置操作符
	postfix operator +++
	extension Vector2D{
	    static postfix func +++ (vector: inout Vector2D) -> Vector2D {
	        return Vector2D(x: vector.x + 1, y: vector.y + 1)
	    }
	}
	//调用方式
	vec2+++

### 操作符的优先级
运算过程中, 优先级非常重要, 同样的算式, 优先级不一样,得到的结果会完全不同. 那么我们自定义的操作符如何规定其优先级呢?

	//和乘法同等优先级
	infix operator **: MultiplicationPrecedence
	extension Vector2D {
	    static func ** (left: Vector2D, right: Vector2D) -> Vector2D {
	        return Vector2D(x: left.x * right.x, y: left.y * right.y)
	    }
	}
	
	vec1 + vec2 ** vec2

注意: 只有双目运算符才能定义优先级.
