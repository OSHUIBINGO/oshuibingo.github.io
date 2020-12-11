---
title: Android 验证码输入框的实现
date: 2019-12-15 23:32:18
tags: Android
categories: Android
---

​		接到公司的一些需求，其中有一个需求挺有意思的，实现一个验证码输入界面，想了一下，感觉还挺简单的，一组TextView和EditText就可以实现了，但是在实现过程中遇到了很多坑，特此记录。

------

# 界面效果

​		实现的界面大概长成这样：

​		![VerifyCodeInputViewDemo](VerifyCodeInputViewDemo.gif)

​		分析：一个验证码输入框+确定按钮。要注意的是，当验证码没有输入完成的时候，确定按钮是灰色且不可点击；当验证码输入完成后，改变背景颜色且可点击。

------

# 具体实现

## 第一步：自定义EditText控件

​		继承重写EditText控件的onSelectionChanged()方法和onTouchEvent()方法。onSelectionChanged()方法是用来监听光标的移动事件，在这个方法里面需要设置光标始终在最后，防止用户控制光标左右移动；onTouchEvent()方法主要对点击事件的处理，我们在里面对双击的事件进行拦截，防止用户双击之后出现出现复制粘贴的选项。

```java
public class VerifyCodeEditText extends AppCompatEditText {

    private long lastTime = 0;

    public VerifyCodeEditText(Context context) {
        super(context);
    }

    public VerifyCodeEditText(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public VerifyCodeEditText(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    protected void onSelectionChanged(int selStart, int selEnd) {
        super.onSelectionChanged(selStart, selEnd);
        //设置光标始终在最后
        this.setSelection(this.getText().length());
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        //拦截双击事件，防止连续双击出现复制粘贴的选项
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                //根据当前点击和上一次点击的时间间隔判断是否是双击
                long currentTime = System.currentTimeMillis();
                if (currentTime - lastTime < 500) {
                    lastTime = currentTime;
                    return true;
                } else {
                    lastTime = currentTime;
                }
                break;
        }
        return super.onTouchEvent(event);
    }
}
```



## 第二步：自定义验证码输入框控件

​		在xml里绘制布局，新建一个类代码里继承RelativeLayout，重写构造方法，在构造方法里面加入我们需要的东西。

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="48dp"
        android:gravity="center"
        android:orientation="horizontal">


        <TextView
            android:id="@+id/tv_0"
            style="@style/VerifyCodeView" />

        <View
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1" />

        <TextView
            android:id="@+id/tv_1"
            style="@style/VerifyCodeView" />

        <View
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1" />

        <TextView
            android:id="@+id/tv_2"
            style="@style/VerifyCodeView" />

        <View
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1" />

        <TextView
            android:id="@+id/tv_3"
            style="@style/VerifyCodeView" />

        <View
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1" />

        <TextView
            android:id="@+id/tv_4"
            style="@style/VerifyCodeView" />

        <View
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1" />

        <TextView
            android:id="@+id/tv_5"
            style="@style/VerifyCodeView" />


    </LinearLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="48dp"
        android:gravity="center"
        android:orientation="horizontal">


        <View
            android:id="@+id/view_0"
            android:layout_width="41dp"
            android:layout_height="2dp"
            android:layout_gravity="bottom"
            android:background="@color/colorPrimary" />

        <View
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1" />

        <View
            android:id="@+id/view_1"
            android:layout_width="41dp"
            android:layout_height="2dp"
            android:layout_gravity="bottom"
            android:background="@color/colorPrimary" />

        <View
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1" />

        <View
            android:id="@+id/view_2"
            android:layout_width="41dp"
            android:layout_height="2dp"
            android:layout_gravity="bottom"
            android:background="@color/colorPrimary" />

        <View
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1" />

        <View
            android:id="@+id/view_3"
            android:layout_width="41dp"
            android:layout_height="2dp"
            android:layout_gravity="bottom"
            android:background="@color/colorPrimary" />

        <View
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1" />

        <View
            android:id="@+id/view_4"
            android:layout_width="41dp"
            android:layout_height="2dp"
            android:layout_gravity="bottom"
            android:background="@color/colorPrimary" />

        <View
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1" />

        <View
            android:id="@+id/view_5"
            android:layout_width="41dp"
            android:layout_height="2dp"
            android:layout_gravity="bottom"
            android:background="@color/colorPrimary" />


    </LinearLayout>

    <com.verifycodedemo.VerifyCodeEditText
        android:id="@+id/verify_code_input_edit_text"
        android:layout_width="match_parent"
        android:layout_height="48dp"
        android:background="@android:color/transparent"
        android:textColor="@android:color/transparent"
        android:longClickable="false"
        android:maxLength="6"
        android:inputType="number" />
</RelativeLayout>
```

```Java
public class VerifyCodeInputView extends RelativeLayout {
    private EditText editText;
    private TextView[] textViews;
    private View[] views;
    private final int MAX = 6;
    private String inputContent;

    private InputContentListener inputContentListener;

    public VerifyCodeInputView(Context context) {
        this(context, null);
    }

