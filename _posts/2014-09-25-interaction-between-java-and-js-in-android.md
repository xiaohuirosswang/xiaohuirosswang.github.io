---
layout: post
title: "Android WebView 使用中的JAVA和JS的交互"
description: ""
category: [Android]
---

### Android 使用浏览器内核并基于 HTML5 的应用

- Android 应用中有很多是浏览器内核和响应式网页的 Web App， 很多这样的应用包括大部分的浏览器都是自带内核的，没有使用 Android 系统提供的 WebView 控件，便于深度定制和更灵活的功能设计。而其中一些 App 是基于 WebView 的，而这种情况下经常会用到 Java 代码和 Javascript 代码的交互。

#### 这篇文章用一个应用程序来演示如果在使用 WebView 的时候进行 Java 和 Javascript 之间的交互。

1. 新建一个 Activity，其 layout 配置如下：
{% highlight xml %}
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:paddingBottom="@dimen/activity_vertical_margin"
    tools:context=".MyActivity">

    <TextView
        android:id="@+id/tv_content"
        android:text="@string/hello_world"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />


    <LinearLayout
        android:id="@+id/tab_item_container"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@+id/tv_content"
        android:layout_centerVertical="true"
        android:gravity="bottom"
        android:background="#3c3c3c"
        android:orientation="vertical" >

        <Button
            android:id="@+id/tab_bt_1"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_gravity="bottom"
            android:layout_weight="1"
            android:text="调用JS函数"
            android:textColor="#ffb43c"
            android:textSize="16sp"
            android:gravity="center_vertical"/>

        <Button
            android:id="@+id/tab_bt_2"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_gravity="bottom"
            android:layout_weight="1"
            android:text="调用带参数的JS函数"
            android:textColor="#ffb43c"
            android:textSize="16sp"
            android:gravity="center_vertical"/>
    </LinearLayout>

    <LinearLayout
        android:id="@+id/content_container"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_below="@+id/tab_item_container"
        android:orientation="vertical">

        <TextView
            android:text="WebView"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_margin="3dp"/>

        <WebView
            android:id="@+id/webview"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_margin="3dp"/>
    </LinearLayout>

</RelativeLayout>
{% endhighlight %}

#### UI 元素界面如下图所示：
![][mainui]

2. Activity 的实现如下：
{% highlight java%}
package xhrwang.javainterjs;

import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.webkit.JavascriptInterface;
import android.webkit.WebView;
import android.widget.Button;
import android.widget.TextView;
import android.view.View.OnClickListener;


public class MyActivity extends Activity {

	// WebView element to display web page
    private WebView webView;
    
    // TextView element to response for actions called from javascript
    private TextView tView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_my);
        
        // Initialization
        webView = (WebView) findViewById(R.id.webview);
        tView = (TextView) findViewById(R.id.tv_content);

        // Listen to the click event of the 2 buttons
        Button button = (Button) findViewById(R.id.tab_bt_1);
        button.setOnClickListener(btnClickListener);
        button = (Button) findViewById(R.id.tab_bt_2);
        button.setOnClickListener(btnClickListener);

        // Allow javascript to execute
        webView.getSettings().setJavaScriptEnabled(true);
        
        // Define the interface to be called in javascript
        webView.addJavascriptInterface(this, "Android");
        
        // Load the test page
        webView.loadUrl("file:///android_asset/index.html");
    }

    /*
     * Java method accepting a String type para
     */
    @JavascriptInterface
    public void calledFromJSCodeWithPara(String input){
        final String msg = input;
        
        // http://stackoverflow.com/questions/16758397/sometimes-throws-uncaught-error-error-calling-method-on-npobject-on-android
        // Need to use runOnUiThread in javascript interface functions
        runOnUiThread(new Runnable() {
            public void run() {
                tView.setText("Called from JS code, para is " + msg);
            }
        });

    }

    /*
     * Java method accepting no para
     */
    @JavascriptInterface
    public void calledFromJSCode(){
        runOnUiThread(new Runnable() {
            public void run() {
                tView.setText("Called from JS code.");
            }
        });

    }

    /*
     * Handle the onClick events of the 2 buttons
     * Call javascript functions accordingly
     */
    OnClickListener btnClickListener = new Button.OnClickListener() {
        public void onClick(View v) {
            switch (v.getId()) {
                case R.id.tab_bt_1:
                    // 无参数调用
                    webView.loadUrl("javascript:CalledByJavaCode()");
                    break;
                case R.id.tab_bt_2:
                    // 传递参数调用
                    webView.loadUrl("javascript:CalledByJavaCodeWithPara('From Java Code')");
                    break;
                default:
                    break;
        }
    }
    };
}

{% endhighlight %}

3. 测试使用的 Web page 内容如下：
{% highlight html %}
<!DOCTYPE html>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<script type="text/javascript">
      function CalledByJavaCode() {
	       document.getElementById("content").innerHTML =     
         "java调用了js函数: CalledByJavaCode()";  
      }
	  
	  function CalledByJavaCodeWithPara(arg) {
	       document.getElementById("content").innerHTML =     
         "java调用了js函数: CalledByJavaCodeWithPara(arg)<br/>Para: " + arg;  
      }
</script>
</head>
<body >
            <ul>
                <li><button type="button" onClick="window.Android.calledFromJSCodeWithPara('I am from JS')">Call Java function with para.</button></li>
                <li><button type="button" onClick="window.Android.calledFromJSCode()">Call Java function.</button></li>
            </ul>
			<div id = "content">
			Content area 
			</div>
</body>
</html>
{% endhighlight %}

- 当点击 WebView 上方的两个 button 的时候，会调用测试页面中 Javascript 定义的相应函数，修改页面内容。
- 当点击 WebView 加载的测试页面中的按钮时，会通过 JavascriptInterface 调用 __“Android”__ 接口定义的 Java 方法，修改 TextView 的内容。

### 最后，使用 Javascript 调用 Java 方法是有风险的，[这篇文章][1]介绍了风险的可能危害以及如何避免的方法，很详细，这里就不重复了。

[1]: http://blog.csdn.net/thestoryoftony/article/details/12061023 
[mainui]: /images/javajs.png

