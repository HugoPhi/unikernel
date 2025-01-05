# modules for ArceOS

这个部分我们将详细地介绍每个模块的作用，以及他们可以如何扩展。

## 必须模块

- **axruntime**：负责从裸机环境启动并进行初始化。
- **axhal**：提供硬件抽象层，为跨平台提供统一的API。
- **axconfig**：包含平台常量和内核参数，例如物理内存基地址、内核加载地址、栈大小等。
- **axlog**：提供多级格式化日志记录功能。

### 

## 可选模块（Feature）
其他模块则是可选的，它们依赖于应用程序通过feature启用的功能，包括：
- **axalloc**：ArceOS全局内存分配器，依赖于`alloc` feature。
- **axdisplay**：ArceOS图形模块，依赖于`display` feature。
- **axfs**：ArceOS文件系统模块，依赖于`fs` feature。
- **axnet**：ArceOS网络模块，依赖于`net` feature。
- **axdriver**：ArceOS设备驱动，依赖于`driver-*`, `fs`, `net`, `display` feature。
- **axtask**：ArceOS任务管理模块，依赖于`multitask` feature。
- **axsync**：ArceOS同步原语，依赖于`multitask` feature。



