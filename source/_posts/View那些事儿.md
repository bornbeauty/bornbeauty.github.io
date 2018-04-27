---
title: View那些事儿
date: 2017-09-11 20:58:16
tags: 
    - Android
---

[TOC]

#### 前言

`View`是我们开发中再熟悉不过的一个组件了，几乎任何需求的开发实现都离不开它。那么今天今天就好好梳理一下View的基础内容，常用自定义方法，事件分发等知识。

> 看了不少Android的源码，总结出一点心得： **所谓的代码简洁不是代码量少，而是逻辑简单明了。** 常常看到源码中会有不少重复判断的地方，或者说可以合并起来的判断语句，但是Google工程师都是分开处理的，这就不得不让我疑惑了。随着每一次的思考，今天突然发现，原来我一直在享受这样的写的便利，就是每一段代码判断都只做一件事，逻辑清晰明了~

#### 从LayoutInflater讲起

为什么讲View要先讲`LayoutInflater`呢？因为View的加载都是通过`LayoutInflater`的`inflate(...)`方法进行的。可是我们在Activity中使用`setContentView(int layoutID)`也调用这个方法了吗？答案是肯定的。

`setContentView()`的源码是这样的：

```
@Override
public void setContentView(int resId) {
    ensureSubDecor();
    ViewGroup contentParent = (ViewGroup) mSubDecor.findViewById(android.R.id.content);
    contentParent.removeAllViews();
    LayoutInflater.from(mContext).inflate(resId, contentParent);
    mOriginalWindowCallback.onContentChanged();
}
```

> 注意：如果你继承的是AppCompatActivity的话会追踪到一个`AppCompatDelegate`类，这个不用管 直接看他的实现类的对应方法就行了

LayoutInflater.inflate()方法的作用也很清晰，就是将XML文件中的一个布局加载转化成一个View类对象。究竟这个过程是如何完成的，我们一步一步的看一下。

##### LayoutInflater的基本用法

首先看一下它的基本用法：

获得LayoutInflater实例有两种方法，
第一种：

```
LayoutInflater inflater = LayoutInflater.from(context);
```

第二种：

```
LayoutInflater inflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
```

其实第一种是第二种方法的一种简单封装，内部还是通过getSystemService去获取的。另外，LayoutInflater是一个单例。

使用LayoutInflater加载布局一般用这两个方法：

```
public View inflate(@LayoutRes int resource, @Nullable ViewGroup root)

public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot)
```

##### LayoutInflater的源码分析

第一个参数是需要加载的布局的ID，第二个参数是加载该布局时候的外部布局，而第三个参数是指是否要将该布局添加到外部的布局上面。当然了，这个外部布局可以使空的。这里我们还是看一下具体的代码。不管你是使用的哪个inflate()方法的重载，最终都会辗转调用到LayoutInflater的如下代码中：

