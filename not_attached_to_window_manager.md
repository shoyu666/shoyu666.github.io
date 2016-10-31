### Dialog.dismiss() ==>  not attached to window manager
```
异步返回后关闭dialog出现如下错误
java.lang.IllegalArgumentException
View=com.android.internal.policy.PhoneWindow$DecorView{7fd6da3 V.E...... R......D 0,0-330,165} not attached to window manager
android.view.WindowManagerGlobal.findViewLocked(WindowManagerGlobal.java:430)
android.view.WindowManagerGlobal.removeView(WindowManagerGlobal.java:356)
android.view.WindowManagerImpl.removeViewImmediate(WindowManagerImpl.java:118)
android.app.Dialog.dismissDialog(Dialog.java:365)
android.app.Dialog.dismiss(Dialog.java:348)
```

```
原因:
Dialog是和Activity相关的,异步返回后,Activity可能已经不在,此时调用dismiss会报上面错误

//android.view.WindowManagerGlobal.findViewLocked
 private int findViewLocked(View view, boolean required) {
        final int index = mViews.indexOf(view);
        if (required && index < 0) {
            throw new IllegalArgumentException("View=" + view + " not attached to window manager");
        }
        return index;
 }
```

```
解决:
既然Dialog是和Activiy相关的,那么可以把Dialog和Activity绑定.
Dialog的显示关闭通过基类BaseActivity操作.

```

```

public abstract class MBaseActivity extends AppCompatActivity implements IActivityBinderDialog {
    private Dialog mBaseDialog;

    @Override
    protected void onDestroy() {
        dismiss();
        super.onDestroy();
    }

    @Override
    public synchronized void showDialog() {
        mBaseDialog = SafeDialog.showDialog(this, mBaseDialog);
    }

    @Override
    public synchronized void dismiss() {
        SafeDialog.dismissDialog(this, mBaseDialog);
        mBaseDialog = null;
    }
}
```

```
public class SafeDialog {
    public static final String TAG="SafeDialog";
    public static Dialog showDialog(Activity ac, Dialog dialog) {
        Lx.d(TAG,"showDialog");
        Dialog newDialog = dialog;
        try {
            if (!ac.isFinishing()) {
                if (dialog == null) {
                    newDialog = you custom dialog;
                }
                if (!newDialog.isShowing()) {
                    newDialog.show();
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return newDialog;
    }

    public static void dismissDialog(Activity ac, Dialog dialog) {
        Lx.d(TAG,"dismissDialog");
        try {
            if (!ac.isFinishing()) {
                if (dialog != null && dialog.isShowing()) {
                    dialog.dismiss();
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```


