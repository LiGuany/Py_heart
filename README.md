# 欢迎使用 `Py_heart`
**Py_heart** 是一个使用 Python 开发的 适用于遵守 [Bluetooth Heart Rate Service 1.0](https://www.bluetooth.com/specifications/specs/heart-rate-service-1.0/) 协议的 BLE 心率设备的 Windows 端工具，尤其是 `Xiaomi Smart Band 9` 和 `Xiaomi Smart Band 10`的工具。轻量且强大；内置网页端控制台，支持TUI和GUI 双端查看；开放api支持第三方程序获取。

---



## **如何使用**

### 1. 打开程序 `py_heart_x.x.exe`，等待程序在联网获取最新版本后 ， 开始扫描设备 `([xxxx/xx/xx xx:xx.xx.xxx] 扫描设备...... )`  之后操作手表。

### 2. 在手环 > 设置 > 心率广播 > 开启 后等待连接成功，此过程大概 1-5s 。等待输出与下文类似的文本即为连接成功。

```js
[2025/10/18 15:55.38.133] 找到设备: C4:D3:4C:B8:9A:E2: Xiaomi ����
[2025/10/18 15:55.38.133] 连接设备中: C4:D3:4C:B8:9A:E2
[2025/10/18 15:55.43.826] 成功连接设备！

```
### 3. 浏览器打开 监听地址(http://127.0.0.1:6123) ， 你会看见如下图
![alt text](image.png)

### 4. 恭喜你！ 你已经完成了基操！


---
## 注意事项

- ### `连接错误: 'BLEDevice' object has no attribute 'metadata'`：这是在早期的bleak库中，作为一个BleakScanner 中获取广告而无需添加回调的。([源网页](https://github.com/hbldh/bleak/issues/1025))

- ### 解释一下，这其中的 `band_name` 是心率设备蓝牙地址和设备蓝牙名混合的结果，`device_connected` 就是心率设备是否连接，返回的是布尔值bool；`heart_rate` 就是心率；`sensor_contact_1`就是拿的“广播数据”里面的 `flag & 0b00100` , 但是我改了很久的bug，包括使用了其他人的软件，一直都是输出的True；所以我又添加了一个 `sensor_contact_2` ，这个就是判断心率值是否为零，没有其他步骤，建议在实际应用时判断`sensor_contact_1`与`sensor_contact_2`是否都为True
```js
{
  "band_name": "C4:D3:4C:B8:9A:E2: Xiaomi Smart Band 5C35",
  "device_connected": true,
  "heart_rate": 83,
  "sensor_contact_1": true,
  "sensor_contact_2": true,
  "timestamp": "2025-10-18T18:13:01.493901"
}
```

- ### 我api文档中写的GET 是真的仅限GET，不要尝试用POST获取。

- ### 设备名有时乱码，例如 `Xiaomi ����` ，这是正常现象，不会影响程序运行，初步判断问题应该是在bleak中处理蓝牙设备名中有存在空格的情况。

- ### 部分心率设备的蓝牙地址是错误的，这并不是程序、蓝牙硬件或系统的问题，这大概是手环厂家为防止mac地址泄露而导致黑客设备无限连接手环吧，反正在我的Xiaomi Smart Band 9 NFC 中可以发现，你们也可以发现在上方的 `image-1.png`中可以看到，我的设备真实地址是????5c35结尾，而它显示的是 `C4:D3:4C:B8:9A:E2`(这就是我不打码的原因) 

- ### 作者没钱买服务器，所以作者使用的是免费空间搭建的更新服务器（免费空间位于美国 加利福尼亚 洛杉矶 Telus），部分地区会超时，可以在运行时附加参数 `-no_update` 避免更新
![alt text](image-2.png)

- ### 在显示如下图 至 显示连接成功时断开连接，程序会因为接收不到响应导致不会输出任何东西也不会退出（我懒得做超时的解决方法,doge）
```js
[2025/10/18 16:09.46.852] 找到设备: C4:D3:4C:B8:9A:E2: Xiaomi Smart Band 9 5C35
[2025/10/18 16:09.46.852] 连接设备中: C4:D3:4C:B8:9A:E2
[2025/10/18 16:09.47.911] 成功连接设备！
[2025/10/18 16:09.47.911] 开启TUI输出服务
```
- ### 只建议在连接心率设备时，使用 TUI ，在其他时候更建议使用浏览器

- ### 该工具理论上适用于所有遵守 [Bluetooth Heart Rate Service 1.0](https://www.bluetooth.com/specifications/specs/heart-rate-service-1.0/) 协议的 BLE 心率设备 ， 注意是 *理论上* ，不排除某些厂家将心率数据放在 *广播数据的厂商自定义数据*  中
- ### 在小米手环4-7代中 ， 运动心率广播是真正包含在 BLE 广播数据中的 ； 但是在9-10代中(目前)只包含了[Bluetooth Heart Rate Service](https://www.bluetooth.com/specifications/specs/heart-rate-service-1.0/)声明和设备名称（心率广播界面中灰色字体那个），其他设备需要先连接才能获取心率

- ### 可以用 WebBluetooth 简单模拟一下这个过程，在Chrome或者Edge浏览器的控制台中输入以下代码： 
```js
(async () => {
  const device = await navigator.bluetooth.requestDevice({filters: [{ services: ["heart_rate"] }]});
  console.log(device.name);
  const gatt = await device.gatt.connect();
  const hrs = await gatt.getPrimaryService("heart_rate");
  const hrm = await hrs.getCharacteristic("00002a37-0000-1000-8000-00805f9b34fb");
  hrm.addEventListener("characteristicvaluechanged", event => console.log(event.target.value));
  hrm.startNotifications();
})(); 
```
- ### 获得的Heart Rate Measurement 数据也并不直接是心率数据，还需要解析：

- ### 第一个字节是 Flags Field，其中bit0代表心率数据格式，0代表心率数据是u8，1代表心率数据是u16；bit1代表传感器接触检测，0代表传感器与皮肤接触不良，1代表传感器与皮肤接触良好；bit2代表传感器是否支持接触检测，0代表不支持，1代表支持；余下的bit这里不作介绍

- ### 第二到三个字节代表心率数据，具体长度取决于Flags Field的bit0

- ### 第 6 点以后我是参考Tnze大佬的帖子 [🔗小米手环心率直播 miband-heart-rate.exe 的高级使用方法](https://www.bilibili.com/read/cv42151192)(点击直达)

## 进阶操作

- ### GET `/heart_rate_data` - 获取当前心率数据 ， 以 `json` 格式返回各项数据， `device_connected`、 `sensor_contact_1` 和 `sensor_contact_2` 返回都为布尔值；`heart_rate`返回为整数型
```js
{
  "band_name": "C4:D3:4C:B8:9A:E2: Xiaomi Smart Band 5C35",
  "device_connected": true,
  "heart_rate": 83,
  "sensor_contact_1": true,
  "sensor_contact_2": true,
  "timestamp": "2025-10-18T18:13:01.493901"
}
```
- ### GET `/health` - 获取当前程序状态 ， 以 `json` 格式返回各项数据，`device_connected`返回为布尔值
```js
{
  "device_connected": true,
  "status": "running",
  "timestamp": "2025-10-18T18:21:05.067044"
}
```

- ### GET `/detail/<detail>` - 获取当前程序状态 ， 以 `json` 格式返回各项数据，返回值为 `json` 中的 `number` 或 `string`, 经  `jsonify` 处理.可获取信息有`device_name`、`rate`、`sc1`、`sc2` 、`version`(系统版本信息)
```js
83
```


## 更新日志
### 2025/10/18
- ### ?????? 写麻了。还有这tui是真的乱，控制台1秒请求获取4个参数，obs端500ms请求1个心率参数

![玛德瞎几把为了把html挤到一块我直接怒写1000行](image-1.png)
![666 一秒6个请求，这个入是桂](image-3.png)
