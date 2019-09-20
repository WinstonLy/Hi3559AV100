### 一 概述

I2C 控制器为 Master 接口,完成 CPU 对 I 2 C 总线上连接的从设备的读写访问。芯片共20 个 I2C 控制器,主 SOC 子系统
12 个,Sensor Hub 子系统8个。

### 二 功能描述

- 芯片的 I 2 C 是 Master 接口,支持标准时序和非标准时序。
- 支持标准地址(7bit)和扩展地址(10bit)。
- 传输速率支持标准模式(100kbit/s)和快速模式(400kbit/s)。
- 支持 64 x 8bit 的 TX FIFO 和 64 x 8bit 的 RX FIFO。

