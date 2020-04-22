#FragmentStateAdapter的使用
在ViewPager中使用Fragment的情况下，可以给ViewPager设置两种Adapter，一种是FragmentStatePagerAdapter，一种是FragmentPagerAdapter。

###FragmentStatePageAdapter
1:会销毁不需要的Fragment,ViewHolder会保存正在显示的Fragment和它左右两边第一个Fragment, 切换的时候会销毁不用的并且离正在显示的fgt比较远的fgt,销毁的fgr会通过onSaveInstanceState方法来保存Fragment中的Bundle信息，当再次切换回来的时候，就可以利用保存的信息来恢复到原来的状态

###FragmentPageAdapter
FragmentPageAdapter会调用事务的detach方法来处理，而不是使用remove方法。因此，FragmentPageAdapter只是销毁了Fragment的视图，其实例还是保存在FragmentManager中
代码如下:实现动态添加fragment,其中FragmentLeaderStateAdapter里面除了可以传fragment还可以传fragmentActivity, 根据是在activity中还是在fragment中来添加fgt来确定
```
package uk.co.avon.mra.common.base.adapter

import androidx.fragment.app.Fragment
import androidx.fragment.app.FragmentActivity
import androidx.viewpager2.adapter.FragmentStateAdapter

class FragmentLeaderStateAdapter(
    private val fragmentList: MutableList<Fragment>,
    fragment: Fragment
) : FragmentStateAdapter(fragment) {
    override fun getItemCount(): Int {
        return fragmentList.size
    }

    override fun createFragment(position: Int): Fragment {
        return fragmentList[position]
    }
}
```
```
val adapter = FragmentLeaderStateAdapter(fragmentList, this)
leads_viewPager.adapter = adapter
```

###对比总结
FragmentStatePageAdapter适用于Fragment较多的情况,如果这些Fragment都保存在FragmentManager中的话，对应用的性能会造成很大的影响。
FragmentPageAdapter则适用于固定的，少量的Fragment情况，例如和TabLayout共同使用时,就需要使用这个adapter