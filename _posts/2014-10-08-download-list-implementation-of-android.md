---
layout: post
title: "Android 下载列表实例"
description: ""
category: [Android]
---

## Android 应用开发中的下载列表

#### 下载列表是很多 Android 应用程序需要用到的模块。虽然功能简单，但是在实现的时候，涉及到很多 Android 的组件及其交互，比如：

- LocalBroadcastManager: android.support.v4.content 中定义的本地广播管理组件，只有在相同应用中才可以被接受，不用担心数据泄漏。
- IntentService: Android 官方提供的较易使用的 Service 组件，重载onHandleIntent函数进行单线程操作。
- CompletionService<>: 可以提供一个固定容量的线程池并管理每一个下载任务的线程。
- BroadcastReceiver: 用来接收 LocalBroadcastManager 发送的本地广播，本例中，即为接受下载进度更新数据并刷新界面显示。
- AsyncTask<Void, Void, Void>: 用来执行异步下载任务。
- Callable<>: 可以被 CompletionService<> 管理的线程池中线程执行的具体操作。
- ListView: 用来显示下载项，并且自定义显示项细节。

## 下载列表示意图：
![][dlist]

## 下载列表 Demo

### [DownloadingService gist][1]：执行具体下载任务的 Intent Service。

- 为了控制同时进行下载的任务个数，可以使用一个固定容量的线程池，通过 CompletionService 来管理：

{% highlight java %}
class NoResultType {
}

private CompletionService<NoResultType> mEcs;
...
mExec = Executors.newFixedThreadPool( /* only 5 at a time */1);
mEcs = new ExecutorCompletionService<NoResultType>(mExec);
...
// 将下载任务加入列表，当线程池中有空闲线程的时候就会被执行。
mEcs.submit(t); // t 为 实现了Callable<>接口的执行下载的任务实例
{% endhighlight %}

- 在下载任务启动后就，需要等待每个下载任务完成，然后 Intent Service 就可以退出了。

{% highlight java %}
NoResultType r = mEcs.take().get();
{% endhighlight %}

### [DownloadListActivity gist][2]：显示下载列表的 Activity，每个下载项包含下载项的图标，名称，当前速度和进度。

- 为展示下载列表的 ListView 制定数据适配器：

{% highlight java %}
        ListView listView = mListView = (ListView) findViewById(R.id.list);
		listView.setAdapter(settings.mAdapter = new ArrayAdapter<Progress>(this,
				R.layout.row, R.id.title, new Progress[] { 
				new Progress( 
"同程旅游-特价",
"http://cdn6.down.apk.gfan.com/asdf/Pfiles/2014/9/23/148567_c4fb5598-c533-43f2-a6db-30314962d73e.apk",
"http://cdn2.image.apk.gfan.com/asdf/PImages/2014/6/9/148567f296c959-7c23-445c-80e7-7d024d60204c_icon.png"),
				new Progress(
"英雄之剑",
"http://cdn6.down.apk.gfan.com/asdf/Pfiles/2014/9/22/972551_fb6ba2d0-57a4-40c6-9c03-fe1efe17f4de.apk",
"http://cdn2.image.apk.gfan.com/asdf/PImages/2014/9/22/972551_2705ed211-1120-4c24-815e-a1783e0cb7b6.png"), 
				new Progress(
"智商球",
"http://cdn2.down.apk.gfan.com/asdf/Pfiles/2014/09/26/976043_a7010cbf-08bd-4465-b278-69fb204fe49f.apk",
"http://cdn6.image.apk.gfan.com/asdf/PImages/2014/09/26/f9df602f-95ff-483f-a96a-25fd9b81de0e.png")
				}) {
			@Override
			public View getView(int position, View convertView, ViewGroup parent) {
				View v = super.getView(position, convertView, parent);
				upadteRow(getItem(position), v);
				return v;
			}
		});
{% endhighlight %}

- 动态更新每个下载任务的进度

{% highlight java %}
	private void upadteRow(Progress p, View v) {
		ImageView iv = (ImageView) v.findViewById(R.id.img); 
		iv.setImageBitmap(p.iconImg);
		ProgressBar bar = (ProgressBar) v.findViewById(R.id.progressBar);
		bar.setProgress(p.progress);
		TextView tv = (TextView) v.findViewById(R.id.title);
		tv.setText(p.title);
		tv = (TextView) v.findViewById(R.id.info);
		if(!p.isStarted){
			tv.setText("正在等待");
		}
		else{
			if(p.progress == 100)
			    tv.setText("下载完成");
			else
				tv.setText(p.toString());
				
		}
	}
{% endhighlight %}


### [GlobalSettings gist][3]：单例模式实现的保存全局数据的类型。

### [Progress gist][4]：保存下载状态数据的类型。

### [完整工程下载][5]

[1]: https://gist.github.com/xhrwang/ec504a1c5ada7271f514
[2]: https://gist.github.com/xhrwang/cdea2f609e41ef9e729b
[3]: https://gist.github.com/xhrwang/6b4fb07cd0f7577857ab
[4]: https://gist.github.com/xhrwang/d87d2b1880e951228a12
[5]: http://pan.baidu.com/s/1eQhA7zO
[dlist]: /images/dlist.png


