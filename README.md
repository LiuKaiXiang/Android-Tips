# Android-Tips
Android 开发中的一些技巧及注意事项

## Markdown
Markdown编辑工具 

* Mac OS: [MacDown: The open source Markdown editor for OS X](http://macdown.uranusjr.com)
* Windows: [MarkdownPad - The Markdown Editor for Windows](http://markdownpad.com)

Markdown语法

MacDown在安装完成打开时会同时打开一个help.md文件，其中演示了markdown中涉及到的大部分语法，以供编辑参考。

* 简明版 [Markdown 语法说明(简体中文版)](https://link.zhihu.com/?target=http%3A//wowubuntu.com/markdown/basic.html)
* 完整版 [Markdown 语法说明(简体中文版)](https://link.zhihu.com/?target=http%3A//wowubuntu.com/markdown/index.html)

## Github工作流
不熟悉Github工作流的同学可以参考这篇文章 [Git Workflows and Tutorials](https://github.com/oldratlee/translations/tree/master/git-workflows-and-tutorials)。在日常的代码提交中，要以此工作流的方式提交代码。

## 权限管理

### [Android6.0M权限实战,轻量级封装](https://github.com/linglongxin24/MPermissions)
### [Android悬浮窗权限检查及设置跳转](https://github.com/zhaozepeng/FloatWindowPermission)

## .jar冲突
```
Error:Execution failed for task ':app:transformResourcesWithMergeJavaResForDebug'.
	> com.android.build.api.transform.TransformException: com.android.builder.packaging.DuplicateFileException: Duplicate files copied in APK META-INF/services/javax.annotation.processing.Processor

解决：build.gradle里
	android {
		packagingOptions {
			exclude 'META-INF/services/javax.annotation.processing.Processor'
			}
	}
```

## 编码技巧

* 通过第三方库[butterknife](https://github.com/JakeWharton/butterknife)以注解的方式自动绑定UI和点击事件，安装[zelezny](http://plugins.jetbrains.com/plugin/7369)插件可自动生成代码（右键点击布局代码R.layout.activity_main，选择Generate...）。注意一定要同时依赖butterknife-compiler，否则会出现空异常和点击事件无效的问题

```
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:24.2.1'
    testCompile 'junit:junit:4.12'
    
    compile 'com.jakewharton:butterknife:8.4.0'
    compile 'com.jakewharton:butterknife-compiler:8.4.0'
}

```

```
public class MainActivity extends AppCompatActivity {

    @BindView(R.id.first_button)
    Button firstButton;
    @BindView(R.id.second_button)
    Button secondButton;
    @BindView(R.id.third_button)
    Button thirdButton;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);
    }


    @OnClick({R.id.first_button, R.id.second_button, R.id.third_button})
    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.first_button:
                startActivity(new Intent(this, FirstActivity.class));
                break;
            case R.id.second_button:
                break;
            case R.id.third_button:
                break;
        }
    }
}
```

## 布局

* 沉浸式状态栏

```
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            //透明状态栏
	getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
            //透明导航栏
	getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_NAVIGATION);
}
```

* 设置状态栏字体颜色
```
//设置小米状态栏字体,true是深色，false是浅色
private boolean setMiuiStatusBarDarkMode(Activity activity, boolean darkmode) {
	Class<? extends Window> clazz = activity.getWindow().getClass();
	try {
		int darkModeFlag = 0;
		Class<?> layoutParams = Class.forName("android.view.MiuiWindowManager$LayoutParams");
		Field field = layoutParams.getField("EXTRA_FLAG_STATUS_BAR_DARK_MODE");
		darkModeFlag = field.getInt(layoutParams);
		Method extraFlagField = clazz.getMethod("setExtraFlags", int.class, int.class);
		extraFlagField.invoke(activity.getWindow(), darkmode ? darkModeFlag : 0, darkModeFlag);
		return true;
	} catch (Exception e) {
		LogUtil.e(e.getMessage());
	}
	return false;
}

//设置魅族状态栏字体
private boolean setMeizuStatusBarDarkIcon(Activity activity, boolean dark) {
	boolean result = false;
	if (activity != null) {
		try {
			WindowManager.LayoutParams lp = activity.getWindow().getAttributes();
			Field darkFlag = WindowManager.LayoutParams.class
					.getDeclaredField("MEIZU_FLAG_DARK_STATUS_BAR_ICON");
			Field meizuFlags = WindowManager.LayoutParams.class
					.getDeclaredField("meizuFlags");
			darkFlag.setAccessible(true);
			meizuFlags.setAccessible(true);
			int bit = darkFlag.getInt(null);
			int value = meizuFlags.getInt(lp);
			if (dark) {
				value |= bit;
			} else {
				value &= ~bit;
			}
			meizuFlags.setInt(lp, value);
			activity.getWindow().setAttributes(lp);
			result = true;
		} catch (Exception e) {
			LogUtil.e(e.getMessage());
		}
	}
	return result;
}
```

* EditText设置回车键为搜索
```
//imeOptions也可在.xml里设置（android:imeOptions="actionSearch"）
editText.setImeOptions(EditorInfo.IME_ACTION_SEARCH);
editText.setOnEditorActionListener(new TextView.OnEditorActionListener() {
	@Override
	public boolean onEditorAction(TextView v, int actionId, KeyEvent event) {
		if (actionId == EditorInfo.IME_ACTION_SEARCH) {
			//搜索函数
			search();
			return true;
		}
		return false;
	}
});
```
