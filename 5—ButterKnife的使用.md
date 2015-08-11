# 开源中国源码学习（六）——ButterKnife的使用

本文翻译自 `Butter Knife`官方网站： [ButterKnife](http://jakewharton.github.io/butterknife/)

## 简介

用@Bind给字段进行注释并且Butter Knife会根据给定的View ID去查找并自动转换为你与你的layout中相匹配的View。

##Activity Binding

Activity绑定示例代码如下：

```
class ExampleActivity extends Activity {
  @Bind(R.id.title) TextView title;
  @Bind(R.id.subtitle) TextView subtitle;
  @Bind(R.id.footer) TextView footer;

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.simple_activity);
    ButterKnife.bind(this);
    // TODO Use fields...
  }
}
```
代替缓慢的反射，代码通常表现的跟View看起来的样子。调用bind委托这些代码你可以看到和debug.

上面的例子生成的代码大概跟下面的一样：

```
public void bind(ExampleActivity activity) {
  activity.subtitle = (android.widget.TextView) activity.findViewById(2130968578);
  activity.footer = (android.widget.TextView) activity.findViewById(2130968579);
  activity.title = (android.widget.TextView) activity.findViewById(2130968577);
}
```

##NON-ACTIVITY BINDING 非Activity绑定

你也可以绑定在自定义对象上来支持你自己的view root。

```
public class FancyFragment extends Fragment {
  @Bind(R.id.button1) Button button1;
  @Bind(R.id.button2) Button button2;

  @Override public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fancy_fragment, container, false);
    ButterKnife.bind(this, view);
    // TODO Use fields...
    return view;
  }
}
```
##VIEW HOLDER BINDING 对View Holder 进行绑定

```
public class MyAdapter extends BaseAdapter {
  @Override public View getView(int position, View view, ViewGroup parent) {
    ViewHolder holder;
    if (view != null) {
      holder = (ViewHolder) view.getTag();
    } else {
      view = inflater.inflate(R.layout.whatever, parent, false);
      holder = new ViewHolder(view);
      view.setTag(holder);
    }

    holder.name.setText("John Doe");
    // etc...

    return view;
  }

  static class ViewHolder {
    @Bind(R.id.title) TextView name;
    @Bind(R.id.job_title) TextView jobTitle;

    public ViewHolder(View view) {
      ButterKnife.bind(this, view);
    }
  }
}
```

你可以看到这些实践在提供的例子中。

调用ButterKnife.bind可以在任何地方使用否则调用findViewById.

其它的已经提供的绑定API：

绑定主观对象，使用Activity作为view根节点。如果你用的模式像MVC是可以用它的Activity绑定控制器，借助ButterKnife.bing(this , activity).

绑定一个view的孩子到字段使用ButterKnife.bind(this).如果你用了<merge>节点在你的布局里或者inflate了自定义view的构造方法你可以立刻调用此方法。或者，从XML反射的自定义view类型可以使用它在onFinishInflate回调方法中。

##VIEW LISTS
你可以组织复杂的views到一个list或者array中。

```
@Bind({ R.id.first_name, R.id.middle_name, R.id.last_name })
List<EditText> nameViews;
```

`apply`方法允许你立即采取行动对在list中的所有view.

```
ButterKnife.apply(nameViews, DISABLE);
ButterKnife.apply(nameViews, ENABLED, false);
```

`Action`和`Setter`接口允许你具体声明简单的表现

```
static final ButterKnife.Action<View> DISABLE = new ButterKnife.Action<View>() {
  @Override public void apply(View view, int index) {
    view.setEnabled(false);
  }
};
static final ButterKnife.Setter<View, Boolean> ENABLED = new ButterKnife.Setter<View, Boolean>() {
  @Override public void set(View view, Boolean value, int index) {
    view.setEnabled(value);
  }
};
```

Android的`Property`（配置信息）也可以被用在`apply`方法中。

```
ButterKnife.apply(nameViews, View.ALPHA, 0.0f);
```



##LISTENER BINDING

监听器也可以自动配置进方法中：

```
@OnClick(R.id.submit)
public void submit(View view) {
  // TODO submit data to server...
}
```

监听方法的所有参数可以被操作：

```
@OnClick(R.id.submit)
public void submit() {
  // TODO submit data to server...
}
```
定义一个具体的类型并且它将会自动的被类型转换

```
@OnClick(R.id.submit)
public void sayHi(Button button) {
  button.setText("Hello!");
}
```

为通用事件操作在一个单独的绑定中声明多个ID

```
@OnClick({ R.id.door1, R.id.door2, R.id.door3 })
public void pickDoor(DoorView door) {
  if (door.hasPrizeBehind()) {
    Toast.makeText(this, "You win!", LENGTH_SHORT).show();
  } else {
    Toast.makeText(this, "Try again", LENGTH_SHORT).show();
  }
}
```

自定义view可以绑定在他们自己的监听器上并且不需要具体声明一个ID。

```
public class FancyButton extends Button {
  @OnClick
  public void onClick() {
    // TODO do something!
  }
}
```

##BINDING RESET
Fragment和Activity相比有不同的生命周期。当在一个Fragment的onCreateView中绑定的时候，在onDestoryView中设置Views为null。Butter Knife 有一个 unbind 方法来自动的做这件事。

```
public class FancyFragment extends Fragment {
  @Bind(R.id.button1) Button button1;
  @Bind(R.id.button2) Button button2;

  @Override public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fancy_fragment, container, false);
    ButterKnife.bind(this, view);
    // TODO Use fields...
    return view;
  }

  @Override public void onDestroyView() {
    super.onDestroyView();
    ButterKnife.unbind(this);
  }
}

```
##OPTIONAL BINDINGS 
默认情况下，@Bind 和 监听器的绑定都是必须的。如果目标view没有被找到的话一个异常将会被抛出。

为了镇压这个表现和创建一个可选择的绑定，为字段或者是方法添加一个@Nullable注解

注意：任何名字为@Nullable的注解可以被这样使用。鼓励你使用Android自己的注解库"support-annotations"中的@Nullable注解，参见[Android Tools Project](http://tools.android.com/tech-docs/support-annotations).

```
@Nullable @Bind(R.id.might_not_be_there) TextView mightNotBeThere;

@Nullable @OnClick(R.id.maybe_missing) void onMaybeMissingClicked() {
  // TODO ...
}
```
##MULTI-METHOD LISTENERS

与方法注解相匹配的监听器有多个回调可以被用来绑定在他们中间的任何一个身上。每一个注解都有默认的回调跟它绑定在一起。可以使用`callback`参数声明一个可替代的回调。

```
@OnItemSelected(R.id.list_view)
void onItemSelected(int position) {
  // TODO ...
}

@OnItemSelected(value = R.id.maybe_missing, callback = NOTHING_SELECTED)
void onNothingSelected() {
  // TODO ...
}
```
##BONUS

Butter Knife同时还包含了findById方法，这个精简的方法依然可以找到目标view在一个View、Activity、或者 Dialog中。
它会通常会推断出返回值的类型然后自动进行类型转换。

```
View view = LayoutInflater.from(context).inflate(R.layout.thing, null);
TextView firstName = ButterKnife.findById(view, R.id.first_name);
TextView lastName = ButterKnife.findById(view, R.id.last_name);
ImageView photo = ButterKnife.findById(view, R.id.photo);
Add a static import for ButterKnife.findById and enjoy even more fun.
```
##下载

[Butter Knife v7.0.1 JAR 下载](http://repo1.maven.org/maven2/com/jakewharton/butterknife/7.0.1/butterknife-7.0.1.jar)
[Butter Knife Github 地址](https://github.com/JakeWharton/butterknife)
[Butter Knife Java Docs 地址](http://jakewharton.github.io/butterknife/javadoc/index.htmls)

##MAVEN
如果你使用Maven进行编译，你可以声明这个library作为你的依赖

```
<dependency>
  <groupId>com.jakewharton</groupId>
  <artifactId>butterknife</artifactId>
  <version>7.0.1</version>
</dependency>
```

##GRADLE

compile 'com.jakewharton:butterknife:7.0.1'

确定压制住这个lint 警告在你的build.gradle文件中。

```
lintOptions {
  disable 'InvalidPackage'
}
```

一些配置可能也需要单独的处理

```
packagingOptions {
  exclude 'META-INF/services/javax.annotation.processing.Processor'
}
```
##IDE CONFIGURATION
一些IDE需要额外的配置为了是注解进程能够使用

* IntelliJ IDEA —— 如果你的工程使用了额外的配置（比如 Maven 的 pom.xml）那么annotation 进程需要仅仅工作。如果没有，试试 [manual configuration](http://jakewharton.github.io/butterknife/ide-idea.html)
* Eclipse —— 设置 [manual configuration](http://jakewharton.github.io/butterknife/ide-eclipse.html)

##PROGUARD

Butter Knife 动态的生成和使用class，这意味着静态的分析工具像ProGuard可能会认为这是毋庸的。为了阻止他们被移除，明确的标明他们将要被保持。为了阻止ProGuard 重命名class因此使用@Bind在一个成员变量，因此keepclasseswithmembernames 选项被使用了。

混淆时代码如下

```
-keep class butterknife.** { *; }
-dontwarn butterknife.internal.**
-keep class **$$ViewBinder { *; }

-keepclasseswithmembernames class * {
    @butterknife.* <fields>;
}

-keepclasseswithmembernames class * {
    @butterknife.* <methods>;
}
```

## 总结


在项目中碰到的问题，如下：
	
在开源中国源码中，Butter Knife 使用的是：ButterKnife.inject(this)
	
	
```
    @InjectView(R.id.quick_option_iv)
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        ButterKnife.inject(this);
    }
```
	
而在读官网doc时，发现该方法并不存在，却是: ButterKnife.bind(this)
	
	
```
	 @Bind(R.id.tv1) TextView tv1;
    @Bind(R.id.tv2) TextView tv2;
    @Bind(R.id.tv3) TextView tv3;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ButterKnife.bind(this);

        tv1.setText("tv1");
        tv2.setText("tv2");
        tv3.setText("tv3");
    }

```
并且在代码中进行试验 inject也不可以使用，后来找到了原因:
	
> 1. 在开源中国中使用的是——butterknife-6.1.0.jar
> 2. 官网文档配套的是——butterknife-7.0.1.jar
	
找到了答案，这应该是版本问题


