
9. Single Root I/O Virtualization and Sharing

9.1 SR-IOV Architectural Overview
工业界花费了巨大代价通过使用虚拟化技术提高硬件使用率（如应用执行）。Single Root I/O Virtualization and Sharing (SR-IOV) 让多个System Images（SI）共享PCI硬件资源。

为了展示这项技术如何被用来提高资源使用效率，参考图9-1所示的通用平台配置。

图9-1 Generic Platform Configuration

通用平台配置由一下组件组成：

·PCIe RC，包含：

  ·处理器 - 通用、嵌入式、或专用处理器单元

  ·内存 - 通用或嵌入式

  ·RC集成EP（RCiEP）

  ·PCIe根端口（RP） - 每个RP代表一个独立的层次结构。每个层次结构都对应一个单一的根层次结构，以区别于MR-IOV中定义的多层次结构技术。

·PCIe Switch - 提供IO扇出和连接

  ·PCIe设备 - 多IO设备类型（例如网络存储等）

  ·System Image - 像操作系统的软件，用于执行程序或可信服务（例如共享或非共享IO设备驱动）

为了无需改变硬件提高硬件资源使用效率，可以执行多SI。如图9-2所示，在硬件和SI之间插入了称为Virtualization Intermediary (VI) 的软件。

图9-2 Generic Platform Configuration with a VI and Multiple SI

VI完全拥有底层硬件的所有权。VI使用超出本规范范围的多种方法对硬件进行抽象，为每个SI提供其自身的虚拟系统。每个SI可用的实际硬件资源随负载或客户策略变化。虽然这种方法在许多环境工作良好，但IO密集负载面临严重性能下降。VI必须拦截并处理每个IO操作，包括进和出，这回显著增加平台资源开销。

SR-IOV提供了减少这些平台资源开销的工具。SR-IOV的好处有：

·能够消除VI对主要数据传输操作（DMA、内存空间访问、中断处理等）参与。消除VI对每个IO操作的拦截和处理可以显著提高程序和平台的性能。

·通过Single Root PCI Manager（SR-PCIM）控制SR-IOV资源配置和管理的标准化方法。

  ·由于存在多种实施方案（例如系统固件，VI，操作系统，IO驱动），SR-PCIM的实现不在本规范的范围内。

·能够在设备内提供大量IO Function的同时，降低硬件要求和相关成本。

·能够把SR-IOV与其他IO虚拟化技术（如ATS、ATPT和中断重映射）集成，从而创建强大完整的IO虚拟化解决方案。

图9-3展示了一个有SR-IOV能力的平台示例。






9.3.3.3.5 ARI Capable Hierarchy

对于连接了上游的设备，该位表示设备正上方的RP或DS是否启用了ARI。软件应该将该位设置于设备正上方的RP或DS的ARI Forwarding Enable匹配。

该位只出现在设备编号最小的PF（如PF0）中，并且影响设备所有的PF。设备其他PF中的该位只读为0。

设备可能根据该位的设置决定Fisrt VF Offset（见9.3.3.9节）和VF Stride（见9.3.3.10节）的值。在任何PF的VF Eanble置位时改变该位的后果未定义。常规复位后该位必须设为默认值。任何PF和VF的FLR不影响该位的值。如果ARI Capable Hierarchy Preserved（见9.3.3.2.2节）或No_Soft_Reset（见9.6.2节）置位，所属PF从D3hot到D0的电源状态改变不影响该位的值（见9.6.2节）。

该位对RCiEP不适用。

实现提示

ARI Capable Hierarchy

连接了上游的设备无法判断是否启用了ARI。如果启用了ARI，设备能够把所捕获的Bus Number中大于7的Function Number分配给VF以节省Bus Number。6.13节定义了ARI。

由于RCiEP没有连接上游，ARI不适用，可以把RC中First VF Offset和VF Stride允许的任何Function Number分配给VF（见9.3.3.8节和9.3.3.9节）。
