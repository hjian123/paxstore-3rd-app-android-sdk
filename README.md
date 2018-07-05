# PAXSTORE 3rd App Android SDK [ ![Download](https://api.bintray.com/packages/paxstore-support/paxstore/paxstoresdk/images/download.svg?version=5.02.02) ](https://bintray.com/paxstore-support/paxstore/paxstoresdk/5.02.02/link)
PAXSTORE 3rd App Android SDK provides simple and easy-to-use service interfaces for third-party developers to develop Android Apps on PAXSTORE. The services currently include the following points:

1. Download parameter
2. Inquire update for 3rd party app

By using this SDK, developers can easily integrate with PAXSTORE. Please take care of your AppKey and AppSecret that generated by PAXSTORE system when you create an app.
<br>Refer to the following steps for integration.

## Step 1: Get Secret Key
Create a new app in PAXSTORE, and get AppKey and AppSecret from DEV INFO.

## Step 2: Download Dependency
Gradle:

    implementation 'com.pax.market:paxstoresdk:5.02.02'

## Step 3: Check for Permissions
PAXSTORE Android SDK need the following permissions, please add them in AndroidManifest.xml.

`<uses-permission android:name="android.permission.INTERNET" />`<br>
`<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />`<br>
`<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />`<br>

## Step 4: Initialization
Configuring the application element, edit AndroidManifest.xml, it will have an application element. You need to configure the android:name attribute to point to your Application class (put the full name with package if the application class package is not the same as manifest root element declared package)

<application
    android:name=".BaseApplication"
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:theme="@style/AppTheme">
Initializing AppKey,AppSecret and SN

public class BaseApplication extends Application {

    private static final String TAG = BaseApplication.class.getSimpleName();

    private String APP_KEY = "Your APPKEY";
    private String APP_SECRET = "Your APPSECRET";
    private String SN = Build.SERIAL;

    @Override
    public void onCreate() {
        super.onCreate();
        initPaxStoreSdk();
    }

    private void initPaxStoreSdk() {

       StoreSdk.getInstance().init(getApplicationContext(), appkey, appSecret, SN, new BaseApiService.Callback() {
                  @Override
                  public void initSuccess() {
                      //TODO Do your business here
                  }

                  @Override
                  public void initFailed(RemoteException e) {
                    //TODO Do failed logic here
                      Toast.makeText(getApplicationContext(), "Cannot get API URL from PAXSTORE, Please install PAXSTORE first.", Toast.LENGTH_LONG).show();
                  }
              });
    }
}
## Step 5：Download Parameters API
Download parameter (Optional, ignore this part if you don't have download parameter requirement)
Register your receiver

     <receiver android:name=".YourReceiver">
              <intent-filter>
                  <action android:name="com.paxmarket.ACTION_TO_DOWNLOAD_PARAMS" />
                  <category android:name="Your PackageName" />
              </intent-filter>
     </receiver>
Create your receiver. Since download will cost some time, we recommand you do it in your own service

      public class YourReceiver extends BroadcastReceiver {
          @Override
          public void onReceive(Context context, Intent intent) {
              // since download may cost a long time, we recommand you do it in your own service
              context.startService(new Intent(context, YourService.class));
          }
      }
After you get broadcast, download params in your service

    public int onStartCommand(Intent intent, int flags, int startId) {
            //Specifies the download path for the parameter file, you can replace the path to your app's internal storage for security.
            final String saveFilePath = getFilesDir() + "YourPath";
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    //Call this method to download into your specific directory
                    DownloadResultObject downloadResult = null;
                    try {
                        downloadResult = StoreSdk.getInstance().paramApi().downloadParamToPath(getApplication().getPackageName(), BuildConfig.VERSION_CODE, saveFilePath);
                    } catch (NotInitException e) {
                        Log.e(TAG, "e:" + e);
                    }

                    //businesscode==0, means download successful, if not equal to 0, please check the return message when need.
                    if(downloadResult != null && downloadResult.getBusinessCode()==0){
                        //file download to saveFilePath above.
                        //todo can start to add your logic.
                    }else{
                        //failed do your logic here
                    }
                }
            });
            thread.start();
            return super.onStartCommand(intent, flags, startId);
        }
## Step 6: Update Inquirer (Optional)
Update inquirer: Your app will be asked whether it can be updated when there is a new version afther you
integrated this function.

If you do not have this requirement, just skip this step.

Integrate with this function only need to call initInquirer() after you init StoreSdk success.

    public class BaseApplication extends Application {

        private static final String TAG = BaseApplication.class.getSimpleName();

        private String APP_KEY = "Your APPKEY";
        private String APP_SECRET = "Your APPSECRET";
        private String SN = Build.SERIAL;
        @Override
        public void onCreate() {
            super.onCreate();
            initPaxStoreSdk();  //Initializing AppKey，AppSecret and SN
        }

         private void initPaxStoreSdk() {
                //1. Init AppKey，AppSecret and SN
                StoreSdk.getInstance().init(getApplicationContext(), appkey, appSecret, SN, new BaseApiService.Callback() {
                           @Override
                           public void initSuccess() {
                               initInquirer();
                           }

                           @Override
                           public void initFailed(RemoteException e) {
                               Toast.makeText(getApplicationContext(), "Cannot get API URL from PAXSTORE, Please install PAXSTORE first.", Toast.LENGTH_LONG).show();
                           }
                       });
            }


         private void initInquirer() {
                //2. Init whether app can be updated
                PaxStoreSdk.initInquirer(new StoreSdk.Inquirer() {
                    @Override
                    public boolean isReadyUpdate() {
                        Log.i(TAG, "call business function....isReadyUpdate = " + !isTrading());
                        //todo call your business function here while is ready to update or not
                        return !isTrading();
                    }
                });
         }

        //This is a sample of your business logic method
        public boolean isTrading(){
            return true;
        }
    }

## Step 7：ProGaurd Exclusions
ProGaurd exclusions for SDK

    #Gson
    -dontwarn com.google.gson.**
    -keep class sun.misc.Unsafe { *; }
    -keep class com.google.gson.** { *; }
    -keep class com.google.gson.examples.android.model.** { *; }

    #JJWT
    -keepnames class com.fasterxml.jackson.databind.** { *; }
    -dontwarn com.fasterxml.jackson.databind.*
    -keepattributes InnerClasses
    -keep class org.bouncycastle.** { *; }
    -keepnames class org.bouncycastle.** { *; }
    -dontwarn org.bouncycastle.**
    -keep class io.jsonwebtoken.** { *; }
    -keepnames class io.jsonwebtoken.* { *; }
    -keepnames interface io.jsonwebtoken.* { *; }
    -dontwarn javax.xml.bind.DatatypeConverter
    -dontwarn io.jsonwebtoken.impl.Base64Codec
    -keepnames class com.fasterxml.jackson.** { *; }
    -keepnames interface com.fasterxml.jackson.** { *; }`

#FAQ
###How to resolve dependencies conflict?
When have dependencies conflict, the error message may like below:
'Program type already present: xxx.xxx.xxx '

You can use **exclude()** method to exclude the conflict dependencies by group or module or both.

e.g. To exclude 'com.google.code.gson:gson:2.8.5' in SDK, you can use below:

`implementation ('com.pax.market:paxstoresdk:5.02.02'){
    exclude group: 'com.google.code.gson', module: 'gson'
}`



