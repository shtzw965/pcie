
# <a id='9'>9 Single Root I/O Virtualization and Sharing</a>
## <a id='9.1'>9.1 SR-IOV Architectural Overview</a>
工业界花费了巨大代价通过使用虚拟化技术提高硬件使用率（如应用执行）。Single Root I/O Virtualization and Sharing (SR-IOV) 让多个System Images（SI）共享PCI硬件资源。

为了展示这项技术如何被用来提高资源使用效率，参考[图9-1](#pic-9-1)所示的通用平台配置。

*<a id='pic-9-1'>图9-1 通用平台配置</a>*

通用平台配置由一下组件组成：
- PCIe RC，包含：
  - 处理器 - 通用、嵌入式、或专用处理器单元
  - 内存 - 通用或嵌入式
  - RC集成EP（RCiEP）
  - PCIe根端口（RP） - 每个RP代表一个独立的层次结构。每个层次结构都对应一个单一的根层次结构，以区别于MR-IOV中定义的多层次结构技术。
- PCIe Switch - 提供IO扇出和连接
  - PCIe设备 - 多IO设备类型（例如网络存储等）
  - System Image - 像操作系统的软件，用于执行程序或可信服务（例如共享或非共享IO设备驱动）

为了无需改变硬件提高硬件资源使用效率，可以执行多SI。如[图9-2](#pic-9-2)所示，在硬件和SI之间插入了称为Virtualization Intermediary (VI) 的软件。

*<a id='pic-9-2'>图9-2 Generic Platform Configuration with a VI and Multiple SI</a>*

VI完全拥有底层硬件的所有权。VI使用超出本规范范围的多种方法对硬件进行抽象，为每个SI提供其自身的虚拟系统。每个SI可用的实际硬件资源随负载或客户策略变化。虽然这种方法在许多环境工作良好，但IO密集负载面临严重性能下降。VI必须拦截并处理每个IO操作，包括进和出，这回显著增加平台资源开销。

SR-IOV提供了减少这些平台资源开销的工具。SR-IOV的好处有：
- 能够消除VI对主要数据传输操作（DMA、内存空间访问、中断处理等）参与。消除VI对每个IO操作的拦截和处理可以显著提高程序和平台的性能。
- 通过Single Root PCI Manager（SR-PCIM）控制SR-IOV资源配置和管理的标准化方法。
  - 由于存在多种实施方案（例如系统固件，VI，操作系统，IO驱动），SR-PCIM的实现不在本规范的范围内。
- 能够在设备内提供大量IO Function的同时，降低硬件要求和相关成本。
- 能够把SR-IOV与其他IO虚拟化技术（如ATS、ATPT和中断重映射）集成，从而创建强大完整的IO虚拟化解决方案。

[图9-3](#pic-9-3)展示了一个有SR-IOV能力的平台示例。

SR-IOV通用平台配置由以下额外功能元件组成：

- SR-PCIM - 负责配置SR-IOV capability、管理PF和VF，以及处理相关错误事件和整体设备控制（例如电源管理和热插拔服务）的软件。
- 可选的Translation Agent（TA） - TA是负责把PCIe trans中地址转化为平台物理地址的硬件或软硬件。TA可能包含Address Translation Cache（ATC）以加速访问转换表。TA可能同时支持Address Translation Serviecs（ATS），ATS使得PCIe Function可以获得对相关存储进行DMA访问的先验地址转换。ATS的益处和操作详见[第10章](#10)。
- 可选的Address Translation and Protection Table（ATPT）- ATPT包含一系列TA处理PCIe请求（DMA读，DMA写，或中断请求）需要访问的地址映射。详见[第10章](#10)。
  - PCIe中把中断当作内存写操作。根据requester id和PCIe trans中的地址，可以将中断路由到任何目标（例如，处理器内核），而这对于相关IO Function是透明的。
  - DMA读写请求根据Routing ID和PCIe trans中的地址进行转换。
- 可选的Address Translation Cache（ATC） - 一个ATC在平台中可以存在于两个位置，在RC集成的或其上的TA中，或在PCIe设备中。RC中，ATC能够加速翻译查找。

### <a id='9.3.2'>9.3.2 Configuration Space</a>
支持SR-IOV的PF应该按照接下来的章节实现SR-IOV Extended Capability。VF应该按照接下来的章节实现配置空间字段和能力。
### <a id='9.3.3'>9.3.3 SR-IOV Extended Capability</a>
##### <a id='9.3.3.3.5'>9.3.3.3.5 ARI Capable Hierarchy</a>
对于连接了上游的设备，该位表示设备正上方的RP或DS是否启用了ARI。软件应该将该位设置于设备正上方的RP或DS的ARI Forwarding Enable匹配。

该位只出现在设备编号最小的PF（如PF0）中，并且影响设备所有的PF。设备其他PF中的该位只读为0。

设备可能根据该位的设置决定Fisrt VF Offset（见[9.3.3.9节](#9.3.3.9)）和VF Stride（见[9.3.3.10节](#9.3.3.10)）的值。在任何PF的VF Eanble置位时改变该位的后果未定义。常规复位后该位必须设为默认值。任何PF和VF的FLR不影响该位的值。如果ARI Capable Hierarchy Preserved（见[9.3.3.2.2节](#9.3.3.2.2)）或No_Soft_Reset（见[9.6.2节](#9.6.2)）置位，所属PF从D3Hot到D0的电源状态改变不影响该位的值（见[9.6.2节](#9.6.2)）。

该位对RCiEP不适用。

> ## *实现须知*
> **ARI Capable Hierarchy**
> 
> 连接了上游的设备无法判断是否启用了ARI。如果启用了ARI，设备能够把所捕获的Bus Number中大于7的Function Number分配给VF以节省Bus Number。[6.13节](#6.13)定义了ARI。

由于RCiEP没有连接上游，ARI不适用，可以把RC中First VF Offset和VF Stride允许的任何Function Number分配给VF（见[9.3.3.8节](#9.3.3.8)和[9.3.3.9节](#9.3.3.9)）。

## <a id='9.5'>9.5 SR-IOV Interrupts</a>
支持SR-IOV的设备使用与[6.1节](#6.1)中定义相同的中断信号机制。

### <a id='9.5.1'>9.5.1 Interrupt机制</a>
传递中断的三种方式：
- INTx
- MSI
- MSI-X

PF可以实现INTx。VF不得实现INTx。PF和VF如果需要中断资源，应该实现MSI或/和MSI-X。每个PF和VF都必须实现其各自的中断capability。

#### <a id='9.5.1.1'>9.5.1.1 MSI Interrupts</a>
除非[表9-40](#tab-9-40)另有规定，MSI capability、PF和VF功能在[7.7节](#7.7)中定义。

*<a id='tab-9-40'>表9-40 MSI Capability: Message Control</a>*

#### <a id='9.5.1.2'>9.5.1.2 MSI-X Interrupts</a>
MSI-X capability在[7.7节](#7.7)中定义，并在[图9-24](#pic-9-24)中描述。

*<a id='pic-9-24'>图9-24 MSI-X Capability</a>*

PF和VF的功能与[7.7.2节](#7.7.2)中定义的Function的功能相同。

注意，VF的Table Offset和PBA Offset是相对于VF的内存地址空间而言的。

#### <a id='9.5.1.3'>9.5.1.3 Address Range Isolation</a>
如果映射MSI-X Talbe或MSI-X PBA地址空间的BAR也映射了其他与MSI-X结构无关的可用地址空间，则其他地址空间中使用的位置（如用于CSR）不得与任何MSI-X结构所在的自然对齐的System Page Size地址范围共享任何地址。MSI-X Table和MSI-X PBA可以共存于自然对齐的System Page Size地址范围内，但它们之间不得重叠。

## <a id='9.6'>9.6 SR-IOV Power Management</a>
本节定义了PCIe SR-IOV电源管理能力和协议。

PF需要[第5章](#5)描述的Power Management Capability 。

对VF，Power Management Capability是可选的。

### <a id='9.6.1'>9.6.1 VF Device Power Management States</a>
如果VF没有实现Power Management Capability，则VF的行为如同被编程为所属PF相同的电源状态。

如果VF实现了Power Management Capability，除非[9.6.4](#9.6.4)节另有规定，功能在[7.5节](#7.5)中定义。

如果VF实现了Power Management Capability，PF的电源状态低于VF时，设备行为未定义。软件应该先将VF置于更低电源状态，再降低它们所属PF的电源状态，以避免这种情况。

当VF完成内部初始化，且VF的Bus Master Enable（见[9.3.4.1.3节](#9.3.4.1.3)）或SR-IOV Capability的VF MSE位（见[9.3.3.3节](#9.3.3.3)）置位时，处于D0状态的VF即处于D0active。VF的内部初始化必须在满足以下任一条件时完成：

- VF已经成功响应一个cfg请求（返回CRS除外）。
- 向VF发出FLR之后，符合以下条件之一：
  - FLR发出后已经过去1.0s。
  - VF支持FRS，并且发出FLR后，收到一条Reason Code为FLR Completed的FRS消息。
  - FLR发出后至少过去FLR Time。FLR Time是VF相关Readiness Time Reporting capability的FLR Time值或由系统软件或固件决定的值。
- 在设置一个PF的VF Enable后，符合以下条件之一：
  - VF Enable置位后至少1.0s。
  - PF支持FRS，且在VF Enable置位后，收到一条Reason Code为VF Enabled的FRS消息。
- 在VF从D3Hot到D0转换后，符合以下条件之一：
  - 进入D0的请求发出过后10ms。
  - VF支持FRS，并且发出进入D0的请求后，收到一条Reason Code为D3Hot to D0 Transition Completed的FRS消息。
  - 发出进入D0的请求后，至少过去D3Hot to D0 Time的时间。D3Hot to D0 Time的时间为VF相关Readiness Time Reporting capability中的D3Hot to D0 Time或由系统软件/固件决定的值。
### <a id='9.6.2'>9.6.2 PF Device Power Management States</a>
PF的电源管理状态（D-state）对其关联的VF具有全局影响。如果VF未实现Power Management Capability，则其行为如同处于其关联的PF相同的电源状态。

如果VF实现了Power Management Capability，PF的电源状态低于VF时，设备行为未定义。软件应该先将VF置于更低电源状态，再降低它们所属PF的电源状态，以避免这种情况。

当PF置于D3Hot时：

- 如果No_Soft_Reset位为0，PF在D3Hot到D0转换时执行内部复位，其所有配置状态恢复为默认值。

注意：重置PF会重置VF Enable，这意味着VF不再存在，并且在D3Hot到D0转换完成后，所有VF的特定上下文都会丢失。
- 如果No_Soft_Reset位置位，内部复位不会发生。SR-IOV extended capability保持状态切相关的VF仍然启用。

当PF进入D3Cold状态时，VF不再存在，所有VF的特定上下文都会丢失，PME事件只能由PF发起。

> ## *实现须知*
> **No_Soft_Reset Strongly Recommended**
> 
> 强烈建议所有Multi-Function设备所有Function置位No_Soft_Reset。此建议适用于PF。

### <a id='9.6.3'>9.6.3 Link Power Management State</a>
VF电源状态不影响链路电源状态。

链路电源状态完全由PF中的设置控制，与VF的D-state无关。

### <a id='9.6.4'>9.6.4 VF Power Management Capability</a>
以下表格列出了PF和VF中Power Management Capability的要求。

除非[表9-41](#tab-9-41)和[表9-42](#tab-9-42)中另有规定，PF和VF的功能在[7.5节](#7.5)中定义。

*<a id='tab-9-41'>表9-41 SR-IOV Power Management Control/Status (PMCSR)</a>*
| Bit Location | PF and VF Register Differences From Base | PF Attributes | VF Attribtutes |
| -- | -- | -- | -- |
| 14:13 | Data_Scale | Base | 00b |
| 12:9 | Data_Select | Base | 0000b |
| 3 | **No_Soft_Reset** - 如果VF实现了Power Management capability，VF的本字段必须和相关PF的值相同 | Base | Base |

*<a id='tab-9-42'>表9-42 SR-IOV Power Management Data Register</a>*
| Bit Location | PF and VF Register Differences From Base | PF Attributes | VF Attribtutes |
| -- | -- | -- | -- |
| 7:0 | Data | Base | 00000000b |

### <a id='9.6.5'>9.6.5 VF EmergencyPower Reduction State</a>
如果VF中的Emergency Power Reduction Supported字段非零，该VF将与关联的PF同时进入和退出紧急功率降低状态。软件可以使用PF中的Power Reduction Detected位模拟VF中的相应位。

