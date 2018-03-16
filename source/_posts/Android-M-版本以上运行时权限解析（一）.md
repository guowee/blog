---
title: Android M 版本以上运行时权限解析（一）
date: 2018-03-16 15:37:57
thumbnail: css/images/xx.jpg
tags:
---

## 一、概述
随着Android 6.0发布以及普及，作为开发者所要应对的主要是新版本SDK带来的一些变化；首先关注的就是权限机制的变化，Android 6.0以后增加了运行时权限（Runtime Permissions）。本篇文章的目的就是对运行时权限处理做一个简单的介绍，并实现在应用开发过程中对权限管理的封装。
> GitHub地址： https://github.com/guowee/EasyPermission

## 二、运行时权限
在Android 6.0以前，在安装应用的时候，会根据权限声明产生一个权限列表，用户只有在同意所有权限以后才能完成app的安装，这样就要忍受一些不必要的权限。而在Android 6.0之后，应用可以直接安装，使用时app需要权限时会给用户提示，可以选择同意或者拒绝（当然也可以在系统设置界面对每个app的权限进行查看，以及对单个权限进行授权或者解除授权）。

新的权限机制更好的保护了用户的隐私，Google 将权限分为两类，一类是Normal Permissions,这类权限一般不涉及用户隐私，不需要用户进行授权，比如访问网络权限；另一类是Dangerous Permissions, 一般涉及到用户的隐私，需要用户进行授权，比如SD卡的读写权限。

- Normal Permissions
```
ACCESS_NETWORK_STATE  
ACCESS_NOTIFICATION_POLICY
ACCESS_WIFI_STATE
BLUETOOTH
CHANGE_NETWORK_STATE
CHANGE_WIFI_MULTICAST_STATE
CHANGE_WIFI_STATE
GET_PACKAGE_SIZE
INTERNET
```
- Dangerous Permissions
```
group:android.permission-group.CONTACTS
  	permission:android.permission.WRITE_CONTACTS
  	permission:android.permission.GET_ACCOUNTS
  	permission:android.permission.READ_CONTACTS
group:android.permission-group.PHONE
  	permission:android.permission.READ_CALL_LOG
  	permission:android.permission.READ_PHONE_STATE
  	permission:android.permission.CALL_PHONE
  	permission:android.permission.WRITE_CALL_LOG
  	permission:android.permission.USE_SIP
  	permission:android.permission.PROCESS_OUTGOING_CALLS
  	permission:com.android.voicemail.permission.ADD_VOICEMAIL
```

Dangerous Permissions 中权限都是分组的，那么分组对权限机制有什么影响呢？
如果app运行在Android 6.0以上版本的机器上，如果你申请某个危险的权限，假设你的app早已被用户授权了同一组的某个危险权限，那么系统会立即授权，而不需要用户去点击授权。比如你的app对READ_CONTACTS已经授权了，当你的app申请WRITE_CONTACTS时，系统会直接授权通过。

## API的使用

1. 在AndroidManifest文件中添加需要的权限
   这个步骤和之前的开发并没有什么变化
2. 检查权限
以申请打电话的权限为例：
   ```
   // ContextCompat.checkSelfPermission()
   // 方法返回值为PackageManager.PERMISSION_DENIED或者PackageManager.PERMISSION_GRANTED
   // 当返回GRANTED表示有该权限，DENIED表示没有该权限
   if(ContextCompat.checkSelfPermission(this,Manifest.permission.CALL_PHONE)!= PackageManager.PERMISSION_GRANTED){
   		//　没有该权限　申请打电话权限
   		//  三个参数 第一个参数是 Context , 第二个参数是用户需要申请的权限字符串数组，第三个参数是请求码 主要用来处理用户选择的返回结果
   		
   }else {
   		...
   }

   ```