    public VerifyCodeInputView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public VerifyCodeInputView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);

        //加载布局文件
        View view= View.inflate(context, R.layout.view_verify_code_input, this);

        //初始化布局ID
        textViews = new TextView[MAX];
        textViews[0] = view.findViewById(R.id.tv_0);
        textViews[1] = view.findViewById(R.id.tv_1);
        textViews[2] = view.findViewById(R.id.tv_2);
        textViews[3] = view.findViewById(R.id.tv_3);
        textViews[4] = view.findViewById(R.id.tv_4);
        textViews[5] = view.findViewById(R.id.tv_5);

        views = new View[MAX];
        views[0] = view.findViewById(R.id.view_0);
        views[1] = view.findViewById(R.id.view_1);
        views[2] = view.findViewById(R.id.view_2);
        views[3] = view.findViewById(R.id.view_3);
        views[4] = view.findViewById(R.id.view_4);
        views[5] = view.findViewById(R.id.view_5);

        editText = view.findViewById(R.id.verify_code_input_edit_text);

        //隐藏光标
        editText.setCursorVisible(false);

        //监听输入的状态
        editText.addTextChangedListener(new TextWatcher() {

            @Override
            public void beforeTextChanged(CharSequence charSequence, int i, int i1, int i2) {

            }

            @Override
            public void onTextChanged(CharSequence charSequence, int i, int i1, int i2) {

            }

            @Override
            public void afterTextChanged(Editable editable) {
                //获取输入框内容
                inputContent = editText.getText().toString();

                if (inputContentListener != null) {

                    //如果输入长度等于字符长度最大值
                    if (inputContent.length() == MAX) {
                        //回调输入完成监听
                        inputContentListener.inputComplete();
                    } else {
                        //否则回调输入内容改变监听
                        inputContentListener.contentChange();
                    }
                }

                //每次输入框内容改变后，把输入框的内容循环赋值给TextView,并改变下划线颜色
                for (int i = 0; i < MAX; i++) {

                    //把输入框的内容循环赋值给TextView
                    if (i < inputContent.length()) {
                        textViews[i].setText(String.valueOf(inputContent.charAt(i)));
                    } else {
                        textViews[i].setText("");
                    }

                    //并改变下划线颜色
                    if (i == inputContent.length()-1){
                        views[i].setBackgroundColor(getContext().getColor(R.color.colorAccent));
                    }else {
                        views[i].setBackgroundColor(getContext().getColor(R.color.colorPrimary));
                    }
                }
            }
        });
    }


    //获取内容
    public String getEditContent() {
        return inputContent;
    }

    //内容清空
    public void setClear() {
        editText.getText().clear();
    }

    public void setInputContentListener(InputContentListener inputContentListener) {
        this.inputContentListener = inputContentListener;
    }

    public interface InputContentListener {

        //输入框完成回调
        void inputComplete();

        //输入框改变回调
        void contentChange();
    }
}
```



## 第三步：使用

​		前置工作都已经完成，下面进行具体的实现。		

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_margin="30dp"
    tools:context=".MainActivity">

    <com.verifycodedemo.VerifyCodeInputView
        android:id="@+id/verify_code_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"

        />

    <Button
        android:id="@+id/bu_verify"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="确认"
        android:textColor="@color/white"
        android:background="@drawable/input_verify_code_button1"
        android:paddingBottom="10dp"
        android:paddingTop="10dp"
        android:paddingRight="135dp"
        android:paddingLeft="135dp"
        android:layout_marginTop="30dp"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/verify_code_view" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

```java
public class MainActivity extends AppCompatActivity {

    Button verifyButton;
    VerifyCodeInputView verifyCodeInputView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        verifyButton = findViewById(R.id.bu_verify);
        verifyButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //清空内容
                verifyCodeInputView.setClear();

                Toast.makeText(getApplicationContext(),"清空内容", Toast.LENGTH_SHORT).show();
            }
        });

        verifyCodeInputView = findViewById(R.id.verify_code_view);
        verifyCodeInputView.setInputContentListener(new VerifyCodeInputView.InputContentListener() {
            @Override
            public void inputComplete() {
                //确认按钮变黄
                verifyButton.setBackgroundResource(R.drawable.input_verify_code_button2);

                //6位验证码输入完成监听
                Toast.makeText(getApplicationContext(), "6位验证码输入完成：" + verifyCodeInputView.getEditContent(), Toast.LENGTH_SHORT).show();
            }

            @Override
            public void contentChange() {
                //确认按钮变灰
                verifyButton.setBackgroundResource(R.drawable.input_verify_code_button1);

                //验证码内容改变监听
                Toast.makeText(getApplicationContext(),"验证码内容改变" + verifyCodeInputView.getEditContent(),Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

-----

# 结语

​		上面没有贴上具体的样式，Demo的GitHub地址：https://github.com/OSHUIBINGO/VerifyCodeInputViewDemo

​		这么做是完成了公司的需求，但是复用性不高，因为引用了很多的xml还有样式之类的，不方便复用，接下来，就提高复用性这方面做出改进，之后也许会出2.0版本的验证码输入框。