```
public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
    synchronized (mConstructorArgs) {
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, "inflate");

        final Context inflaterContext = mContext;
        final AttributeSet attrs = Xml.asAttributeSet(parser);
        Context lastContext = (Context) mConstructorArgs[0];
        mConstructorArgs[0] = inflaterContext;
        View result = root;

        try {
            // Look for the root node.
            int type;
            while ((type = parser.next()) != XmlPullParser.START_TAG &&
                    type != XmlPullParser.END_DOCUMENT) {
                // Empty
            }

            if (type != XmlPullParser.START_TAG) {
                throw new InflateException(parser.getPositionDescription()
                        + ": No start tag found!");
            }

            final String name = parser.getName();

            if (DEBUG) {
                System.out.println("**************************");
                System.out.println("Creating root view: "
                        + name);
                System.out.println("**************************");
            }

            if (TAG_MERGE.equals(name)) {
                if (root == null || !attachToRoot) {
                    throw new InflateException("<merge /> can be used only with a valid "
                            + "ViewGroup root and attachToRoot=true");
                }

                rInflate(parser, root, inflaterContext, attrs, false);
            } else {
                // Temp is the root view that was found in the xml
                final View temp = createViewFromTag(root, name, inflaterContext, attrs);

                ViewGroup.LayoutParams params = null;

                if (root != null) {
                    if (DEBUG) {
                        System.out.println("Creating params from root: " +
                                root);
                    }
                    // Create layout params that match root, if supplied
                    params = root.generateLayoutParams(attrs);
                    if (!attachToRoot) {
                        // Set the layout params for temp if we are not
                        // attaching. (If we are, we use addView, below)
                        temp.setLayoutParams(params);
                    }
                }

                if (DEBUG) {
                    System.out.println("-----> start inflating children");
                }

                // Inflate all children under temp against its context.
                rInflateChildren(parser, temp, attrs, true);

                if (DEBUG) {
                    System.out.println("-----> done inflating children");
                }

                // We are supposed to attach all the views we found (int temp)
                // to root. Do that now.
                if (root != null && attachToRoot) {
                    root.addView(temp, params);
                }

                // Decide whether to return the root that was passed in or the
                // top view found in xml.
                if (root == null || !attachToRoot) {
                    result = temp;
                }
            }

        } catch (XmlPullParserException e) {
            final InflateException ie = new InflateException(e.getMessage(), e);
            ie.setStackTrace(EMPTY_STACK_TRACE);
            throw ie;
        } catch (Exception e) {
            final InflateException ie = new InflateException(parser.getPositionDescription()
                    + ": " + e.getMessage(), e);
            ie.setStackTrace(EMPTY_STACK_TRACE);
            throw ie;
        } finally {
            // Don't retain static reference on context.
            mConstructorArgs[0] = lastContext;
            mConstructorArgs[1] = null;

            Trace.traceEnd(Trace.TRACE_TAG_VIEW);
        }

        return result;
    }
}
```

从这这里我们可以看出，LayoutInflater其实就是使用Android提供的pull解析方式来解析文件的。其中我们发现调用了`createViewFromTag()`这个方法，接收一个rootView参数和一个属性参数，通过名字可以看出，这个方法是用来创建View对象的。跟踪这个方法，里面又调用了createView()方法，然后反射创建View实例并且返回。

创建完成这个根布局后，又调用`rInflate()`方法来循环遍历这个跟布局下面的所有子元素：

```
private void rInflate(XmlPullParser parser, View parent, final AttributeSet attrs)
        throws XmlPullParserException, IOException {
    final int depth = parser.getDepth();
    int type;
    while (((type = parser.next()) != XmlPullParser.END_TAG ||
            parser.getDepth() > depth) && type != XmlPullParser.END_DOCUMENT) {
        if (type != XmlPullParser.START_TAG) {
            continue;
        }
        final String name = parser.getName();
        if (TAG_REQUEST_FOCUS.equals(name)) {
            parseRequestFocus(parser, parent);
        } else if (TAG_INCLUDE.equals(name)) {
            if (parser.getDepth() == 0) {
                throw new InflateException("<include /> cannot be the root element");
            }
            parseInclude(parser, parent, attrs);
        } else if (TAG_MERGE.equals(name)) {
            throw new InflateException("<merge /> must be the root element");
        } else {
            final View view = createViewFromTag(name, attrs);
            final ViewGroup viewGroup = (ViewGroup) parent;
            final ViewGroup.LayoutParams params = viewGroup.generateLayoutParams(attrs);
            rInflate(parser, view, attrs);
            viewGroup.addView(view, params);
        }
    }
    parent.onFinishInflate();
}
```

我们发现其实也是调用了`createViewFromTag()`方法来创建View实例，然后又递归调用`rInflate()`方法查找View的子元素，每次递归玩成后将这个View添加到父布局中。这样，把xml中的所有View都解析完成，然后将跟布局返回，这样inflate()方法就到此结束了，而我们也得到了View对象实例。


```
inflate(int resource, ViewGroup root, boolean attachToRoot)  
```

我们可以通过代码解释一下这个root参数和attachroot参数的意思和用处。