这里涉及到一个API方法，`ContextCompat.checkSelfPermission` 主要用于检测某个权限是否已经被授权，方法返回值为`PackageManager.PERMISSION_DENIED` 或者 `PackageManager.PERMISSION_GRANNTED`
3. 申请权限
```
ActivityCompat.requestPermissions(this,new String[]{"Manifest.permission.CALL_PHONE"},CALL_PHONE_REQUEST_CODE);
```
这个方法是异步的，第一个参数是Context,第二个参数是需要申请的权限的字符串数组，第三个参数是requestCode,主要用于回调的时候检查。从第二个参数来看，是支持一次性申请多个权限的，系统会弹出对话框逐一询问用户是否授权。
4. 处理回调
如果用户同意或是拒绝那么会回调`onRequestPermissionsResult()`

```
@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        if(requestCode == CALL_PHONE_REQUEST_CODE){
            if (grantResults !=null&&grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                // permission was granted, yay! Do the
                // contacts-related task you need to do.  
                ...
            } else {
                // permission denied, boo! Disable the
                // functionality that depends on this permission.  
                
            }
        }
}

```
对于权限的申请结果，首先验证requestCode定位到你的申请，然后验证grantResults对应于申请的结果，这里的数组对应于申请时的第二个权限字符串数组。如果你同时申请两个权限，那么grantResults的length就为2，分别记录你两个权限的申请结果。如果申请成功，就可以做你的事情了。

不过还有一个API方法值得注意,就是`ActivityCompat.shouldShowRequestPermissionRationale`

```
// Should we show an explanation?
if (ActivityCompat.shouldShowRequestPermissionRationale(thisActivity,
        Manifest.permission.READ_CONTACTS)) 
    // Show an expanation to the user *asynchronously* -- don't block
    // this thread waiting for the user's response! After the user
    // sees the explanation, try again to request the permission.

}
```
这个API主要用于给用户一个申请权限的解释，该方法只有在用户在上一次已经拒绝过你的这个权限申请。也就是说，用户已经拒绝一次了，又弹个授权框，需要给用户一个解释，为什么要授权，则使用该方法。


## 简单的例子
这里写一个简单的例子，针对于运行时权限。看看直接拨打电话在Android 6.x的设备上权限需要如何处理。
```
public class MainActivity extends AppCompatActivity {
    // 打电话权限申请的请求码
    private static final int REQUEST_PERMISSION_CALL_CODE = 0x0011;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void phoneClick(View view){
        if(ContextCompat.checkSelfPermission(this, Manifest.permission.CALL_PHONE)
                != PackageManager.PERMISSION_GRANTED){
            Toast.makeText(this, "申请权限", Toast.LENGTH_SHORT).show();
            ActivityCompat.requestPermissions(this,
                    new String[]{"Manifest.permission.CALL_PHONE"}, REQUEST_PERMISSION_CALL_CODE);
        }else {
            callPhone();
        }
    }

    /**
    * 拨打电话
    **/
    private void callPhone() {
        Intent intent = new Intent(Intent.ACTION_CALL);
        Uri data = Uri.parse("tel:13111111111");
        intent.setData(data);
        startActivity(intent);
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if(requestCode == REQUEST_PERMISSION_CALL_CODE){
            if (grantResults !=null&&grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                // Permission Granted
                callPhone();
            } else {
                // Permission Denied
                Toast.makeText(this,"权限被拒绝了",Toast.LENGTH_SHORT).show();
            }
        }
    }
}

```

## 封装
虽然权限处理并不复杂，但是需要编写很多重复的代码，所以目前也有很多库对用法进行了封装（可以在github首页搜索）。本节将要讲解的是使用注解和反射的方式封装，封装的代码很简单，接下来看看如何实现。
**(1) 权限申请**
从上面的例子中可以看到，如果申请其他权限，申请的逻辑是类似的，区别就在于参数不同，我们需要三个参数： `Activity\Fragment`，`permissions数组`，`requestCode申请码`， 也就是说只需要写个方法，接收这几个参数即可，然后逻辑就是统一的。

