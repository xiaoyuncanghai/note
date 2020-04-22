#MVVM的使用总结


## 项目设计模式
解决目的:
Activity本身需要担负与用户之间的操作交互，再加上现在大部分的Activity还对整个App起到控制器的作用，这又带入了大量的逻辑代码，造成Activity的臃肿

## MVP设计模式
1: MVP:基于事件驱动应用框架
- View负责显示
- Presenter负责逻辑处理
- Model提供数据

在MVP框架中，View与Model并不直接交互，所有的交互放在Presenter中.其中使用最广泛的是Passive View模式，即被动视图。在这种模式下，整个框架内部模块之间的逻辑操作均由Presenter控制，View仅仅是整个操作的汇报者和结果接收者，Model根据Presenter的单向调用返回数据(图片来自网络)。并且MVP模式使得View与Model的耦合性更低，降低了Presenter对View的依赖，实现了关注点分离的初衷，方便开发人员的编码和测试工作。
```
关系图解: P层和V层互持引用,P层给V层数据,V层必须实现接口,然后P层调用V层对象的接口方法返回数据(接口回调)
view <=> present -> model
```
2: Android的MVP抽取归纳
- M: Model-模型层:数据data类,网络请求得到的json数据等,model接口提供给Present层直接调用
- P: Present-逻辑层:处理UI的操作, 决定用户的响应, 并且设计对应数据的填充等
- V: View-UI显示层: Activity, fragment, Adapter等和UI有直接相关的类(显示和操作UI)

UI层的Activity在启动之后实例化相应的Presenter，App的控制权后移，
由UI转移到Presenter，两者之间的通信通过BroadCast、Handler或者接口完成，只传递事件和结果。
```
example: UI层通知逻辑层（Presenter）用户点击了一个Button，
逻辑层（Presenter）自己决定应该用什么行为进行响应，该找哪个模型（Model）去做这件事，
最后逻辑层（Presenter）将完成的结果更新到UI层。
```

3:缺点
Android对整个框架的控制能力还是不够强

##MVVM Android设计结构
###databinding的基本使用
1: 说明:
针对每个Activity或者Fragment的布局，编译阶段，生成一个ViewDataBinding类的对象，该对象持有Activity要展示的数据和布局中的各个view的引用。同时还有：将数据分解到各个view、在UI线程上更新数据、监控数据的变化，实时更新.展示的数据和展示的布局绑定。

2:使用
```
//1:build.grade配置databinding
android {
    dataBinding {
	    enabled = true
 	}
}
```

```
//2:创建Javabean对象
class User(){
	String name;
	int age;
}
```

```
//3:使用databinding之后Activity的布局
<Layout ...>
	<data>
		<variable
			name = "user"
			type = "package-User()"
	</data>
	<LinearLayout ...>
	</LinearLayout>
</Layout>
```
根节点变成了layout，里面包括了data节点和传统的布局。data节点作用是连接 View 和 Modle 的桥梁。在这个data节点中声明一个variable变量，值就可以传到布局文件中来了。而且TextView中没有给控件定义id，在text的时候用了@{ }的方法，可以直接引用User对象的属性完成赋值。

```
//MainActivity.java
//获取ActivityMainBinding
ActivityMainBinding binding = DataBindingUtil.setContentView(this,R.layout.activity_main);
UserBean userBean = new UserBean ("张三", "25");
//使用binding和user进行绑定
binding.setUser(userBean );
```

3: 原理:
DataBinding相关的task也是系统预先帮我们定义好的，默认情况下，DataBinding相关的task在task列表中是没有的，一旦我们通过 dataBinding{enabled = true}的方式开启DataBinding之后，DataBinding相关的task就会出现在task列表中，每当我们执行编译之类的操作时，就会执行这些dataBinding Task, 这些task的作用就是检查并生成相关dataBinding代码，

###MVVM结构
1: 层级:
- Model层: 数据结构层,设计网络请求数据以及我们定义的dataclass数据类
- ViewModel层: 中间逻辑接口层:负责提供view层所需要的接口,执行数据转换的逻辑
- View层: 只负责显示UI,不执行数据处理逻辑

```
持有的单向引用
Activity/Fgt(View层) --> ViewModel --> model
```
```
如何实现解耦:观察者模式
1: Model层:观测数据变化,并将数据传递出去给ViewModel,实现了model到ViewModel的单向传递
2: ViewModel层:ViewModel层和model层进行交互, 持有view层的引用,调用model层的接口方法, 然后viewmodel被view观察(View层成了消费者)
3: View 层: UI层的主要工作是观察,观察ViewModel的数据变化,然后数据变化了去获取数据跟新UI
```

2:LiveData:LiveData是一个可以被观察的数据持有者
特点:
```
1: 新引入的组件
2: 可被观察数据持有者
3: 遵循应用程序组件的生命周期,对组件生命周期管理,避免内存泄漏
```
具体使用:
LiveData<List<Date>> liveDataObserver = MutableLiveData<List<Data>>
将数据放在liveData中, 接受观察者的观察,并且遵循启生命周期,在观察者组件销毁的时候也会将数据注销

3:项目展示:
https://github.com/xiaoyuncanghai/MvvmDemo
结构详解:
```
Model层相关:
	Databean-package
		ImageJsonBean: 数据类 图片Url+图片标题Title
		UrlStatusBean: 数据类 Json获取成功以及获取失败
    http-package
		httpUtils: 网络请求积累, 初始化retrofit, 并且提供返回ImageJsonBean的接口
					GetImageUrlCallback: 获取ImageJson的callback接口,目的是为了让m层和vm层解除耦合,通过callback vm可以拿到M层的数据
		IRequestImage: retrofit请求网络接口

ViewModel层:
	viewModel-package:
		ImageViewModel: 初始化的时候,callback拿数据,并且数据放在liveData中.
						处理中转逻辑，将真正获取数据的操作交给Model去执行，然后将获取到的数据更改到自身的LiveData中去。 LiveData中的数据改变后，会去通知监听它改变的、当前状态是可见的View们去更改UI。
	adapter-package:
		GetBingImageAdapter: 加载图片,databing 不需要手动设置adapter

View 层:
	MainActivity: 1:建立ViewModel，并将Activity的生命周期绑定到ViewModel上
				  2:为ViewModel的UrlData建立数据监听，并监听数据变化，根据数据更新UI
				  3:监听到数据变化后，通过databinding更改布局UI
	activity_main: databinding绑定数据
       	 	<variable name="image"
					type="com.example.mvvmdemo.kotlinbean.ImageJsonBean.Image" />
       		 <variable
       		     name="clicker"
       		     type="com.example.mvvmdemo.MainActivity.Clicker" />
       		 <variable
       		     name="uiChange"
       		     type="Integer" />
		xml中声明数据映射关系, 并且声明点击事件
```

4: 关于项目开发:增加了Repository层
```
View层: 只显示UI,不执行数据处理逻辑
|
ViewModel层: 负责提供view层需要的接口,执行数据转化逻辑
|
Repository层: 处理从localStorage, webservice, database中得到的数据,返回给ViewModel
|
Model层:数据结构(DataClass)
```