在加载View的时候，如果root存在并且attachToRoot为false的话，View在调用`createViewFromTag()`创建的时候会把root的ViewGroup.LayoutParams当参数传递进去。那么有什么用吗？当然有用。我们知道，在View的onMeasure()方法中需要根据父布局去确定当前View的大小，除非我们指定某一确定的值为View的大小，否则在inflater一个View的时候传递空的root进去，可能得到的布局大小并不是你所想要的。这个时候你就要考虑一下这个因素的了。那么attachToRoot什么意思呢？我们看到，在它为true的时候直接调用了root.addView()将temp view添加了进去，也就是说我们加载的View会被自动添加绘制到root下面，说道这里，这俩参数什么意思应该很明确了。

#### View的绘制流程

在创建得到一个View之后，那么显示在屏幕上的时候，显示大小，显示位置是怎么确定的呢？这就涉及到了View的绘制过程了。接下来就看一下View的绘制流程。

View要想显示在屏幕上面，要经过测算绘制才能显示在屏幕上面。每一个View的绘制过程都必须经过三个过程：测算，布局和绘制，即`onMeasure()`,`onLayout()`,`onDraw()`。

##### onMeasure()

通过名字我们就可以知道这个方法是用来测算View大小的。

```
public final void measure(int widthMeasureSpec, int heightMeasureSpec)
```

View的measure()方法接收两个整形参数，这两个值分贝用于确定视图的宽度和高度的规则和大小。但是他们并不是直接表示宽度和高度的，而是带有不同规格的。什么意思呢？

简单讲是这样的，我们知道Java中的int是32位的，`widthMeasureSpec`和`heightMeasureSpec`中的两位代表了测试模式，也就是`specMode`，剩下的才是测量的大小，即`specSize`。也就是说，`MeasureSpecMode`和`MeasureSpecSize`组成。

而specMode有三种：

- EXACTLY：精确模式，父布局提供一个精确的尺寸给这个View。无论View想要多大的尺寸，父布局给的布局边界已经确定。
- AT_MOST：该View可以获得的最大的尺寸
- UNSPECIFIED：父布局对该View的大小没有任何局限，View想要多大都可以。

那么，`widthMeasureSpec`和`heightMeasureSpec`这两个值是从哪里来的呢？

我们发现，在View中还有一个`measure()`方法：

```
public final void measure(int widthMeasureSpec, int heightMeasureSpec) {  
    if ((mPrivateFlags & FORCE_LAYOUT) == FORCE_LAYOUT ||  
            widthMeasureSpec != mOldWidthMeasureSpec ||  
            heightMeasureSpec != mOldHeightMeasureSpec) {  
        mPrivateFlags &= ~MEASURED_DIMENSION_SET;  
        if (ViewDebug.TRACE_HIERARCHY) {  
            ViewDebug.trace(this, ViewDebug.HierarchyTraceType.ON_MEASURE);  
        }  
        onMeasure(widthMeasureSpec, heightMeasureSpec);  
        if ((mPrivateFlags & MEASURED_DIMENSION_SET) != MEASURED_DIMENSION_SET) {  
            throw new IllegalStateException("onMeasure() did not set the"  
                    + " measured dimension by calling"  
                    + " setMeasuredDimension()");  
        }  
        mPrivateFlags |= LAYOUT_REQUIRED;  
    }  
    mOldWidthMeasureSpec = widthMeasureSpec;  
    mOldHeightMeasureSpec = heightMeasureSpec;  
}
```

可以看到，`measure()`方法调用了`onMeasure()`方法去计算View大小。这个方法是一个final类型的，所以子类是无法继承重写这个方法的，说明Google并不希望开发者改变measure的流程，我们只能去改变这个计算的数值。而`onMeasure()`的源码也很简单：

```
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
```

`getDefaultSize()`:

```
public static int getDefaultSize(int size, int measureSpec) {
    int result = size;
    int specMode = MeasureSpec.getMode(measureSpec);
    int specSize = MeasureSpec.getSize(measureSpec);

    switch (specMode) {
    case MeasureSpec.UNSPECIFIED:
        result = size;
        break;
    case MeasureSpec.AT_MOST:
    case MeasureSpec.EXACTLY:
        result = specSize;
        break;
    }
    return result;
}
```

