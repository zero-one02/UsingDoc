# PHConnector 使用文档

## 1. 用户可调用 API 及使用说明

### 构造函数

```csharp
public PHConnector(Com.Logger.ILogger logger)
```
- **参数**：`logger` 日志接口实例（可为 null）
- **说明**：创建 PHConnector 实例，初始化共享内存映射。

---

### 监听相关

#### 开始监听

```csharp
public bool StartListen()
```
- **返回值**：`bool`，是否成功启动监听线程
- **说明**：启动后台线程，监听共享内存中各项状态变化。监听到变化时会触发 `ListenItemChanged` 事件。

#### 停止监听

```csharp
public void StopListenItems()
```
- **说明**：停止监听线程。

#### 监听事件

```csharp
public event EventHandler<ListenItem> ListenItemChanged;
```
- **说明**：监听到关键项变化时触发，事件参数为 `ListenItem` 枚举，指示变化类型（如批次结束、比对失败等）。

---

### 模式与测试控制

#### 设为常规模式

```csharp
public PHConnector SetNormalMode(out bool result)
```
- **参数**：`out bool result`，操作是否成功
- **返回值**：自身实例（链式调用）
- **说明**：将设备设置为常规（非手动）模式。

#### 设为点测（手动）模式

```csharp
public PHConnector SetManualMode(out bool result)
```
- **参数**：`out bool result`，操作是否成功
- **返回值**：自身实例
- **说明**：将设备设置为手动点测模式。

#### 执行一次点测

```csharp
public bool ExecuteSingleTest()
```
- **返回值**：`bool`，操作是否成功
- **说明**：触发一次单次测试。

#### 清除点测标志位

```csharp
public bool ClearSingleTestFlag()
```
- **返回值**：`bool`，操作是否成功
- **说明**：清除单次测试标志。

---

### 卡控项设置与忽略

#### 设置批次号卡控

```csharp
public PHConnector SetLotNo(string lotno, out bool result)
```
- **参数**：`lotno` 批次号字符串；`out bool result` 操作是否成功
- **返回值**：自身实例
- **说明**：设置批次号卡控信息。

#### 忽略批次号卡控

```csharp
public PHConnector IgnoreLotNoControl(out bool result)
```
- **参数**：`out bool result` 操作是否成功
- **返回值**：自身实例
- **说明**：忽略批次号卡控。

#### 设置设备名卡控

```csharp
public PHConnector SetDeviceName(string deviceName, out bool result)
```
- **参数**：`deviceName` 设备名字符串；`out bool result` 操作是否成功
- **返回值**：自身实例
- **说明**：设置设备名卡控信息。

#### 忽略设备名卡控

```csharp
public PHConnector IgnoreDeviceName(out bool result)
```
- **参数**：`out bool result` 操作是否成功
- **返回值**：自身实例
- **说明**：忽略设备名卡控。

#### 设置晶圆片号卡控

```csharp
public PHConnector SetWaferId(byte[] waferId, out bool result)
```
- **参数**：`waferId` 晶圆片号数组（byte[]，每个元素为片号）；`out bool result` 操作是否成功
- **返回值**：自身实例
- **说明**：设置晶圆片号卡控信息。

#### 忽略晶圆片号卡控

```csharp
public PHConnector IgnoreWaferId(out bool result)
```
- **参数**：`out bool result` 操作是否成功
- **返回值**：自身实例
- **说明**：忽略晶圆片号卡控。

#### 设置温度卡控

```csharp
public PHConnector SetTemperature(string temperature, out bool result)
```
- **参数**：`temperature` 温度字符串；`out bool result` 操作是否成功
- **返回值**：自身实例
- **说明**：设置温度卡控信息。

#### 忽略温度卡控

```csharp
public PHConnector IgnoreTemperature(out bool result)
```
- **参数**：`out bool result` 操作是否成功
- **返回值**：自身实例
- **说明**：忽略温度卡控。

