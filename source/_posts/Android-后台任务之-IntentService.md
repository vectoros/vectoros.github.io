---
title: Android 后台任务之 IntentService
tags:
  - Android
  - Background Tasks
  - IntentService
categories:
  - Android
  - Application
  - Service
abbrlink: 2fa13cac
date: 2019-04-15 15:10:22
---

IntentService 是Android SDK中引入的一个非常好用的，有效的单线程后台任务工具

## 优点
* 不影响UI响应
* 不受生命周期影响

## 限制
* 不能和UI直接交互
* 任务顺序运行，一次只能运行一个任务
* 不能被中断

## 创建方式
1. 继承IntentService类
2. 提供一个默认的构造函数
3. 实现`onHandleIntent(Intent workIntent)`来处理后台任务
4. 在Manifest文件中声明

## 使用IntentService
1. 创建一个Intent，将任务数据用Intent发送给IntentService
2. 调用`startService(Intent workIntent)`启动`IntentService`

## 如何将IntentService处理完的数据返回到Activity
1. 使用`LocalBroadcastManager`，任务完成后，通过广播把数据用Intent返回到Activity


## Demo
假设用后台服务来获取天气信息

* `WeatherFetchService.java`
```Java
public class WeatherFetchService extends IntentService {
    private static final String TAG = "WeatherFetchService";
    private Random random;

    public WeatherFetchService() {
        super(TAG);
        random = new Random();
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        if(intent == null){
            return;
        }
        String city = intent.getStringExtra(Constants.CITY_NAME);
        // Simulate weather information.
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // Return result.
        int t = random.nextInt(100);
        intent.setAction(Constants.ACTION_WEATHER_RESULT);
        intent.putExtra(Constants.TEMPERATURE, t);
        LocalBroadcastManager.getInstance(getApplicationContext()).sendBroadcast(intent);
    }
}
```

* `MainActivity.java`
```Java
public class MainActivity extends AppCompatActivity implements View.OnClickListener{

    private BroadcastReceiver weatherResultReceiver;
    private TextView temperature;
    private EditText city;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        city = findViewById(R.id.city);
        temperature = findViewById(R.id.temperature);
        findViewById(R.id.refresh).setOnClickListener(this);
        findViewById(R.id.clear).setOnClickListener(this);

        weatherResultReceiver = new BroadcastReceiver() {
            @Override
            public void onReceive(Context context, Intent intent) {
                if(intent == null){
                    return;
                }
                String action = intent.getAction();
                if(Constants.ACTION_WEATHER_RESULT.equals(action)){
                    int temp = intent.getIntExtra(Constants.TEMPERATURE,Constants.TEMPERATURE_DEFAULT);
                    temperature.setText(String.valueOf(temp));
                }
            }
        };
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction(Constants.ACTION_WEATHER_RESULT);

        LocalBroadcastManager.getInstance(getApplicationContext()).registerReceiver(weatherResultReceiver, intentFilter);
    }

    @Override
    protected void onDestroy() {
        LocalBroadcastManager.getInstance(getApplicationContext()).unregisterReceiver(weatherResultReceiver);
        super.onDestroy();
    }

    @Override
    public void onClick(View v) {
        int id = v.getId();
        switch (id){
            case R.id.refresh:
                String cityName = city.getText().toString();
                Intent intent = new Intent(MainActivity.this, WeatherFetchService.class);
                intent.putExtra(Constants.CITY_NAME, cityName);
                startService(intent);
                break;
            case R.id.clear:
                city.setText("");
                temperature.setText("");
                break;
            default:
                break;
        }
    }
}
```



## 参考
* https://developer.android.com/guide/components/services
* https://developer.android.com/training/run-background-service/send-request
* https://developer.android.com/training/run-background-service/report-status