`getSuggestedMinimumWidth()`:

```
protected int getSuggestedMinimumHeight() {
    return (mBackground == null) ? mMinHeight : max(mMinHeight, mBackground.getMinimumHeight());
}
```

而`setMeasuredDimension()`方法最终会调用`setMeasuredDimensionRaw()`方法：

```
private void setMeasuredDimensionRaw(int measuredWidth, int measuredHeight) {
    mMeasuredWidth = measuredWidth;
    mMeasuredHeight = measuredHeight;

    mPrivateFlags |= PFLAG_MEASURED_DIMENSION_SET;
}
```

最终我们在`onMeasure()`方法中传入的specWidth和specHeight会赋值给mMeasureWIdth和mMeasureHeight。这样View的大小就确定了。

那么，measure方法里面的specWidth和specHeight是从哪里来得呢？measure方法又是怎么被调用的呢？这就涉及到ViewGroup了，我们知道，任何View都是要添加到ViewGroup里面才能显示出来的，而测算View的大小也是在ViewGroup里面完成的。ViewGroup中定义了一个measureChildren()方法来去测量子视图的大小，如下所示：

```
protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {  
    final int size = mChildrenCount;  
    final View[] children = mChildren;  
    for (int i = 0; i < size; ++i) {  
        final View child = children[i];  
        if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {  
            measureChild(child, widthMeasureSpec, heightMeasureSpec);  
        }  
    }  
}  
```

在measureChildren()方法里面遍历了所有的View，并且调用measureChild()方法：

```
protected void measureChild(View child, int parentWidthMeasureSpec,  
        int parentHeightMeasureSpec) {  
    final LayoutParams lp = child.getLayoutParams();  
    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,  
            mPaddingLeft + mPaddingRight, lp.width);  
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,  
            mPaddingTop + mPaddingBottom, lp.height);  
    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);  
}  
```

可以看到，在第4行和第6行分别调用了getChildMeasureSpec()方法来去计算子视图的MeasureSpec，计算的依据就是布局文件中定义的MATCH_PARENT、WRAP_CONTENT等值，这个方法的内部细节就不再贴出。然后在第8行调用子视图的measure()方法，并把计算出的MeasureSpec传递进去，之后的流程就和前面所介绍的一样了。

这样，View绘制流程的第一步就完成了。

##### onLayout()

measure过程结束后，View的大小就已经测量好了，接下来就是layout的过程了。这个方法是用于给View进行布局的，也就是确定View的位置。View的onMeasure和onLayout的调用都是在ViewRootImpl的performTraversals()方法中，在onMeasure()方法执行完成后还会接着调用View的layout()。

```
// 这段代码存在于performLayout()方法中
host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());
```

layout()方法接收四个参数，分别代表左上右下的坐标，需要注意的是这里的坐标值都是相对于当前的父布局来说的，是相对坐标。host代表的就是当前的View。所以这里的`host.getMeasuredWidth()`和`host.getMeasuredHeight()`就是上一步中测算出来的值。

```
public void layout(int l, int t, int r, int b) {
        if ((mPrivateFlags3 & PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT) != 0) {
            onMeasure(mOldWidthMeasureSpec, mOldHeightMeasureSpec);
            mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
        }

        int oldL = mLeft;
        int oldT = mTop;
        int oldB = mBottom;
        int oldR = mRight;

        boolean changed = isLayoutModeOptical(mParent) ?
                setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

        if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
            onLayout(changed, l, t, r, b);

            if (shouldDrawRoundScrollbar()) {
                if(mRoundScrollbarRenderer == null) {
                    mRoundScrollbarRenderer = new RoundScrollbarRenderer(this);
                }
            } else {
                mRoundScrollbarRenderer = null;
            }

            mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;

            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnLayoutChangeListeners != null) {
                ArrayList<OnLayoutChangeListener> listenersCopy =
                        (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
                int numListeners = listenersCopy.size();
                for (int i = 0; i < numListeners; ++i) {
                    listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
                }
            }
        }

        mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;
        mPrivateFlags3 |= PFLAG3_IS_LAID_OUT;
    }
```

