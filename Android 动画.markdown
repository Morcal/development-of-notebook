# Android 动画

标签（空格分隔）： 共享元素过度 矢量图 属性动画

---

## Activity共享元素过渡动画
### 共享元素过渡动画：一个共享元素过渡动画决定两个Activity之间的过渡怎么共享它们的视图
changeBounds：改变目标视图的布局边界；
changeClipBounds：裁剪目标视图的边界；
changeTransform：改变目标视图的缩放比例和旋转角度；
changeImageTransform：改变目标图片的大小和缩放比例。

### 使用场景：Activity A跳转到Activity B
在主题中设置使用
```
 <style name="AppTheme" parent="android:Theme.Material">
    <!--开启tranition动画-->
    <item name="android:windowContentTransitions">true</item>
     
    <item name="android:windowEnterTransition">@transition/tran</item>
    <item name="android:windowExitTransition"></item>
    <item name="android:windowReenterTransition"></item>
    <item name="android:windowReturnTransition"></item>
```
在代码中使用
```
getWindows.requestFeature(Window.FEATURE_CONTEXT_TRASITIONS);

```
```
//A 中startActivity
//单个共享元素的调用方式
startActivity(intent,ActivityOptions.makeSceneTransitionAnimation(this, view, "share").toBundle());
//多个共享元素的调用方式
startActivity(intent,ActivityOptions.makeSceneTransitionAnimation(this,
                Pair.create(view, "share"),
                Pair.create(fab, "fab")).toBundle());
```

## VectorDrawable 矢量图的pathData
* 在values下创建paths.xml添加path string
```
<resources>
    <string name="path1">M38,26H26v12h-4V26H10v-4h12V10h4v12h12V26z</string>
    <string name="path2">M20,36h8v-4h-8V36z M6,12v4h36v-4H6z M12,26h24v-4H12V26z</string>
</resources>
```
* drawable目录下创建xml,根节点为vector填充path,android.xml
```
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="48"
    android:viewportHeight="48">
    <group android:name="action1">
            <path
                android:name="head"
                android:fillColor="#9FBF3B"
                android:pathData="@string/path1"/>
    </group>
    <group android:name="action2">
            <path
                android:name="head"
                android:fillColor="#9FBF3B"
                android:pathData="@string/path2"/>
    </group>
</vector>
```
* 创建一个节点为<animated-vector>的元素，给grouph元素设置动画,animation_android.xml
```
<animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/android">

    <target
        android:animation="@animator/shrug"
        android:name="action1" />

    <target
        android:animation="@animator/shrug"
        android:name="action2" />
</animated-vector>
```
* 为布局中的View设置src
```
<ImageView
        android:id="@+id/fab"
        android:layout_width="@dimen/fab_size"
        android:layout_height="@dimen/fab_size"
        android:src="@drawable/animation_android" />
```

* 让动画动起来
```
public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_vector_drawables);
        ImageView androidImageView = (ImageView) findViewById(R.id.android);
        Drawable drawable = androidImageView.getDrawable();
        if (drawable instanceof Animatable) {
            ((Animatable) drawable).start();
        }
    }
}
```

