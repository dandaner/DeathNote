#Android Support Annotations
[TOC]

##Android Support Annotations简介
Android support library从19.1版本开始，给开发者引入了注解库，并且在后续的22.2版本中又新增了13个新的注解，来帮助我们发现代码中的一些潜在的问题。

注解可以用来修饰代码，以@开头，他可以修饰参数，函数返回值，方法以及类。

##引入Annotations
对于使用Android Studio开发的同学来说，引入注解库非常简单，我们只需要在build.gralde中引入下面这段代码即可，需要注意的是如果你的项目中引入了appcompat library，那么我们就不需要声明，因为appcompat library本身就依赖注解库，但是appcompat lib的版本如果较低的话，注解库的版本也是较低的，新增的注解就不支持，**所以友情提示：最好使用最新版本的注解库，本文中，部分注解22.2.0之后才支持。使用下面这句话就一切搞定！！！**
```
dependencies {
    compile 'com.android.support:support-annotations:22.2.0'
}
```
但是对于一个纯java module,我们需要单独设置，也很简单
```
repositories {
   jcenter()
   maven { url '<your-SDK-path>/extras/android/m2repository' }
}
```
##执行Annotation
1、最简单的方法就是使用最新版本的Android Studio,IDE会自动进行annotation的检测并实时提示错误信息，**Studio版本最少需要在1.5.1及以上，否则无法实时提示，只能通过执行lint,在lint生成的报告中才能看见问题**。
2、当你使用gradle版本高于1.3的时候，手动执行lint,在生成的lint报告中，你能看见Annotation的相关警告或者错误。

##常见的注解类型：
---
###Nullness Annotations
空类型的注解，一般用来修饰参数，函数返回值，不能用来修饰基本数据类型
- @Nullable 可以为空
- @NonNull 不允许为空
- 未指定类型
```java
public static void nullnessAnnot(@NonNull String name, int age) {
        LogHelper.d(name + String.valueOf(age));
    }
```
如上代码：
1.当我们这样调用的时候nullnessAnnot（null, 0）,Android Studio 将会抛出一下警告：**passing 'null' argument to parameter annotated as @NotNull。**
2.如果我们在参数age前添加@NonNull，也会出现警告：**primitive type member cannot be annotated**，所以Nullness类型注解是不能用来修饰基本数据类型的。

需要注意的是：**@Nullable和@NonNull并不是对立的,还有第三种情况就是未指定类型，如果不添加注解，lint也就不会做检测**。其实在说未指定类型之前，是有一个小问题需要抛出来，就是既然注解这么好，那么我们是不是应该在所有的函数，参数上都添加@Nullable和@NonNull注解呢？答案显然是否定的，我们只要看一下系统的源码就能找出答案，并不是所有的接口都是用注解标示。官方文档上给出了一小段解释：
> 最初，我们在findViewById方法上标注@Nullable，从技术上说，这是正确的：findViewById可以返回null。但是如果你知道你在做什么的时候(如果你传递给他一个存在的id)他是不会返回null的。当我们使用@Nullable注解它的时候，就意味着源码编辑器会有大量的代码（我们没有对findViewById进行非空判断的代码）出现高亮警告。只有当你需要每次使用该方法都应该明确的进行null检查，那么才能用@Nullable标注返回值。根据经验来说：看现有的“好的代码”(比如审查产品代码)，看看这些API是怎么被使用的。如果该代码为null检查结果，你应该为方法注解@Nullable。

为了更好的展示上面这段表述的含义，我写了如下代码片段
```java
// 1.在AnnotationUtils类定义一个API接口
@Nullable
public static View findViewById(@NonNull Activity ctx, int resId {
        return ctx.findViewById(resId);
    }

// 2.使用该API
{
	...
	
	View v = AnnotationUtils.findViewByID(this, R.id.action_bar);
	if (v != null) {
        ViewGroup.LayoutParams layoutParams = v.getLayoutParams();
        layoutParams.height = 0;
    }
    
    ...
}
```
上述代码2中，如果将变量v的非空判断去掉，Studio将会给出以下警告 **Method invocation 'v.getLayoutParams()' may produce 'java.lang.NullPointerException'** 如此看来，注解也不是顺便乱用的。

###Resource Type Annotations
资源类型的注解，主要用来修饰函数参数，而且都是整形的，道理很简单，Android将所有的资源都映射成一个整型常量。举个例子，非常简单：
```java
// 声明一个@StringRes类型的资源类型
public void setText(TextView textView, @StringRes int resId) {
        textView.setText(resId);
    }
// 如下代码调用，如果强制传递一个其他资源类型的参数，就不是警告那么简单，编译器将会直接红色报错显示Expected resource of type string
setText(new TextView(mContext), R.drawable.abc_ab_share_pack_holo_dark);
```
一般情况下，一个foo类型的资源都拥有一个FooRes类型的注解，比如：@StringRes, @DrawableRes, @ColorRes, @InterpolatorRes等等。
此外，还有一个注解类型@AnyRes，主要用于你不知道当前使用的是哪种资源类型，eg:Resources#getResourceName(@AnyRes int resId)