在这个方法中，会先调用`setFrame()`方法来判断View的大小是否已经发生了变化。同时还会在这里把传递过来的四个参数分别赋值给mLeft、mTop、mRight和mBottom这几个变量。接下来就会调用`onLayout()`，这个方法在View中是一个空方法:

```
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
    }
```

因为一个View的布局是由父布局完成的，所以这个方法已经在`ViewGroup`中来实现。我们知道`ViewGroup`是所有布局的父类，而`onLayout()`也是一个抽象方法，所以这个方法会在`LinearLayout`等子类来实现的。所以一般来说，如果我们要自定义一个View是不需要重写这个方法的，当我们需要自定义一个布局的时候才会用到它。比如我们需要一个侧滑布局-`DrawerLayout`。

在五个布局中，`FrameLayout`是最简单的一个布局，我们可以看一下这个代码:

```
    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        layoutChildren(left, top, right, bottom, false /* no force left gravity */);
    }
```

```
void layoutChildren(int left, int top, int right, int bottom, boolean forceLeftGravity) {
        final int count = getChildCount();

        final int parentLeft = getPaddingLeftWithForeground();
        final int parentRight = right - left - getPaddingRightWithForeground();

        final int parentTop = getPaddingTopWithForeground();
        final int parentBottom = bottom - top - getPaddingBottomWithForeground();

        for (int i = 0; i < count; i++) {
            final View child = getChildAt(i);
            if (child.getVisibility() != GONE) {
                final LayoutParams lp = (LayoutParams) child.getLayoutParams();

                final int width = child.getMeasuredWidth();
                final int height = child.getMeasuredHeight();

                int childLeft;
                int childTop;

                int gravity = lp.gravity;
                if (gravity == -1) {
                    gravity = DEFAULT_CHILD_GRAVITY;
                }

                final int layoutDirection = getLayoutDirection();
                final int absoluteGravity = Gravity.getAbsoluteGravity(gravity, layoutDirection);
                final int verticalGravity = gravity & Gravity.VERTICAL_GRAVITY_MASK;

                switch (absoluteGravity & Gravity.HORIZONTAL_GRAVITY_MASK) {
                    case Gravity.CENTER_HORIZONTAL:
                        childLeft = parentLeft + (parentRight - parentLeft - width) / 2 +
                        lp.leftMargin - lp.rightMargin;
                        break;
                    case Gravity.RIGHT:
                        if (!forceLeftGravity) {
                            childLeft = parentRight - width - lp.rightMargin;
                            break;
                        }
                    case Gravity.LEFT:
                    default:
                        childLeft = parentLeft + lp.leftMargin;
                }

                switch (verticalGravity) {
                    case Gravity.TOP:
                        childTop = parentTop + lp.topMargin;
                        break;
                    case Gravity.CENTER_VERTICAL:
                        childTop = parentTop + (parentBottom - parentTop - height) / 2 +
                        lp.topMargin - lp.bottomMargin;
                        break;
                    case Gravity.BOTTOM:
                        childTop = parentBottom - height - lp.bottomMargin;
                        break;
                    default:
                        childTop = parentTop + lp.topMargin;
                }

                child.layout(childLeft, childTop, childLeft + width, childTop + height);
            }
        }
    }
```

这段代码不是很复杂，就是首先获取`FrameLayout`的`Padding`值计算出`FrameLayout`内容可以使用的空间大小，然后通过遍历每一个View，根据它的`Gravity`，`margin`属性计算出View的位置。在方法最后将值回调给`child.layout()`，View可以根据自己需要再次更改

到此为止，我们把视图绘制流程的第二阶段也分析完了。

##### onDraw()

