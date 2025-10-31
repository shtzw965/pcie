
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
VI完全拥有底层硬件的所有权。VI使用超出本规范范围的多种方法对硬件进行抽象，为每个SI提供其自身的虚拟系统。每个SI可用的实际硬件资源随负载或客户策略变化。