#### 晶圆结束时阻塞/暂停

```csharp
public PHConnector HoldWhenWaferEnd(bool hold, out bool result)
```
- **参数**：`hold` 是否阻塞（true 阻塞，false 解除）；`out bool result` 操作是否成功
- **返回值**：自身实例
- **说明**：设置晶圆结束时是否阻塞。

---

## 2. 完整使用样例

```csharp
using Sandtek.OI.PHConnector;
using Sandtek.OI.Com.Logger; // 假设有 ILogger 实现

class Program
{
    static void Main()
    {
        // 1. 创建日志实例（可为 null）
        ILogger logger = new ConsoleLogger();

        // 2. 创建 PHConnector 实例
        var connector = new PHConnector(logger);

        // 3. 订阅监听事件
        connector.ListenItemChanged += (sender, item) =>
        {
            switch (item)
            {
                case ListenItem.LotEnd:
                    Console.WriteLine("检测到结批信号，处理结批逻辑");
                    break;
                case ListenItem.LotNoResult:
                    Console.WriteLine("批次号比对失败，处理异常");
                    break;
                case ListenItem.DeviceNameResult:
                    Console.WriteLine("设备名比对失败，处理异常");
                    break;
                case ListenItem.WaferIdResult:
                    Console.WriteLine("晶圆片号比对失败，处理异常");
                    break;
                case ListenItem.TemperatureResult:
                    Console.WriteLine("温度超限，处理异常");
                    break;
                case ListenItem.WaferEndSignal:
                    Console.WriteLine("检测到晶圆结束信号，处理逻辑");
                    break;
            }
        };

        // 4. 启动监听
        if (connector.StartListen())
        {
            Console.WriteLine("监听已启动");
        }

        // 5. 设置批次号和设备名卡控
        connector.SetLotNo("LOT12345", out bool lotNoResult)
                 .SetDeviceName("DEV001", out bool deviceResult);

        // 6. 忽略设备名卡控
        connector.IgnoreDeviceName(out bool ignoreDeviceResult);

        // 7. 设置为手动点测模式并执行一次点测
        connector.SetManualMode(out bool manualModeResult);
        if (manualModeResult)
        {
            connector.ExecuteSingleTest();
        }

        // 8. 停止监听（如需退出时）
        connector.StopListenItems();
    }
}
```

### 关键注释说明

- **ListenItemChanged 事件**：监听共享内存关键项变化，需订阅处理。
- **链式调用**：如 `SetLotNo().SetDeviceName()` 支持链式设置。
- **out 参数**：所有设置/忽略方法均通过 `out bool result` 返回操作是否成功。
- **模式切换**：`SetNormalMode`/`SetManualMode` 控制设备工作模式。
- **点测控制**：`ExecuteSingleTest` 触发一次点测，`ClearSingleTestFlag` 清除标志。
- **监听线程**：`StartListen`/`StopListenItems` 控制监听线程生命周期。

---

## 3. ListenItem 枚举说明

| 枚举值            | 说明             |
| ----------------- | ---------------- |
| LotEnd            | 结批信号         |
| LotNoResult       | 批次号比对结果   |
| DeviceNameResult  | 设备名比对结果   |
| WaferIdResult     | 晶圆片号比对结果 |
| TemperatureResult | 温度比对结果     |
| WaferEndSignal    | 晶圆结束信号     |

---

## 4. 典型应用场景

- **自动化测试流程**：通过共享内存与外部设备（如 Prober/Handler）交互，自动控制批次、设备、温度等卡控项。
- **异常监控与处理**：实时监听比对结果，及时响应异常信号，提升自动化产线的可靠性。
- **手动点测与调试**：支持切换手动模式，便于调试和特殊场景下的人工干预。

---

如需更详细的底层实现或扩展用法，请参考源码注释及 `ShareMemItem.cs` 内部枚举定义。