measure和layout的过程都结束后，接下来就进入到draw的过程了。同样，根据名字你就能够判断出，在这里才真正地开始对视图进行绘制。ViewRootImpl中的代码会继续执行并创建出一个Canvas对象，然后调用View的draw()方法来执行具体的绘制工作。draw()方法内部的绘制过程总共可以分为六步，其中第二步和第五步在一般情况下很少用到，因此这里我们只分析简化后的绘制过程。代码如下所示：

```
public void draw(Canvas canvas) {  
    if (ViewDebug.TRACE_HIERARCHY) {  
        ViewDebug.trace(this, ViewDebug.HierarchyTraceType.DRAW);  
    }  
    final int privateFlags = mPrivateFlags;  
    final boolean dirtyOpaque = (privateFlags & DIRTY_MASK) == DIRTY_OPAQUE &&  
            (mAttachInfo == null || !mAttachInfo.mIgnoreDirtyState);  
    mPrivateFlags = (privateFlags & ~DIRTY_MASK) | DRAWN;  
    // Step 1, draw the background, if needed  
    int saveCount;  
    if (!dirtyOpaque) {  
        final Drawable background = mBGDrawable;  
        if (background != null) {  
            final int scrollX = mScrollX;  
            final int scrollY = mScrollY;  
            if (mBackgroundSizeChanged) {  
                background.setBounds(0, 0,  mRight - mLeft, mBottom - mTop);  
                mBackgroundSizeChanged = false;  
            }  
            if ((scrollX | scrollY) == 0) {  
                background.draw(canvas);  
            } else {  
                canvas.translate(scrollX, scrollY);  
                background.draw(canvas);  
                canvas.translate(-scrollX, -scrollY);  
            }  
        }  
    }  
    final int viewFlags = mViewFlags;  
    boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;  
    boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;  
    if (!verticalEdges && !horizontalEdges) {  
        // Step 3, draw the content  
        if (!dirtyOpaque) onDraw(canvas);  
        // Step 4, draw the children  
        dispatchDraw(canvas);  
        // Step 6, draw decorations (scrollbars)  
        onDrawScrollBars(canvas);  
        // we're done...  
        return;  
    }  
}  
```