## Reveal 
### Android5.0使用
```
Animator animator = ViewAnimationUtils.createCircularReveal(Circularbutton,  Circularbutton.getWidth()/2, Circularbutton.getHeight()/2,      Circularbutton.getWidth(),0);
animator.setInterpolator(new AccelerateDecelerateInterpolator());
                animator.setDuration(5000);
                animator.start();
```
### 低版本实现
* 使用第三方库：[CircularReveal](https://github.com/ozodrukh/CircularReveal) 兼容2.3+
* 自定义View：自定义View使用ObjectAnimation，通过设置当前半径值来播放圆形动画，重写onDraw()来绘制一个指定半径和圆心的圆就可实现Reveal效果。
```
//自定义View
public class RevealBackgroundView extends View {
    public static final int STATE_NOT_STARTED = 0;//为开始
    public static final int STATE_FILL_STARTED = 1;//开始
    public static final int STATE_FINISHED = 2;//结束
    private static final Interpolator INTERPOLATOR = new AccelerateInterpolator();
    private int state = STATE_NOT_STARTED;
    ObjectAnimation objectAnimation；
    private int startLocationX;
    private int startLocationY;
    private OnStateChangeListener onStateChangeListener;
    
    public RevealBackgroundView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }
    
    public void startFromLocation(int[] tapLocationOnScreen) {
        changeState(STATE_FILL_STARTED);
        startLocationX = tapLocationOnScreen[0];
        startLocationY = tapLocationOnScreen[1];
        revealAnimator = ObjectAnimator.ofInt(this, "currentRadius", 0, getWidth() + getHeight()).setDuration(FILL_TIME);
        revealAnimator.setInterpolator(INTERPOLATOR);
        revealAnimator.addListener(new AnimatorListenerAdapter() {
            @Override
            public void onAnimationEnd(Animator animation) {
                changeState(STATE_FINISHED);
            }
        });
        revealAnimator.start();
    }
    
   @Override
    protected void onDraw(Canvas canvas) {
        if (state == STATE_FINISHED) {
            canvas.drawRect(0, 0, getWidth(), getHeight(), fillPaint);
        } else {
            canvas.drawCircle(startLocationX, startLocationY, currentRadius, fillPaint);
        }
    }
    
    private void changeState(int state) {
        if (this.state == state) {
            return;
        }
 
        this.state = state;
        if (onStateChangeListener != null) {
            onStateChangeListener.onStateChange(state);
        }
        
    public void setOnStateChangeListener(OnStateChangeListener onStateChangeListener) {
        this.onStateChangeListener = onStateChangeListener;
    }
    
    public static interface OnStateChangeListener {
        void onStateChange(int state);
    }
}
```
在Activity中使用
```
public class MainActivity implenments RevealBackgroundView.OnStateChangeListener {

    @Bind(R.id.vRevealBackground)
    RevealBackgroundView vRevealBackground;
    
    protected void onCreate(Bundle saveInstanceState){
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_user_profile);
        setupRevealBackground(savedInstanceState);
    }
    private void setupRevealBackground(Bundle savedInstanceState) {
        vRevealBackground.setOnStateChangeListener(this);
        if (savedInstanceState == null) {
            final int[] startingLocation = getIntent().getIntArrayExtra(ARG_REVEAL_START_LOCATION);
            vRevealBackground.getViewTreeObserver().addOnPreDrawListener(new ViewTreeObserver.OnPreDrawListener() {
                @Override
                public boolean onPreDraw() {
                    vRevealBackground.getViewTreeObserver().removeOnPreDrawListener(this);
                    vRevealBackground.startFromLocation(startingLocation);
                    return false;
                }
            });
        } else {
            userPhotosAdapter.setLockedAnimations(true);
            vRevealBackground.setToFinishedFrame();
        }
    }
    
    @Override
    public void onStateChange(int state) {
        if (RevealBackgroundView.STATE_FINISHED == state) {
            rvUserProfile.setVisibility(View.VISIBLE);
            userPhotosAdapter = new UserProfileAdapter(this);
            rvUserProfile.setAdapter(userPhotosAdapter);
        } else {
            rvUserProfile.setVisibility(View.INVISIBLE);
        }
    }
}
```
另一种实现方式：
提供一个简单的实现思路 [详情](http://blog.csdn.net/singwhatiwanna/article/details/42614953)
> 1.当我们点击Down时，获取地点击事件的屏幕坐标，然后遍历视图树找到的所点击的view.
> 2.通过mTouchTarget.getLocationOnScreen(location)来获取被点击元素的信息。
> 3.绘制时机，view的绘制力流程 背景->自己(onDraw)->子元素(dispatchDraw)->修饰物(onDrawScroolBar),因此需将水波纹的回绘制放在dispatchDraw中。
> 4.出于对性能考虑，选择用postInvalidateDelayed来对view进行重绘。
> 5.为使效果更好，需对延时up事件分发

核心代码片段如下
```
    protected void dispatchDraw(Canvas canvas) {
        super.dispatchDraw(canvas);
        if (!mShouldDoAnimation || mTargetWidth <= 0 || mTouchTarget == null) {
            return;
        }

        if (mRevealRadius > mMinBetweenWidthAndHeight / 2) {
            mRevealRadius += mRevealRadiusGap * 4;
        } else {
            mRevealRadius += mRevealRadiusGap;
        }
        int[] location = new int[2];
        mTouchTarget.getLocationOnScreen(location);
        int left = location[0] - mLocationInScreen[0];
        int top = location[1] - mLocationInScreen[1];
        int right = left + mTouchTarget.getMeasuredWidth();
        int bottom = top + mTouchTarget.getMeasuredHeight();

        canvas.save();
        canvas.clipRect(left, top, right, bottom);
        canvas.drawCircle(mCenterX, mCenterY, mRevealRadius, mPaint);
        canvas.restore();

        if (mRevealRadius <= mMaxRevealRadius) {
            postInvalidateDelayed(INVALIDATE_DURATION, left, top, right, bottom);
        } else if (!mIsPressed) {
            mShouldDoAnimation = false;
            postInvalidateDelayed(INVALIDATE_DURATION, left, top, right, bottom);
        }
    }
```
```
//up事件延迟分发
if(action==MotionEvent.ACTION_UP){
    mIsPressed = false;  
    postInvalidateDelayed(INVALIDATE_DURATION);  
    mDispatchUpTouchEventRunnable.event = event;  
    postDelayed(mDispatchUpTouchEventRunnable, 400);  
    return true; 
}
//强行消耗up事件
    private class DispatchUpTouchEventRunnable implements Runnable {
        public MotionEvent event;

        @Override
        public void run() {
            if (mTouchTarget == null || !mTouchTarget.isEnabled()) {
                return;
            }
            if (isTouchPointInView(mTouchTarget, (int)event.getRawX(), (int)event.getRawY())) {
                mTouchTarget.dispatchTouchEvent(event);
            }
        }
    };
```






