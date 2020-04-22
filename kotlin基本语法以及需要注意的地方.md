#kotlin基本语法以及需要注意的地方

##常见符号
1: ?可空类型
2: ?.安全调用符: 如果a非空，就返回a.length，否则返回null，这个表达式的类型是 Int?
3: ？:Elvis 操作符  a?.length ?: -1
   当a不为空时返回正常的值，但当a为空时返回null，返回null不能满足我们开发要求，所以可以使用?:操作符返回-1
4: !!操作符:希望直接抛出NPE异常可以用!!操作符
5: ==判断值是否相等，===判断值及引用是否完全相等
6: ..区间以及 in 和 !in 操作符, 结合until使用
7: downTo() 函数, 倒序迭代, 如 5 downTo 1
8: step()函数, 指定步数, 如i in 1..5 step 2 => 1 3 5
9: _(下划线) 在结构对应函数的时候,直传部分的变量, 其余的可用_代替
10:得到类的Class对象 ::
11:限定符号: @
   限定this的类型,给出区域,标识是哪个区域的变量
   跳出for循环,回到标记点, 在for循环之前loop@,break@loop回到标记处
   命名函数自定义标签:匿名函数可以通过自定义标签进行跳转和返回
12: $操作符, 模板表达式,可用连接字符串


##kotlin中的普通的内联函数的使用

###回调函数的Kotin的lambda的简化
可以使用一个lambda函数来代替。可以简化写一些不必要的嵌套回调方法。但是需要注意:在lambda表达式，只支持单抽象方法模型，也就是说设计的接口里面只有一个抽象的方法，才符合lambda表达式的规则，多个回调方法不支持。

###let函数
1:let扩展函数的实际上是一个作用域函数，当你需要去定义一个变量在一个特定的作用域范围内，let函数的是一个不错的选择；let函数另一个作用就是可以避免写一些判断null的操作。

2: 使用场景:
场景一: 最常用的场景就是使用let函数处理需要针对一个可null的对象统一做判空处理。
场景二: 然后就是需要去明确一个变量所处特定的作用域范围内可以使用

###with函数
1:格式
```.kt
with(object){
   //todo
 }
```
2:with函数的适用的场景
适用于调用同一个类的多个方法时，可以省去类名重复，直接调用类的方法即可，经常用于Android中RecyclerView中onBinderViewHolder中，数据model的属性映射到UI上
```
 with(item){
   
      holder.tvNewsTitle.text = StringUtils.trimToEmpty(titleEn)
	   holder.tvNewsSummary.text = StringUtils.trimToEmpty(summary)
	   holder.tvExtraInf.text = "难度：$gradeInfo | 单词数：$length | 读后感: $numReviews"
       ...   
   
   }
```
###run函数
1:run函数使用的一般结构
```
object.run{
//todo
}
```
2:run函数实际上可以说是let和with两个函数的结合体，run函数只接收一个lambda函数为参数，以闭包形式返回，返回值为最后一行的值或者指定的return的表达式。

3:场景:
适用于let,with函数任何场景。因为run函数是let,with两个函数结合体，准确来说它弥补了let函数在函数体内必须使用it参数替代对象，在run函数中可以像with函数一样可以省略，直接访问实例的公有属性和方法，另一方面它弥补了with函数传入对象判空问题，在run函数中可以像let函数一样做判空处理

###apply函数
```
object.apply{
//todo
}
```
整体作用功能和run函数很像，唯一不同点就是它返回的值是对象本身，而run函数是一个闭包形式返回，返回的是最后一行的值。正是基于这一点差异它的适用场景稍微与run函数有点不一样。apply一般用于一个对象实例初始化的时候，需要对对象中的属性进行赋值。或者动态inflate出一个XML的View的时候需要给View绑定数据也会用到，这种情景非常常见。特别是在我们开发中会有一些数据model向View model转化实例化的过程中需要用到。

###also函数
1: 结构:和上面类似
2: 适用于let函数的任何场景，also函数和let很像，只是唯一的不同点就是let函数最后的返回值是最后一行的返回值而also函数的返回值是返回当前的这个对象。一般可用于多个扩展函数链式调用

###对比
let:  适用于处理不为null的操作场景
```
//it指代当前对象
fun <T, R> T.let(block: (T) -> R): R = block(this)
```

with: 适用于调用同一个类的多个方法时，可以省去类名重复，直接调用类的方法即可，经常用于Android中RecyclerView中onBinderViewHolder中，数据model的属性映射到UI上 
```
//this指代当前对象或者省略
fun <T, R> with(receiver: T, block: T.() -> R): R = receiver.block()
```

run:适用于let,with函数任何场景。
```
//this指代当前对象或者省略
fun <T, R> T.run(block: T.() -> R): R = block()
```

apply:
1、适用于run函数的任何场景，一般用于初始化一个对象实例的时候，操作对象属性，并最终返回这个对象。
2、动态inflate出一个XML的View的时候需要给View绑定数据也会用到.
3、一般可用于多个扩展函数链式调用
4、数据model多层级包裹判空处理的问题
```
//this指代当前对象或者省略
fun T.apply(block: T.() -> Unit): T { block(); return this }
```

also: 适用于let函数的任何场景，一般可用于多个扩展函数链式调用
```
//it指代当前对象
fun T.also(block: (T) -> Unit): T { block(this); return this }
```


##Kotlin常见高阶函数
###forEach
```
//类似于java的迭代器
list.forEach{
        val newElement = it * 2 + 3
        newList.add(newElement)
    }
```

###Filter: 筛选高阶函数, 挑选符合条件的应用
将中的元素遍历,把符合要求的元素添加到新的list中,并将新list返
```
arrayOf(1, 2, 3, 0, 4).filter { i -> i >= 2 }.forEach(::println)
```
输出: 2 3 4

###map:转化成集合
将List中每个元素转换成新的元素,并添加到一个新的List中,最后将新List返回
```
arrayOf(1, 2, 3).map { i: Int -> i * 10 }.forEach(::println)
```

###flatMap:将数组中全部元素按顺序组成一个list
```
listOf(listOf("a", "b"), listOf("c", "d")).flatMap { i: List<String> -> i.asIterable() }.forEach(::println)
```

###fold:将集合中的元素依次冒泡组合,最终得到一个结果
```
val foldResult1 = arrayOf(1, 2, 3).fold(10, { a, b -> a + b })//计算过程为10+1+2+3,等于16
val foldResult2 = arrayOf(1, 2, 3).fold(10, { a, b -> a * b })//计算过程为10*1*2*3,等于60
```

###reduce:与fold类似,区别是reduce没有初始值
```
val reduceResult1 = arrayOf(1, 2, 3, 4).reduce { acc, i -> acc + i }//计算过程为1+2+3+4,等于10
val reduceResult2 = arrayOf(1, 2, 3, 4).reduce { acc, i -> acc * i }//计算过程为1*2*3*4,等于24
```

###joinToString:为集合元素添加分隔符,组成一个新的字符串并返回
```
val joinToStringResult1 = arrayOf("a", "b", "c", "d").joinToString { i -> i }
val joinToStringResult2 = arrayOf("a", "b", "c", "d").joinToString(separator = "#", prefix = "[前缀]", postfix = "[后缀]", limit = 3, truncated = "[省略号]") { i -> i }
```

###takeWhile
遍历list中的元素,将符合要求的元素添加到新集合中
注意:一旦遇到不符合要求的,直接终止
```
//注意:返回的集合中只有4和3,没有5,因为遇到2时,不符合要求,程序直接终止
arrayOf(4, 3, 2, 5).takeWhile { i -> i > 2 }.forEach(::println)
```
