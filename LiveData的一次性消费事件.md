#LiveData的一次性消费事件

##问题描述:
Fragment里面的WebView的状态在fragment置于后台后, webView的状态不保存, 恢复到初始状态

##分析过程
###1: WebView当前状态没有保存:  
当fragment销毁的时候,再重新创建的时候,webView的状态丢失.
WebView的状态保存和恢复不像其他原生View一样是自动完成的.
WebView不是继承自View的.如果我们把WebView放在布局里, 不加处理,   
那么Activity或Fragment重建的过程中, WebView的状态就会丢失, 变成初始状态.  
**处理办法:**  
1:在fragment的onSaveInstance()里面可以加入代码保存webView的状态  
```  
@Override
public void onSaveInstanceState(Bundle outState) {  
    super.onSaveInstanceState(outState);
    webView.saveState(outState);
}
```  
2:在初始化的时候,增加判断, 不必每次都打开初始链接  
```     
if (savedInstanceState != null) {
    webView.restoreState(savedInstanceState);
} else {
    webView.loadUrl(TEST_URL);
}    
```
再重新建立的时候, 无论WebView是放在Activity中的还是放在Fragment中的,WebView都可以恢复到离开时候的界面  

相关链接:https://www.cnblogs.com/mengdd/p/5582244.html

###2: Fragment没有被销毁, 可是WebView为什么还是会发生变化?
**问题分析**  
onCreateView() -> onViewCreate() -> onResume()  
这个是Fragment的基础的生命周期, 在onCreateView里面会加载视图, 在置于后台的时候, Fragment不会销毁.  
所以在生命周期里面只会走一次.  那么既然只走了一次, onCreateView里面loadUrl加载资源的方法为什么会被触发呢?  
来看代码:  
 ![](https://github.com/xiaoyuncanghai/note/tree/master/image/live一次性消费事件.png)  


``` 
lifecycleScope.launch(Dispatchers.Main) { 
               dashboardViewModel.dashboardMediator.observe(lifecycleOwner,
                    Observer<AvonResponses<DashboardPageResponse>> {
                        when (it) {
                            is AvonResponseSuccess -> dashboardPageResponse = it.data
                            is AvonResponseError -> dashboardContentError = it.exception
                        }
```  

```
                        if (dashboardPageResponse != null && !hadObserver) {
							...
                            if (webViewType == LEADS_TYPE && apptId.isNotEmpty()) {
                                webViewURL = getNewURLLeads(LEADS_TYPE, dashboardPageResponse!!)
                                loadUrlWithToken(
                                    getNewURLLeads(
                                        LEADS_TYPE,
                                        dashboardPageResponse!!
                                    ), getAccessToken(),
                                    dashboardViewModel.getUserId(),
                                    dashboardViewModel.getLoginStatus(),
                                    dashboardViewModel.getUserCategory(),
                                    callback,
                                    webViewViewModel.getLocalStorage().getMarketCode(),
                                    webViewViewModel.getLocalStorage().getLatestCommonErrorInfo()
                                )
                            } }})}  
```   
**很明显,是因为将dashboardInfo设置成为了观察对象, 当fragment从后台恢复的时候, 数据会重新处理,导致了dashboardInfo再次被消费, 执行了loadUrl**

**解决办法:**  
LiveData+Observer特性就是每次绑定的时候就会触发一次,如果没有处理好, 就会有莫名的UI显示,   
就和本例一样,导致WebView重新加载并且显示  

为了解决此问题, 可以通过消费掉LiveData值之后置空LiveData,但是不利于V和VM的分离,在界面上不应该操作liveData的值,Google建议封装一个一次性的LiveData数据,对MutableLiveData的值进行封装, 创建一个MutableLiveData子类OneOffLiveData的方式解决此问题  
1: 需要消耗一次值(标志位), 封装类OneTimeEvent，通过标志位来判断是否调用过  
2: 将一次性事件封装LiveData子类中, 使用内联函数操作
核心思想, 就观察事件一次性消费掉,然后再将标志位更改

相关链接:https://www.haomeiwen.com/subject/bblpmhtx.html