对于资源类型的注解是不是也都写上呢？看看源码中，也并不是所有的都使用了这种注解。个人见解，资源类型的注解，大部分我们在自定义View中会使用到API定义，这里，如果需要传递一个资源类型的参数，最好加上这中注解，能够起到很好的错误检测作用

###IntDef/StringDef: Typedef Annotations
类型定义注解，这类注解主要用来替换枚举类型的，在android性能优化典范中，官方不推荐我们在代码中使用枚举类型，因为他会产生class文件以及类加载，影响性能，所以，一般推荐我们使用整型或者String来定义我们需要的类型，那么这两种注解类型就应运而生，他们的使用几乎是一样的。直接上官网的例子，非常清晰：
```java
import android.support.annotation.IntDef;
...
public abstract class ActionBar {
    ...
    @IntDef({NAVIGATION_MODE_STANDARD, NAVIGATION_MODE_LIST, NAVIGATION_MODE_TABS})
    @Retention(RetentionPolicy.SOURCE)
    public @interface NavigationMode {}

    public static final int NAVIGATION_MODE_STANDARD = 0;
    public static final int NAVIGATION_MODE_LIST = 1;
    public static final int NAVIGATION_MODE_TABS = 2;

    @NavigationMode
    public abstract int getNavigationMode();

    public abstract void setNavigationMode(@NavigationMode int mode);
```
当我们在API接口中，要求SDK使用者传递的参数只能是我们定义的参数时，我们可以采用这种方式来约束开发人员，来保证接口质量。
> 注意：上述代码中注解NavigationMode上使用了@Retention(RetentionPolicy.SOURCE)注解，他代表的含义是使用该注解的注解的保留策略，有3种保留策略。可以参见RetentionPolicy类，简单来说
> 1、SOURCE 注解只在源码中保留，不会生成class文件
> 2、CLASS注解会生成class文件，但是不会在运行时，这是默认策略
> 3、RUNTIME会生成class文件，在运行时也是可用的。
> 大部分情况下，我们使用SOURCE模式即可，简单实用，不会生成多余代码，其他两种方式我没有深入研究。

另外，上面这种方式限定了参数只能使用定义的三种模式，如果想参数混合使用，也就是我们常见的使用 | ， & 连接常量使用的话，可以通过下面这种方式,就是多了一个flag,非常简单。
```java
@IntDef(flag=true, value={
            DISPLAY_USE_LOGO,
            DISPLAY_SHOW_HOME,
            DISPLAY_HOME_AS_UP,
            DISPLAY_SHOW_TITLE,
            DISPLAY_SHOW_CUSTOM
    })
@Retention(RetentionPolicy.SOURCE)
public @interface DisplayOptions {}
```

###Threading Annotations
- @UiThread
- @MainThread
- @WorkerThread
- @BinderThread

线程类型的注解，用来修饰方法和类，如果你的方法需要运行在指定类型的线程中，你可以使用上面四种类型。如果你的一个类中，所有的方法都需要运行在同样的线程中，那么就可以在类上使用此类型注解。
下面仍然是典型的使用案例：
```java
@WorkerThread
protected abstract Result doInBackground(Params... params);

@MainThread
protected void onProgressUpdate(Progress... values);
```
如果你强行在一个标注了@WorkerThread的方法中，调用了一个标注了@MainThread或者@UiThread的方法，lint将会提示错误：**Method XXX must be called from the UI thread, currently inferred thread is worker...**。

还有一个比较有意思的问题是，在执行lint的时候，显示的在线程调用@MainThread或者@UiThread方法，IDE并不会有任何警告提示，如下：
```java
// 只有在一个明确标记@WorkerThread的方法中调用一个明确标记@MainThread的方法，才会有错误提示，比如上述的onProgressUpdate（）在doInBackground（）中调用，lint才会提出警告。下面这种写法IDE没有任何提示。
new Thread(new Runnable() {
            @Override
            public void run() {
                onProgressUpdate();
            }
        }).start();
```

线程类型的注解非常有效，我们以前的写法都是使用注释的方式，提醒API 的使用者，某个方法需要运行在UI线程，或者工作线程，但是他并不具备约束力。很容易产生运行时错误。