可以看到，第一步是从第9行代码开始的，这一步的作用是对视图的背景进行绘制。这里会先得到一个mBGDrawable对象，然后根据layout过程确定的视图位置来设置背景的绘制区域，之后再调用Drawable的draw()方法来完成背景的绘制工作。那么这个mBGDrawable对象是从哪里来的呢？其实就是在XML中通过android:background属性设置的图片或颜色。当然你也可以在代码中通过setBackgroundColor()、setBackgroundResource()等方法进行赋值。
接下来的第三步是在第34行执行的，这一步的作用是对视图的内容进行绘制。可以看到，这里去调用了一下onDraw()方法，那么onDraw()方法里又写了什么代码呢？进去一看你会发现，原来又是个空方法啊。其实也可以理解，因为每个视图的内容部分肯定都是各不相同的，这部分的功能交给子类来去实现也是理所当然的。
第三步完成之后紧接着会执行第四步，这一步的作用是对当前视图的所有子视图进行绘制。但如果当前的视图没有子视图，那么也就不需要进行绘制了。因此你会发现View中的dispatchDraw()方法又是一个空方法，而ViewGroup的dispatchDraw()方法中就会有具体的绘制代码。
最后还会绘制一个滚动条，所以，我们可以知道不只是`srcollview`或者`recyclerView`这类的View，任何一个View都有一个滚动条，只是它默认没有开启罢了。
我们可以在`canvas`上面绘制我们想要的任何试图。具体`canvas`是有许多方法的，可以参考[官网文档](https://developer.android.com/reference/android/graphics/Canvas.html)去学习。

#### 实战应用-实现两端对齐的TextView

 记得产品之前提了一个需求，要我们实现类似于微信聊天窗口那样的文本对齐方式-两端对齐。这个在android原生的API里面并没有提供，先看下效果:

![](http://7xjtan.com1.z0.glb.clouddn.com/WechatIMG268.jpeg)

我们发现，微信通过调整文字之间的间距来保证每一行文字的左右端都是对齐的状态的，这样文字看起来就会整齐很多。我们要实现这种效果就要自定义一个TextView，在onDraw()方法中处理每一个文字之间的间隔。思路大概这样:

- 将一篇文章按段落分成若干段
- 将每一段的文字拆分成各个单词，然后根据控件长度确定每一行最多可以填入的单词数，并且算出排满该行还需要填入多大的间距
- 将间距平均填充到文字之间

主要涉及`onDraw()`方法和`calc()`方法

```
@Override
    protected void onDraw(Canvas canvas) {
        TextPaint paint = getPaint();
        paint.setColor(getCurrentTextColor());
        paint.drawableState = getDrawableState();

        width = getMeasuredWidth();

        Paint.FontMetrics fm = paint.getFontMetrics();
        float firstHeight = getTextSize() - (fm.bottom - fm.descent + fm.ascent - fm.top);

        int gravity = getGravity();
        if ((gravity & 0x1000) == 0) { // 是否垂直居中
            firstHeight = firstHeight + (textHeight - firstHeight) / 2;
        }

        int paddingTop = getPaddingTop();
        int paddingLeft = getPaddingLeft();
        int paddingRight = getPaddingRight();
        width = width - paddingLeft - paddingRight;

        for (int i = 0; i < lines.size(); i++) {
            float drawY = i * textHeight + firstHeight;
            String line = lines.get(i);
            // 绘画起始x坐标
            float drawSpacingX = paddingLeft;
            float gap = (width - paint.measureText(line));
            float interval = gap / (line.length() - 1);

            // 绘制最后一行
            if (tailLines.contains(i)) {
                interval = 0;
                if (align == Align.ALIGN_CENTER) {
                    drawSpacingX += gap / 2;
                } else if (align == Align.ALIGN_RIGHT) {
                    drawSpacingX += gap;
                }
            }

            for (int j = 0; j < line.length(); j++) {
                float drawX = paint.measureText(line.substring(0, j)) + interval * j;
                canvas.drawText(line.substring(j, j + 1), drawX + drawSpacingX, drawY +
                        paddingTop + textLineSpaceExtra * i, paint);
            }
        }
    }
```

```
/**
     * 计算每行应显示的文本数
     *
     * @param text 要计算的文本
     */
    private void calc(Paint paint, String text) {
        if (text.length() == 0) {
            lines.add("\n");
            return;
        }
        int startPosition = 0; // 起始位置
        float oneChineseWidth = paint.measureText("中");
        int ignoreCalcLength = (int) (width / oneChineseWidth); // 忽略计算的长度
        StringBuilder sb = new StringBuilder(text.substring(0, Math.min(ignoreCalcLength + 1,
                text.length())));

        for (int i = ignoreCalcLength + 1; i < text.length(); i++) {
            if (paint.measureText(text.substring(startPosition, i + 1)) > width) {
                startPosition = i;
                //将之前的字符串加入列表中
                lines.add(sb.toString());

                sb = new StringBuilder();

                //添加开始忽略的字符串，长度不足的话直接结束,否则继续
                if ((text.length() - startPosition) > ignoreCalcLength) {
                    sb.append(text.substring(startPosition, startPosition + ignoreCalcLength));
                } else {
                    lines.add(text.substring(startPosition));
                    break;
                }

                i = i + ignoreCalcLength - 1;
            } else {
                sb.append(text.charAt(i));
            }
        }
        if (sb.length() > 0) {
            lines.add(sb.toString());
        }

        tailLines.add(lines.size() - 1);
    }
```

看一下实现的效果，上面是系统自带的效果，下面使我们实现的效果:

![](http://7xjtan.com1.z0.glb.clouddn.com/Jietu20171206-195250.jpg)

#### 总结

View是我们日常开发中用到的频率很高的一部分，要掌握好了绘制的基本流程以及一些canvas的基本方法，对于三大流程`onMeasure()`，`onLayout()`以及`onDraw()`，我们要很熟悉他的调用流程以及每个方法的作用，这样很多效果实现起来就比较简单了。
