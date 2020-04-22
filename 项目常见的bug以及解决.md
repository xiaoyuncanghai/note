#项目常见的bug以及解决:

## activity回退栈的管理以及特殊的启动模式
1: SingTop
2: SingTask
3: SingInstance
其中SingleInstance是单独管理了一个栈, 如果说设置了这个启动模式, A->B->C->D(SingeInstance),回退会导致一个问题,D->B->A->C,因为ABD单独保存在一个栈里面,而C处于另一个栈里面单独维护,回退的时候会出现问题, 解决方法:设置Android全局FLAG,即sharepreference,保存一个变量值,如果回退的时候判断这个值,which activity拥有这个索引就去哪里,然后重置这个FLAG
4: intent.addFlags(Intent.FLAG_ACTIVITY_NO_HISTORY)
此FLAG很坑爹, 设置了这个FLAG, 只要是从这个启动的intent到的activity, 只要处于不可见的情况就被destory掉了,会导致置于后台后目标activity恢复不了,误认为是crash
5: Intent.FLAG_ACTIVITY_CLEAR_TASK: 清空activity所有的task,如果其只是一个loading页码,后面的某一个task需要做home页码, 点击back键需要退回主页面,此时就需要前面的loading页面全部清除掉task

## Fragment回退栈和back键的处理
Navigation为Fragment添加了自动的返回栈管理，非常便于处理多个Fragment的相互跳转。但Fragment默认没有onBackPressed()方法，在按下返回键时无法处理除Fragment出栈以外的其他操作,Fragment有嵌套或者是嵌套在ViewPager中,处理起来比较麻烦
###onBackPressedDispatcher添加回调
```
requireActivity().onBackPressedDispatcher
	.addCallback(viewLifecycleOwner, object: OnBackPressedCallback(true) {
		override fun handleOnBackPressed() {
        //Handle back event from any fragment 
        }
    })
```
最大弊病是不能在里面调用Activity的onBackPressed()（会产生循环），当Fragment不需要处理返回操作时不能向上传递到Activity，必须在Fragment里处理包括Activity在内的所有返回键操作

###给rootView设置一个OnKeyListener来监听key事件
要在Fragment里获取到RootView，二是如果Activity和Fragment都需要对返回键做出反应，那么必须是优先触发Fragment的返回操作，而Activity不知道返回操作什么时候会被Fragment拦截！

###给Fragment编写各自的onBackPressed()方法并在Activity的onBackPressed()里调用
在BaseFragment里获取到当前Fragment的实例以及子Fragment重写BaseFragment的onBackpressed()方法，使得Activity能自动调用当前Fragment的onBackpressed()方法
实现了Activity与Fragment返回逻辑代码的完全分离，Activity不需要知道Fragment具体干了什么，只需要知道结果（是否消耗掉事件），并且能通过调整condition的顺序来调整Activity与Fragment返回操作的优先级
```
private var mBackHandledInterface: BackHandledInterface? = null
    /**
     * 所有继承BackHandledFragment的子类都将在这个方法中实现物理Back键按下后的逻辑
     * FragmentActivity捕捉到物理返回键点击事件后会首先询问Fragment是否消费该事件
     * 如果没有Fragment消息时FragmentActivity自己才会消费该事件
     */
    abstract fun onBackPressed(): Boolean

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        if (activity !is BackHandledInterface) {
            throw ClassCastException("Hosting Activity must implement BackHandledInterface")
        } else {
            mBackHandledInterface = activity as BackHandledInterface?
        }
    }

    override fun onStart() {
        super.onStart()
        mBackHandledInterface!!.setSelectedFragment(this)
    }
```

```
interface BackHandledInterface {
    fun setSelectedFragment(selectedFragment: BackHandledFragment?)

    fun getSelectedFragment(): BackHandledFragment
}
```

```
class DashboardActivity : BackHandledInterface{
	override fun setSelectedFragment(selectedFragment: BackHandledFragment?) {
        this.mBackHandedFragment = selectedFragment;
    }

    override fun getSelectedFragment(): BackHandledFragment {
        return mBackHandedFragment!!
    }

	override fun onBackPressed() {
        if (drawer_layout.isDrawerOpen(GravityCompat.START)) {
            drawer_layout.closeDrawer(GravityCompat.START)
        } else {
            if (mBackHandedFragment == null || !mBackHandedFragment!!.onBackPressed()) {
                if (supportFragmentManager.backStackEntryCount == 0) {
                    //fgt 消费事件全部指向完毕,然后执行activity的backPress事件
                } else {
                    supportFragmentManager.popBackStack()
                }
            }
        }
    }
} 
```
```
class DashboardFragment : BackHandledFragment() {
	override fun onBackPressed(): Boolean {
        return if (hadIntercept) {
            false
        } else {
            //消费事件,然后popUp fragment
            true
        }
    }
}
```

##RxJava针对retrofit网络请求更换域名
1: 普通模式
2: 高级模式
3: 超级模式
RetrofitUrlManager:
```
RetrofitUrlManager.getInstance().with(builder) 设置拦截器
RetrofitUrlManager.getInstance().setGlobalDomain(Url) 全局设置域名
RetrofitUrlManager.getInstance().startAdvancedModel(url) 设置高级模式
```