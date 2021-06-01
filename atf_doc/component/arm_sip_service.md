## 2. Arm SiP Services

本文档列举并描述了 Arm SiP（Silicon Provider）服务。

SiP 服务是由芯片实施者或平台提供商提供的非标准、特定于平台的服务。 它们通过从低于 EL3 的异常级别执行的 SMC（“SMC 调用”）指令访问。 SMC 要求 SiP 服务：
- 遵循 SMC Calling Convention；
- 使用属于 SiP 范围的 SMC 函数 ID，对于 64 位调用，它们是 0xc2000000 - 0xc200ffff，对于 32 位调用，它们是 0x82000000 - 0x8200ffff。

Arm SiP 实施提供以下服务：
- Performance Measurement Framework (PMF)
- Execution State Switching service
- DebugFS 接口
- Arm SiP 服务的源定义位于 arm_sip_svc.h 头文件中。

#### 2.1. Performance Measurement Framework (PMF)

Performance Measurement Framework允许调用者检索在 TF-A 执行中在不同路径上捕获的时间戳。

##### 2.2.1. ARM_SIP_SVC_EXE_STATE_SWITCH
```c
Arguments:
    uint32_t Function ID
    uint32_t PC hi
    uint32_t PC lo
    uint32_t Cookie hi
    uint32_t Cookie lo

Return:
    uint32_t
```

函数 ID 参数必须为 0x82000020。它唯一标识所请求的执行状态切换服务。

参数 PC hi 和 PC lo 分别定义了执行状态切换后应该开始执行的入口点（物理地址）的高位字和低位字。从 AArch64 调用时，PC hi 必须为 0。

当执行状态切换后从提供的入口点开始执行时，参数 Cookie hi 和 Cookie lo 分别被传递到 CPU 寄存器 0 和 1。从 AArch64 调用时，Cookie hi 必须为 0。

此调用只能在主 CPU 上进行，然后才通过 CPU_ON PSCI 调用启动任何辅助 CPU。否则，调用将始终失败。

切换执行状态的效果就像上电后第一次进入异常级别一样。这意味着具有架构定义的复位值的 CPU 寄存器将采用该值。在进行调用之前，不应期望其他寄存器保持它们的值。但是，CPU 字节序保留了先前的执行状态。请注意，这仅切换调用 CPU 的执行状态。这不能替代 PSCI SYSTEM_RESET。

该服务可能会返回以下错误代码：
- STATE_SW_E_PARAM：如果任何参数被认为对特定请求无效。
- STATE_SW_E_DENIED：如果调用不成功，或者当为 AArch32 构建 TF-A 时。
- 如果调用成功，调用者将不会观察到 SMC 返回。相反，执行从提供的入口点开始，CPU 寄存器 0 和 1 分别填充有提供的 Cookie hi 和 Cookie lo 值。

#### 2.2. Execution State Switching service

执行状态切换服务为非安全的较低异常级别（EL2 或 NS EL1，如果未实现 EL2）请求切换其执行状态（又名寄存器宽度），从 AArch64 到 AArch32，或从 AArch32 到 AArch64，用于调用 CPU。 此服务仅在为 AArch64 构建 Trusted Firmware-A (TF-A) 时可用（即当构建选项 ARCH 设置为 aarch64 时）。

#### 2.3. DebugFS interface

通过 SMC SiP 服务访问可选的 DebugFS 接口。 有关详细信息，请参阅组件文档。

字符串参数使用特定联合通过共享缓冲区传递：

```c
union debugfs_parms {
    struct {
        char fname[MAX_PATH_LEN];
    } open;

    struct mount {
        char srv[MAX_PATH_LEN];
        char where[MAX_PATH_LEN];
        char spec[MAX_PATH_LEN];
    } mount;

    struct {
        char path[MAX_PATH_LEN];
        dir_t dir;
    } stat;

    struct {
        char oldpath[MAX_PATH_LEN];
        char newpath[MAX_PATH_LEN];
    } bind;
};
```

dir_t structure的格式如下:
```c
typedef struct {
    char            name[NAMELEN];
    long            length;
    unsigned char   mode;
    unsigned char   index;
    unsigned char   dev;
    qid_t           qid;
} dir_t;
```

Identifiers
- SMC_OK 0
- SMC_UNK -1
- DEBUGFS_E_INVALID_PARAMS -2

- MOUNT 0
- CREATE 1
- OPEN 2
- CLOSE 3
- READ 4
- WRITE 5
- SEEK 6
- BIND 7
- STAT 8
- INIT 10
- VERSION 11

##### 2.3.1. MOUNT

**2.3.1.1. Description**

此操作使用由存储在 src 中的路径指向的文件系统位置挂载存储在 src 中的路径指向的数据块，使用规范中的路径指向的驱动程序。

**2.3.1.2. Parameters**

uint32_t  FunctionID (0x82000030 / 0xC2000030)

**2.3.1.3. Return values**

- w0 == SMC_OK on success
- w0 == DEBUGFS_E_INVALID_PARAMS if mount operation failed

##### 2.3.2.  OPEN
##### 2.3.3.  CLOSE
##### 2.3.4.  READ
##### 2.3.5.  SEEK
##### 2.3.6.  BIND
##### 2.3.7.  STAT
##### 2.3.8.  INIT
##### 2.3.9.  VERSION