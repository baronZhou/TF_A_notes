## 1. Secure Payload Dispatcher (SPD)

#### 1.0. 术语
- FOSS - Free Open Source Software
- OTE - open-source trusted execution environment（TLK）

#### 1.1. OP-TEE Dispatcher
#### 1.2. Trusted Little Kernel (TLK) Dispatcher

TLK 调度程序 (TLK-D) 增加了对 NVIDIA的Trusted Little Kernel (TLK) 的支持，以与可信固件(TF-A) 配合使用。 TLK-D 可以通过将其包含在平台的 makefile 中进行编译。 TLK 主要用于 Tegra SoC，因此虽然 TF-A仅支持 Tegra 上的 TLK，但调度程序代码可能为其他平台编译。

为了编译 TLK-D，我们需要一个 BL32镜像。 因为TLKD 只需要编译，任何 BL32 镜像都可以。 要将 TLK 用作 BL32，请参阅“Build TLK”部分。

一旦 BL32 准备就绪，就可以通过在构建命令中添加`SPD=tlkd`来将 TLKD 包含在映像中。

##### 1.2.1. Trusted Little Kernel (TLK)

TLK 是作为 Secure EL1 运行的可信操作系统。 它是 NVIDIA® Trusted Little Kernel (TLK) 技术的免费开源软件 (FOSS) 版本，它扩展了随着 Little Kernel (LK) 的开发而提供的技术。 您可以从 https://github.com/travisg/lk 下载用于 Arm、x86 和 AVR32 系统的 LK 模块化嵌入式抢占式内核

NVIDIA 实施了其 Trusted Little Kernel (TLK) 技术，该技术被设计为免费和开源的可信执行环境 (OTE)。

TLK 功能包括：
- 小型抢占式内核
- 支持多线程、IPC 和线程调度
- 添加了 TrustZone 功能
- 添加了安全存储
- 在 MIT/FreeBSD 许可下

NVIDIA 对 Little Kernel (LK) 的扩展包括：
- 用户模式
- TA 的地址空间分离
- TLK 客户端应用程序 (CA) 库
- TLK TA 库
- 通过 OpenSSL 的加密库（加密/解密、密钥处理）
- Linux内核驱动程序
- Cortex-A9/A15 支持
- 电源管理
- TrustZone 内存分割（可重新配置）
- 页表管理
- UART 调试支持（USB 计划）

TLK 由 NVIDIA 在 http://nv-tegra.nvidia.com 上的 3rdparty/ote_partner/tlk.git 存储库下托管。 有关 TLK 和 OTE 的详细信息可以在位于“文档”目录_下的 Tegra_BSP_for_Android_TLK_FOSS_Reference.pdf 手册中找到。

##### 1.2.2. Build TLK

要构建和执行 TLK，请按照 Tegra_BSP_for_Android_TLK_FOSS_Reference.pdf 手册中“构建 TLK 设备”部分的说明进行操作。

##### 1.2.3. Input parameters to TLK

TLK 需要 TZDRAM 大小和包含引导参数的结构。 BL2 将此信息作为 bl32_ep_info 结构的成员传递给 EL3 软件，其中 bl32_ep_info 是 bl31_params_t 的一部分（由 X0 中的 BL2 传递）

##### 1.2.3.1. Example

```c
bl32_ep_info->args.arg0 = TZDRAM size available for BL32
bl32_ep_info->args.arg1 = unused (used only on Armv7-A)
bl32_ep_info->args.arg2 = pointer to boot args
```

#### 1.3. Trusty Dispatcher

Trusty 是一组软件组件，支持移动设备上的可信执行环境 (TEE)，由 Google 发布和维护。

可在托管于 https://source.android.com/security/trusty 的 Trusty 的 Android 开源项目 (AOSP) 网页上找到详细信息和构建说明

##### 1.3.1. Boot parameters

通过提供特定于平台的函数，可以将自定义引导参数传递给 Trusty：
```c
void plat_trusty_set_boot_args(aapcs64_params_t *args)
```

如果提供此函数，则必须将 args->arg0 设置为分配给 trusty 的内存大小。 如果平台不提供此函数，但定义了 TSP_SEC_MEM_SIZE，则默认实现将从 TSP_SEC_MEM_SIZE 传递内存大小。 args->arg1 可以设置为特定于平台的参数块，然后 args->arg2 应设置为该块的大小。

##### 1.3.2. Supported platforms

在 Trusted Firmware-A 支持的所有平台中，只有 NVIDIA 的 Tegra SoC 验证和支持 Trusty。

