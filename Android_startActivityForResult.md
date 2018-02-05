## 问题描述
使用startActivityForResult方法连续启动launchMode为singleTop、singleTask、singleInstance模式的Activity都没有出现相应的launch效果。出现的效果是：会打开多个待启动的Activity。而通过startActivity方法连续启动却是正常的。

## 可能出现的场景（startActivityForResult）
1. 用户快速连续点击启动按钮；
2. 待启动的Activity需要一定的启动时间（比如插件化开发模式），导致用户以为没有触发，再次点击了启动按钮；

## 原因分析
### 测试方案
```
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />    
    </intent-filter>    
</activity>

<activity android:name=".SecondActivity"
    android:launchMode="singleInstance" />
```
启动方式：
```
    public void startForResult(View view) {
        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                Intent intent = new Intent(MainActivity.this, SecondActivity.class);
                startActivityForResult(intent, 1);
            }
        }, 1000); // 1s延时主要是为了看启动效果
    }

    public void start(View view) {
        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                Intent intent = new Intent(MainActivity.this, SecondActivity.class);
                startActivity(intent);
            }
        }, 1000);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        Log.d(TAG, "onActivityResult: " + requestCode);
    }
```
### 测试结果
startActivityForResult对singleTop、singleTask、singleInstance模式都表现为standard的效果。<br>
查看栈信息：（以singleInstance为例）<br>
startActivity一次：
![startActivity一次](https://user-gold-cdn.xitu.io/2018/2/4/1615faef89bc2e4a?w=800&h=356&f=jpeg&s=35232)
startActivity多次：
![startActivity多次](https://user-gold-cdn.xitu.io/2018/2/4/1615faf8f325ce83?w=800&h=385&f=jpeg&s=34860)
startActivityForResult一次：
![startActivityForResult一次](https://user-gold-cdn.xitu.io/2018/2/4/1615fb038f44aa4f?w=800&h=288&f=jpeg&s=27017) 
startActivityForResult多次：
![startActivityForResult多次](https://user-gold-cdn.xitu.io/2018/2/4/1615fafcca1c9988?w=800&h=265&f=jpeg&s=28044) 

### 结果分析
startActivityForResult在低版本（好像5.0之前）有个问题：startActivityForResult所启动的Activity如果是singleTask或者singleInstance的，会立马回调onActivityResult，返回cancel；高版本为了兼容singleTask和singleInstacne模式，把这两种模式都转成standard模式来处理了；所以启动效果就和standard模式一样。

## 解决方案
暂时没有想到怎么通过startActivityForResult方法启动单例模式的Activity，欢迎大神们解惑！