# Camera 2

# 整体架构

> [< Android Camera2 HAL3 学习文档 > - 黄龙士 - 博客园](https://www.cnblogs.com/ymjun/p/13201363.html)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200408153026895.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RheWxvclBvdHRlcg==,size_16,color_FFFFFF,t_70)





自上而下分为5层：

1. **应用层 (App Layer)**
2. **Framework API / Java API 层**
3. **原生 Framework 服务层 (Native Framework Services)**
4. **硬件抽象层 (HAL)**
5. **驱动与硬件层 (Kernel Driver & Hardware)**



整个调用流程和数据流是自上而下的，而底层的事件（如对焦完成、拍照回调）和数据（如预览图像流）则是自下而上传递。其核心通信机制是 **Binder IPC**（用于进程间通信）和 **HWBinder**（用于与更底层的 HAL 进程通信）。



#### 第 1 层：应用层 (App Own Code)

- •

  **职责**：实现具体的相机业务逻辑，如拍照、录像、滤镜、人像模式等。

- •

  **实现方式**：开发者调用 Google 提供的标准 **AOSP API**（即 `android.hardware.camera2`包）。

- •

  **关键点**：应用代码运行在**自己的应用进程**中。

#### 第 2 层：App 进程中的 Framework 组件

- •

  **职责**：作为 App 与系统相机服务连接的**客户端代理**。它将应用层的 Java 调用转换为跨进程通信（IPC）请求。

- •

  **关键组件**：

  - •

    `CameraManager`：入口点，用于发现和连接相机设备。

  - •

    `CameraDevice`：代表一个连接的相机设备，用于创建拍摄会话。

  - •

    `ICameraService`：**这是一个关键接口**。它是 `CameraService`在 App 进程中的 **Binder 代理端**。App 通过这个代理与运行在另一个进程（`mediaserver`或 `cameraserver`）中的 `CameraService`进行通信。

- •

  **通信机制**：**Binder IPC**。图中红色部分（如 `ICameraService`）代表 Binder 接口。

#### 第 3 层：Camera Framework (Native Service)

- •

  **职责**：这是相机系统的**核心中枢**。它管理所有相机设备的生命周期、权限控制、协调多个客户端的访问请求，并将上层的标准 API 调用适配到下层的特定 HAL 实现。

- •

  **运行环境**：作为一个独立的**系统服务进程**（通常是 `cameraserver`）。

- •

  **关键组件**：

  - •

    `CameraService`：系统的总管家，实现了 `ICameraService`接口。

  - •

    `CameraDeviceClient`：代表一个与应用建立的连接，处理该客户端的全部请求。

  - •

    `Camera3Device`：实现了 Camera API 2（CAMERA3）的模型，负责配置数据流、处理请求队列。

  - •

    `CameraProvider`：**这是另一个关键接口**。它是与下层 HAL 交互的**客户端代理**。Framework 通过这个代理与运行在另一个独立进程（`vendor.process`）中的 **Camera HAL** 实现进行通信。

- •

  **通信机制**：与下层 HAL 通信使用 **HWBinder**（一种为 HAL 设计的高性能 Binder）。

#### 第 4 层：Camera HAL (Hardware Abstraction Layer)

- •

  **职责**：**由芯片厂商（如 Qualcomm, Mediatek, Spreadtrum）或设备制造商实现**。它将标准的 Android HAL 接口（如 `ICameraDevice`）翻译成对自家芯片平台**专属驱动和图像处理管线（ISP）** 的调用。

- •

  **运行环境**：通常运行在独立的 **Vendor 进程** 中（例如 `vendor.qti.hardware.camera.provider@2.4-service`），与系统进程隔离，保证了稳定性。

- •

  **关键接口**：

  - •

    `ICameraDevice`：代表一个具体的相机硬件设备。

  - •

    `ICameraDeviceSession`：代表一个活跃的相机会话，用于下发拍照/预览请求。

- •

  **关键点**：这是 **Android 开源项目（AOSP）** 和 **厂商私有实现** 的分界线。Google 定义标准接口，厂商负责实现它，从而带来丰富的功能和性能优化。

#### 第 5 层：Camera Driver & Hardware

- •

  **职责**：最底层的软件和硬件。

- •

  **内核驱动**：通过 `ioctl`等系统调用与硬件交互，控制传感器（Sensor）、镜头（Lens）、闪光灯（Flash）等。

- •

  **硬件**：图像传感器（Sensor）、图像信号处理器（ISP）、及其他相关硬件。

好的，我们来系统地解析您提供的这张 Android 相机架构图。

这张图清晰地描绘了 Android 相机系统从上到下的**分层架构**和**跨进程通信**机制，是理解相机如何工作的绝佳蓝图。其核心思想是 **“上层应用与底层硬件实现解耦”**，通过定义标准的接口，让手机厂商（OEM/ODM）可以在下层实现自己的特色功能，而上层应用开发者只需使用统一的 API。

### 架构总览

整个架构遵循经典的 Android **HAL（硬件抽象层）** 原则，自上而下分为五层：

1. 1.

   **应用层 (App Layer)**

2. 2.

   **Framework API / Java API 层**

3. 3.

   **原生 Framework 服务层 (Native Framework Services)**

4. 4.

   **硬件抽象层 (HAL)**

5. 5.

   **驱动与硬件层 (Kernel Driver & Hardware)**

整个调用流程和数据流是自上而下的，而底层的事件（如对焦完成、拍照回调）和数据（如预览图像流）则是自下而上传递。其核心通信机制是 **Binder IPC**（用于进程间通信）和 **HWBinder**（用于与更底层的 HAL 进程通信）。

------

### 分层解析

为了更直观地理解这个分层架构及其交互流程，我们可以参考下面的示意图：

```
flowchart TD
    A[App Own Code<br>使用 Camera2 API] --> B[CameraManager<br>CameraDevice]
    B -- "Binder IPC" --> C[CameraService<br>CameraFramework]
    C -- "HWBinder" --> D[Camera Provider<br>Camera HAL]
    D -- "ioctl" --> E[Camera Driver<br>Sensor Driver]
    
    E -- "数据流上行" --> D
    D -- "事件回调" --> C
    C -- "Binder回调" --> B
    B -- "回调接口" --> A

    subgraph A [应用层 Application]
        A1[Client Activity/Service]
    end

    subgraph B [Framework API层]
        B1[Java Classes]
    end

    subgraph C [原生框架层 Native Framework]
        C1[C++ Services]
    end
    
    subgraph D [硬件抽象层 HAL]
        D1[Vendor Implementation<br>Qcom/Mtk/Spreadtrum]
    end

    subgraph E [Linux内核层 Kernel]
        E1[Driver & Hardware]
    end
```



#### 第 1 层：应用层 (App Own Code)

- •

  **职责**：实现具体的相机业务逻辑，如拍照、录像、滤镜、人像模式等。

- •

  **实现方式**：开发者调用 Google 提供的标准 **AOSP API**（即 `android.hardware.camera2`包）。

- •

  **关键点**：应用代码运行在**自己的应用进程**中。

#### 第 2 层：App 进程中的 Framework 组件

- •

  **职责**：作为 App 与系统相机服务连接的**客户端代理**。它将应用层的 Java 调用转换为跨进程通信（IPC）请求。

- •

  **关键组件**：

  - •

    `CameraManager`：入口点，用于发现和连接相机设备。

  - •

    `CameraDevice`：代表一个连接的相机设备，用于创建拍摄会话。

  - •

    `ICameraService`：**这是一个关键接口**。它是 `CameraService`在 App 进程中的 **Binder 代理端**。App 通过这个代理与运行在另一个进程（`mediaserver`或 `cameraserver`）中的 `CameraService`进行通信。

- •

  **通信机制**：**Binder IPC**。图中红色部分（如 `ICameraService`）代表 Binder 接口。

#### 第 3 层：Camera Framework (Native Service)

- •

  **职责**：这是相机系统的**核心中枢**。它管理所有相机设备的生命周期、权限控制、协调多个客户端的访问请求，并将上层的标准 API 调用适配到下层的特定 HAL 实现。

- •

  **运行环境**：作为一个独立的**系统服务进程**（通常是 `cameraserver`）。

- •

  **关键组件**：

  - •

    `CameraService`：系统的总管家，实现了 `ICameraService`接口。

  - •

    `CameraDeviceClient`：代表一个与应用建立的连接，处理该客户端的全部请求。

  - •

    `Camera3Device`：实现了 Camera API 2（CAMERA3）的模型，负责配置数据流、处理请求队列。

  - •

    `CameraProvider`：**这是另一个关键接口**。它是与下层 HAL 交互的**客户端代理**。Framework 通过这个代理与运行在另一个独立进程（`vendor.process`）中的 **Camera HAL** 实现进行通信。

- •

  **通信机制**：与下层 HAL 通信使用 **HWBinder**（一种为 HAL 设计的高性能 Binder）。

#### 第 4 层：Camera HAL (Hardware Abstraction Layer)

- •

  **职责**：**由芯片厂商（如 Qualcomm, Mediatek, Spreadtrum）或设备制造商实现**。它将标准的 Android HAL 接口（如 `ICameraDevice`）翻译成对自家芯片平台**专属驱动和图像处理管线（ISP）** 的调用。

- •

  **运行环境**：通常运行在独立的 **Vendor 进程** 中（例如 `vendor.qti.hardware.camera.provider@2.4-service`），与系统进程隔离，保证了稳定性。

- •

  **关键接口**：

  - •

    `ICameraDevice`：代表一个具体的相机硬件设备。

  - •

    `ICameraDeviceSession`：代表一个活跃的相机会话，用于下发拍照/预览请求。

- •

  **关键点**：这是 **Android 开源项目（AOSP）** 和 **厂商私有实现** 的分界线。Google 定义标准接口，厂商负责实现它，从而带来丰富的功能和性能优化。

#### 第 5 层：Camera Driver & Hardware

- **职责**：最底层的软件和硬件。
- **内核驱动**：通过 `ioctl`等系统调用与硬件交互，控制传感器（Sensor）、镜头（Lens）、闪光灯（Flash）等。
- **硬件**：图像传感器（Sensor）、图像信号处理器（ISP）、及其他相关硬件。

------

### 总结与核心设计思想

1. **分层解耦**：下层为上层提供服务，上层无需关心下层的具体实现。例如，应用开发者只需调用 `Camera2`API，无需关心手机是骁龙芯片还是联发科芯片。
2. **进程隔离**：
   - **App 进程**、**System Server 进程** (`cameraserver`)、**Vendor 进程** (`camera-provider`) 三者相互隔离。
   - 这种设计极大地提升了系统的安全性和稳定性。即使某个厂商的 HAL 实现崩溃，也不会导致整个系统重启，通常只会让相机应用无法使用。
3. **标准化接口**：
   - **AOSP API** (`Camera2`)：为应用开发者提供统一的标准。
   - **Camera HAL Interface** (HIDL/AIDL)：为芯片厂商和手机制造商提供实现标准。只要实现了标准接口，其相机功能就可以被 Android 系统识别和调用。
4. **Binder IPC 是纽带**：整个架构的核心通信机制是 Binder IPC（包括 HWBinder），它使得跨进程的调用看起来像是在调用本地方法，实现了高效的进程间通信。



### AOSP API 和 Framework 协同工作

![1f3b80a2c06c88](.\image\1f3b80a2c06c88.png)



## camera 的工作大体流程

![2020041117222010](./\image\2020041117222010.png)



![20200412191631734](.\image\20200412191631734.png)



![20200413122743578](.\image\20200413122743578.png)

# 设计核心

**“请求（Request）- 处理（Capture）- 结果（Result）”模型**





# camera源码探索

## Api调用关系

![3768281-40a60559e6988963](.\image\3768281-40a60559e6988963.webp)



## createCaptureSession 模块

![3768281-53061f15ea77c0cf](.\image\3768281-53061f15ea77c0cf.webp)



## setRepeatingRequest 与 Capture模块





## ImageReader 回调接口

![3768281-ffe83e2f5ce7f51e](.\image\3768281-ffe83e2f5ce7f51e.webp)