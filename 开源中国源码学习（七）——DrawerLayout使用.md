# 开源中国源码学习（七）——DrawerLayout使用

DrawerLayout是Google官方推荐的一种实现侧滑菜单的方式。

开源中国oschina客户端和git@osc的侧滑菜单都是借助DrawerLayout来实现的。在git@osc项目的学习时，只是草草过了一下，既然又碰到了，有必要详细的总结一下。

学习DrawerLayout的最好教材无非就是Google的官方文档—— [Creating a Navigation Drawer](https://developer.android.com/training/implementing-navigation/nav-drawer.html)

下文的DrawerLayout的使用是根据Google的官方文档翻译而来。

##简介

导航抽屉是隐藏在屏幕左边缘的一部分界面它包含了APP的主要导航选项。大多数时候它是隐藏的，但是当用户手指轻轻从屏幕左边缘往右面滑动时这个导航界面将会展现，或者是，在APP的最顶部，当用户点击了在Action Bar中的app icon它也会呈现。

**导航抽屉的设计**

在你决定在应用中使用导航抽屉之前，你必须得明白[Navigation Drawer](https://www.google.com/design/spec/patterns/navigation-drawer.html#)的使用情景和设计原则。

##创建一个Drawer Layout

为了添加导航抽屉，你需要在你的布局界面中声明一个`DrawerLayout`对象作为布局的根节点。同时在`DrawerLayout`内部添加两个view:

> 1. 添加一个View，它包含应用的主内容（当抽屉隐藏时你的主要布局）；
> 2. 添加另一个View它包含了导航抽屉；

如下面例子所示：该布局使用了DrawerLayout它包含了两个子节点：一个`FrameLayout`它包含了主要内容（在运行时将会被Fragment替换） 和 一个`ListView`作为导航抽屉

```
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <!-- The main content view -->
    <FrameLayout
        android:id="@+id/content_frame"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
    <!-- The navigation drawer -->
    <ListView android:id="@+id/left_drawer"
        android:layout_width="240dp"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:choiceMode="singleChoice"
        android:divider="@android:color/transparent"
        android:dividerHeight="0dp"
        android:background="#111"/>
</android.support.v4.widget.DrawerLayout>
``` 

上面这个例子包含了一些重要的布局技巧：

* 主内容View（FrameLayout在最上层）必须是`Drawerlayout`的第一个子节点因为XML在安排这些界面的时候是按照Z轴的顺序来安排的 同时 抽屉必须在主内容的顶部。
* 主内容View被设置成匹配父View的宽和高，因为当导航抽屉隐藏的时候它要填充整个UI。
* 导航View（ListView）必须被声明一个水平的gravity借助属性`android:layout_gravity`。为了满足从右到左的约定，声明它的值为"start" 代替 "left"(因此这个抽屉将会在右面呈现当布局是RTL时)
* 在导航View声明时：宽度用dp为单位、高度匹配父View。为了保证用户无论怎样都能看到主内容的一部分，导航抽屉的宽度不能超过320dp


##初始化Drawer List

在你的Activity中，要做的第一件事是初始化导航抽屉的列表项。具体该怎么做根据你APP的内容来定，但是导航抽屉通常包含一个Listview，所以还需要一个相匹配的Adapter（比如 ArrayAdapter 或者 SimpleCursorAdapter）

下面的例子，告诉你该如何借助一个string array 来初始化一个导航list

```
public class MainActivity extends Activity {
    private String[] mPlanetTitles;
    private DrawerLayout mDrawerLayout;
    private ListView mDrawerList;
    ...

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mPlanetTitles = getResources().getStringArray(R.array.planets_array);
        mDrawerLayout = (DrawerLayout) findViewById(R.id.drawer_layout);
        mDrawerList = (ListView) findViewById(R.id.left_drawer);

        // Set the adapter for the list view
        mDrawerList.setAdapter(new ArrayAdapter<String>(this,
                R.layout.drawer_list_item, mPlanetTitles));
        // Set the list's click listener
        mDrawerList.setOnItemClickListener(new DrawerItemClickListener());

        ...
    }
}
```
上面的代码调用了`setOnItemClickListener()`来接受在导航list中的点击事件。下面的内容展示如何实现这个接口来改变主界面当用户选择了一个条目后。


##处理导航点击事件

当用户选择了Drawer list中的某个条目后，系统将会调用OnItemClickListener对象的onItemClick()方法，前提是要给ListView设置setOnItemClickListener()。

在onItemClick()方法中做什么根据你想如何实现你的APP结构来定。在下面的例子中，每当选择了一个item后将会在主界面中插入一个不同的Fragment（FrameLayout元素根据R.id.content_frame来定）

```
private class DrawerItemClickListener implements ListView.OnItemClickListener {
    @Override
    public void onItemClick(AdapterView parent, View view, int position, long id) {
        selectItem(position);
    }
}

/** Swaps fragments in the main content view */
private void selectItem(int position) {
    // Create a new fragment and specify the planet to show based on position
    Fragment fragment = new PlanetFragment();
    Bundle args = new Bundle();
    args.putInt(PlanetFragment.ARG_PLANET_NUMBER, position);
    fragment.setArguments(args);

    // Insert the fragment by replacing any existing fragment
    FragmentManager fragmentManager = getFragmentManager();
    fragmentManager.beginTransaction()
                   .replace(R.id.content_frame, fragment)
                   .commit();

    // Highlight the selected item, update the title, and close the drawer
    mDrawerList.setItemChecked(position, true);
    setTitle(mPlanetTitles[position]);
    mDrawerLayout.closeDrawer(mDrawerList);
}

@Override
public void setTitle(CharSequence title) {
    mTitle = title;
    getActionBar().setTitle(mTitle);
}
```

##监听打开和关闭事件

为了监听抽屉的打开和关闭事件，在你的Drawerlayout中调用setDrawerListener 同时给它传递一个DrawerLayout.DrawerListener的具体实现。这个接口提供了抽屉事件的回调方法，比如：onDrawerOpened()和onDrawerClosed().

然而，通常情况下不是实现DrawerLayout.DrawerListener,如果你的Activity包含了Action Bar，你可以继承ActionBarDrawerToggle类。ActionBarDrawerToggle实现了DrawerLayout.DrawerListener，所以你依然可以复写上述回调方法，但是它真的加快了Action Bar icon 和 导航抽屉之间的交互（将会在下一部分详细讨论）

在Navigation Drawer的设计指南中讨论的，当Drawer可见的时候，你应该修改Action Bar的内容，比如：改变标题或者移除根据主内容界面的上下文决定的Action items. 

下面的代码向你展示如何借助ActionBarDrawerToggle实例来覆写DrawerLayout.DrawerListener的回调方法：

```
public class MainActivity extends Activity {
    private DrawerLayout mDrawerLayout;
    private ActionBarDrawerToggle mDrawerToggle;
    private CharSequence mDrawerTitle;
    private CharSequence mTitle;
    ...

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ...

        mTitle = mDrawerTitle = getTitle();
        mDrawerLayout = (DrawerLayout) findViewById(R.id.drawer_layout);
        mDrawerToggle = new ActionBarDrawerToggle(this, mDrawerLayout,
                R.drawable.ic_drawer, R.string.drawer_open, R.string.drawer_close) {

            /** Called when a drawer has settled in a completely closed state. */
            public void onDrawerClosed(View view) {
                super.onDrawerClosed(view);
                getActionBar().setTitle(mTitle);
                invalidateOptionsMenu(); // creates call to onPrepareOptionsMenu()
            }

            /** Called when a drawer has settled in a completely open state. */
            public void onDrawerOpened(View drawerView) {
                super.onDrawerOpened(drawerView);
                getActionBar().setTitle(mDrawerTitle);
                invalidateOptionsMenu(); // creates call to onPrepareOptionsMenu()
            }
        };

        // Set the drawer toggle as the DrawerListener
        mDrawerLayout.setDrawerListener(mDrawerToggle);
    }

    /* Called whenever we call invalidateOptionsMenu() */
    @Override
    public boolean onPrepareOptionsMenu(Menu menu) {
        // If the nav drawer is open, hide action items related to the content view
        boolean drawerOpen = mDrawerLayout.isDrawerOpen(mDrawerList);
        menu.findItem(R.id.action_websearch).setVisible(!drawerOpen);
        return super.onPrepareOptionsMenu(menu);
    }
}
```

下面的章节描述了ActionBarDrawerToggle的构造参数和其它需要设置的步骤，用来完成和Action Bar icon的交互。


##用App Icon 完成打开和关闭Drawer

用户可以打开或者关闭抽屉在屏幕的左边缘进行滑动，但是如果你使用了Action Bar，你应该允许用户通过触碰APP icon来打开或者关闭抽屉。APP icon 也应该用一个特殊的icon对navigation Drawer 进行暗示。你可以实现这些所有的表现通过使用在上一节展示的ActionBarDrawerToggle 。

为了确保ActionBarDrawerToggle有效工作，用它的构造方法创建一个实例，它需要如下参数：

* 持有Drawer的Activity
* DrawerLayout
* 作为Drawer 指示器的图片资源 
* 描述抽屉打开的字符串资源
* 描述抽屉关闭的字符串资源

接着，不管你是否已经创建了ActionBarDrawerToggle的子类作为你Drawer 的监听器，你需要在一些地方调用你的ActionBarDrawerToggle贯穿你的Activity的整个生命周期。

```
public class MainActivity extends Activity {
    private DrawerLayout mDrawerLayout;
    private ActionBarDrawerToggle mDrawerToggle;
    ...

    public void onCreate(Bundle savedInstanceState) {
        ...

        mDrawerLayout = (DrawerLayout) findViewById(R.id.drawer_layout);
        mDrawerToggle = new ActionBarDrawerToggle(
                this,                  /* host Activity */
                mDrawerLayout,         /* DrawerLayout object */
                R.drawable.ic_drawer,  /* nav drawer icon to replace 'Up' caret */
                R.string.drawer_open,  /* "open drawer" description */
                R.string.drawer_close  /* "close drawer" description */
                ) {

            /** Called when a drawer has settled in a completely closed state. */
            public void onDrawerClosed(View view) {
                super.onDrawerClosed(view);
                getActionBar().setTitle(mTitle);
            }

            /** Called when a drawer has settled in a completely open state. */
            public void onDrawerOpened(View drawerView) {
                super.onDrawerOpened(drawerView);
                getActionBar().setTitle(mDrawerTitle);
            }
        };

        // Set the drawer toggle as the DrawerListener
        mDrawerLayout.setDrawerListener(mDrawerToggle);

        getActionBar().setDisplayHomeAsUpEnabled(true);
        getActionBar().setHomeButtonEnabled(true);
    }

    @Override
    protected void onPostCreate(Bundle savedInstanceState) {
        super.onPostCreate(savedInstanceState);
        // Sync the toggle state after onRestoreInstanceState has occurred.
        mDrawerToggle.syncState();
    }

    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
        mDrawerToggle.onConfigurationChanged(newConfig);
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Pass the event to ActionBarDrawerToggle, if it returns
        // true, then it has handled the app icon touch event
        if (mDrawerToggle.onOptionsItemSelected(item)) {
          return true;
        }
        // Handle your other action bar items...

        return super.onOptionsItemSelected(item);
    }

    ...
}
```

至此，官方文档翻译完毕！

***

##总结

开源中国中使用的导航抽屉就是基于官方提供的DrawerLayout来做的，按照上面的文档一步步来，应该就会使用DrawerLayout了。

同时，我将这部分抽取出来做了一个单独的项目,可以移步我的github下载源码：

 [App_Frame](https://github.com/SOFTPOWER1991/App_Frame)



