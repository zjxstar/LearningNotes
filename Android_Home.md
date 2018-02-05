## 问题描述
部分机型（Opple出现过）安装APP后，直接打开APP，吊起其他页面，使用Home键回到桌面，通过APP图标再次打开APP，发现直接回到了MainActivity，并没有还原之前的页面，且通过返回键却可以回到之前页面。

## 原因分析
部分厂商Rom可能对通过安装APP后直接打开APP和通过桌面图标启动APP走了不同的流程，对Activity栈的处理不完善，导致桌面图标启动找不到Activity栈，反而直接启动了一个新MainActivity。但是，之前启动的Activity栈是存在的，使得通过返回键可以回到之前的页面。

## 解决方案
在MainActivity（这里的MainActivity指代第一个启动的Activity）中加入以下代码：
```
@Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        if (!isTaskRoot()) {
            Intent intent = getIntent();
            if (intent != null && Intent.ACTION_MAIN.equals(intent.getAction()) && intent.hasCategory(Intent.CATEGORY_LAUNCHER)) {
                finish();
                return;
            }
        }
    }
```