```
 @TargetApi(Build.VERSION_CODES.M)
    private void requestPermissions(Object obj, int requestCode, String[] permissions) {
        if (!PermissionUtil.isOverMarshmallow()) {
            //如果运行在Android 6.0以下的版本上，直接执行方法
            executeSuccess(obj, requestCode);
            return;
        }
        //寻找是否有未被允许的权限
        List<String> deniedPermissions = PermissionUtil.findDeniedPermissions(getActivity(obj), permissions);
        if (deniedPermissions.size() > 0) {
            //存在未允许的权限，需要申请权限
            if (obj instanceof Activity) {
                ((Activity) obj).requestPermissions(deniedPermissions.toArray(new String[deniedPermissions.size()]), requestCode);
            } else if (obj instanceof Fragment) {
                ((Fragment) obj).requestPermissions(deniedPermissions.toArray(new String[deniedPermissions.size()]), requestCode);
            } else {
                throw new IllegalArgumentException(object.getClass().getName() + " is not supported");
            }
        } else {
            // 权限都已经被允许，直接执行方法
            executeSuccess(obj, requestCode);
        }

    }
```
`PermissionUtil.findDeniedPermissions` 就是检查 没有授权的权限.

```
    @TargetApi(Build.VERSION_CODES.M)
    public static List<String> findDeniedPermissions(Activity activity, String... permissions) {
        List<String> deniedPermissions = new ArrayList<>();

        for (String permission : permissions) {
            if (activity.checkSelfPermission(permission) != PackageManager.PERMISSION_GRANTED) {
                deniedPermissions.add(permission);
            }
        }

        return deniedPermissions;
    }
```
那么上述的逻辑就很清晰了，需要的3种参数传入，先去除已经申请的权限，然后开始申请权限。


**(2) 回调处理**

回调主要做的事情：
   1. 了解是否授权
   2. 根据授权情况进行回调

对于第二条，会涉及到两个分支，每个分支执行不同的方法，这里利用注解去确定需要执行的方法，存在两个注解：
```
@PermissionSuccess(requestCode = 0x01)
@PermissionFail(requestCode = 0x01)
```
利用反射根据授权情况+requestCode即可找到注解标注的方法，然后直接执行即可。

大致代码如下：
```
 public static void onRequestPermissionsResult(Object obj, int requestCode, String[] permissions, int[] grantResults) {
        List<String> deniedPermissions = new ArrayList<>();
        for (int i = 0; i < grantResults.length; i++) {
            if (grantResults[i] != PackageManager.PERMISSION_GRANTED) {
                deniedPermissions.add(permissions[i]);
            }
        }

        if (deniedPermissions.size() > 0) {
            executeFail(obj, requestCode);
        } else {
            executeSuccess(obj, requestCode);
        }

    }
```
首先根据grantResults进行判断成功还是失败，对于成功则执行成功的回调：

```
    private static void executeSuccess(Object obj, int requestCode) {
        Method succMethod = PermissionUtil.findMethodWithRequestCode(obj.getClass(), PermissionSuccess.class, requestCode);
        executeMethod(obj, succMethod);
    }
```
`PermissionUtil.findMethodWithRequestCode` 根据注解和requestCode找到方法，然后反射执行即可,代码如下：

```
public static <T extends Annotation> Method findMethodWithRequestCode(Class clazz, Class<T> annotation, int requestCode) {
        for (Method method : clazz.getDeclaredMethods()) {
            if (method.isAnnotationPresent(annotation)) {
                if (isEqualRequestCodeFromAnnotation(method, annotation, requestCode)) {
                    return method;
                }
            }
        }
        return null;
}
```

对于失败则执行失败的回调，代码如下：

```
 private static void executeFail(Object obj, int requestCode) {
        Method failMethod = PermissionUtil.findMethodWithRequestCode(obj.getClass(), PermissionFail.class, requestCode);
        executeMethod(obj, failMethod);
 }
```

到此，对于权限处理的封装基本介绍完了。