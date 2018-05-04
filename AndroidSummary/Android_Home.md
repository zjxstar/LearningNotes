## 问题描述
安装APP成功后，弹出“完成”和“打开”选择页，直接“打开”APP吊起其他页面，使用Home键回到桌面，通过APP图标再次打开APP，发现直接回到了MainActivity，并没有还原之前的页面，且通过返回键却可以回到之前页面。

## 原因分析
通过系统安装程序安装成功的APP（如：从应用商城下载安装），如果选择直接打开的话，走的是该程序启动Activity的流程，产生的Activity栈是属于该程序的；而直接通过桌面图标启动APP，则走的是APP自己的启动流程，该过程中无法找到安装程序所产生的Activity栈，所以APP会重新启动MainActivity。

**强调：** 以上描述可能不是很清晰，大神们可以指点一二！

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