> **官方文档中关于@MainThread和@UiThread区别的解释：**
> 
> 在一个进程中有且仅有一个主线程@MainThread。同时它也是一个UI线程@UiThread。举个例子，我们的Activity就运行在这个线程之中。然而，对于一个进程来说，它有能力去创建另外一个线程来运行不同的窗口，但是这种情况是及其少见的，一般都是系统进程拥有这种能力。一般情况下，和生命周期相关的方法，我们使用@MainThread，和View相关的，我们使用@UiThread。因为一个@MainThread就是一个@UiThread，大多数情况下，@UiThread又是一个@MainThread,所以他们是可以互换的。
> 
> 对于上面这段话理解，一句话总结，大部分情况下，两者是可以互换的。使用原则就是和View相关的我们使用@UiThread。其他我们基本都可以使用@MainThread
###RGB Color Integers
- @ColorInt
RGB类型的注解，专门用来修饰RGB类型的色值整型。在上面的资源类型的注解中，我们提到过，如果你想在API接口参数中，约束用户必须传入一个color类型的资源ID，那么你可以使用@ColorRes注解。但是，在另外一种情况下，你的API接口不想使用color类型的的资源ID，而是一个实实在在的RGB或者ARGB类型的整型值，这个时候，你就可以使用@ColorInt:
```java
public void setTextColor(@ColorInt int color)
```
如果你在上述代码中，传递了一个R.color.black,那么lint就会报错**should pass resolved color instead of resource id ...**

个人认为，不论是@ColorRes还是@ColorInt, 都是非常必要的，我们写代码过程中，经常没有写清楚API说明，导致使用过程中会发现，明明设置了颜色，却没有生效，往往就是因为传递了一个资源ID而并没有解析，然后找半天，没找到问题所在。使用这种注解，能够快速帮我们定位发现问题。
###Value Constraints
- @Size
- @IntRange
- @FloatRange

值约束类注解，主要用于定义参数范围的注解，举个栗子：**当你的参数是一个float或者double类型**，而你又期望参数的传值只能在[0.0，1.0]之间取值的时候，那么你就可以使用@FloatRange,同样道理适用于@IntRange。（**当参数是int或者long型参数**)
```java
public void setAlpha(@FloatRange(from=0.0, to=1.0) float alpha)
public void setAlpha(@IntRange(from=0, to=255) int alpha)

//完整用法fromInclusive和toInclusive代表是否包含边界值，默认为true
@FloatRange(from = 0.0, to = 1.0, fromInclusive = false, toInclusive = false)
```

对于数组，集合以及String,可以通过@Size来约束集合的size.下面是一些使用场景
- 集合不能为空@Size(min=1)
- String必须最多23个字符 @Size(max=23)
- 数组必须有且仅有2个元素@Size(2)
- 数组的size必须是2的倍数，比如一个位置坐标x/y @Size(multiple=2)

###Permissions
权限类型类型注解，直接上代码
```java
// 1、接口要求指定权限
@RequiresPermission(Manifest.permission.SET_WALLPAPER)
public abstract void setWallpaper(Bitmap bitmap) throws IOException;

// 2、接口要求多选一
@RequiresPermission(anyOf = {
    Manifest.permission.ACCESS_COARSE_LOCATION,
    Manifest.permission.ACCESS_FINE_LOCATION})
public abstract Location getLastKnownLocation(String provider);

// 3、接口同时要求多个权限
@RequiresPermission(allOf = {
    Manifest.permission.READ_HISTORY_BOOKMARKS, 
    Manifest.permission.WRITE_HISTORY_BOOKMARKS})
public static final void updateVisitedHistory(ContentResolver cr, String url, boolean real) {

// 4、Content Provider权限
@RequiresPermission.Read(@RequiresPermission(READ_HISTORY_BOOKMARKS))
@RequiresPermission.Write(@RequiresPermission(WRITE_HISTORY_BOOKMARKS))
public static final Uri BOOKMARKS_URI = Uri.parse("content://browser/bookmarks");
```

###Overriding Methods: @CallSuper
对于一个允许重写的方法，如果你要求重写的方法必须调用父类的方法，那么可以如下使用
```java
@CallSuper
protected void onCreate(@Nullable Bundle savedInstanceState) 
```
如果子类在重写你的方法的时候，没有调用父类方法，则会提出警告。
(Studio 1.3 preview1 存在检测bug，即使你调用了父类方法，仍然提示错误，preview2 已经修复了)
###Return Values: @CheckResult

###@VisibleForTesting

###@Keep

---
##参考资料
---
- http://tools.android.com/tech-docs/support-annotations
- http://www.flysnow.org/2015/08/13/android-tech-docs-support-annotations.html



