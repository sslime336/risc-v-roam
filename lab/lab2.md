### 地址空间

#### 分页相关概念

在没有分页的环境中，应用程序被直接加载到物理内存的相应地址。在编写程序时，需要手动指明程序运行所需要的内存空间，这个空间不能和其他正常运行的程序（其他用户程序和OS内核）重叠，不然可能引发错误。而引入分页机制后，每个应用程序所使用的内存，或者说程序能”看到“的内存范围，是一段连续但虚拟的内存。在编写程序的时候，不需要考虑和其他程序之间的内存关系，只需要设置好一个统一的内存地址（作为程序的起始地址）就可以，程序具体的运行时内存则自动分配。

基于分页，每个应用程序所占用的内存空间被分为多个（虚拟）页面（Page），这些具有相同大小的虚拟页面（实际上一段合理划分大小的内存空间）在物理内存上也有与其相对应的物理页面，叫做”页帧“，

因为每个页面（无论虚拟还是物理）实际上都是一段内存空间，所以要执行在这一个空间中的一条指令时，我们需要知道这个空间的**开始位置**，和要执行的**指令**相对应该空间开始位置的**偏移量**

因为每个页面的大小是固定的，所以我们可以通过**按序编号**来指定页面的位置。所以，对于每一个页面（无论是虚拟的还是物理的），我们称对其编号的号为“页号”（从 0 开始），那么一个页面开始位置的地址（一个页面的**起始地址**）的计算方法：

页面起始地址 = 页面大小 * 页号

而对于该页面中的指令：

指令地址 = 页面起始地址 + 偏移量

总结一下，在分页机制开启后，一个（虚拟 / 物理）地址的表示方法就变成了：

页号 + 偏移量

#### 地址格式与组成（SV39分页模式下）_x64

虚拟地址(39 bits)：

[38:12]虚拟页号 + [11:0]页内偏移（偏移量）

物理地址(56 bits)：

[55:12]物理页号 + [11:0]页内偏移

两者不等长的原因：

在 SV39 分页模式下，只有低 39 位是有效的

SV39 分页模式规定 64 位的虚拟地址的 [63:39] 位必须和第 38 位相同（从 0 位开始计算）否则 MMU 会直接将判定为非法虚拟地址

所以**虚拟地址**中，只有最低的  （当第 38 位为 0 时） 以及最高的  （当第 38 位为 1 时）是**可能**通过 MMU 检查的



#### 页表

将程序的运行空间分为虚拟地址和物理地址后，我们还需要一个数据结构来保存虚拟页面到页帧的**对应关系**，这个结构叫做**页表**

页表中记录了虚拟页号和物理页帧的一一对应关系

注意，页表也需要存储在内存中，同时在 risc-v 架构中，使用了 **CSR**（控制与状态寄存器）来保存页表的根地址（之所以说是根地址，是因为我们在实际中使用的是[多级页表](https://blog.csdn.net/ibless/article/details/81275009)）


#### lab2 相关概念

```rust
/// 物理地址
#[derive(Copy, Clone, Ord, PartialOrd, Eq, PartialEq)]
pub struct PhysAddr(pub usize); 

/// 虚拟地址
#[derive(Copy, Clone, Ord, PartialOrd, Eq, PartialEq)]
pub struct VirtAddr(pub usize);

/// 物理页号
#[derive(Copy, Clone, Ord, PartialOrd, Eq, PartialEq)]
pub struct PhysPageNum(pub usize);

/// 虚拟页号
#[derive(Copy, Clone, Ord, PartialOrd, Eq, PartialEq)]
pub struct VirtPageNum(pub usize);

// ...

/// SV39 分页模式下的页表项，其中[53:10]这44位是物理页号，最低的8位[7:0]，则是标志位
/// 物理页号和全部的标志位以某种固定的格式保存在一个结构体中，它被称为 **页表项** (PTE, Page Table Entry) ，其是利用虚拟页号在页表中查到的结果。
/// 简单来说，PageTableEntry 是对按照 SV39标准排列的物理地址和标志位整体，即 bits 字段，的包装
/// 页表的一个 key-value 格式为 <VirtAddr, PageTableEntry>，这其中，作为 value 的 PageTableEntry 就是虚拟地址 VirtAddr 对应的物理地址和标志位的包装
#[derive(Copy, Clone)]
#[repr(C)]
pub struct PageTableEntry { // 这名字取得多少有点迷惑性……叫 Entry 多少有点不合适= 。=
    pub bits: usize,
}

// ...

/*
我们以逻辑段 `MapArea` 为单位描述一段连续地址的虚拟内存。所谓逻辑段，就是指地址区间中的一段实际可用的地址连续的虚拟地址区间，该区间内包含的所有虚拟页面都以一种相同的方式映射到物理页帧，具有可读/可写/可执行等属性
*/
pub struct MapArea {
    vpn_range: VPNRange,
    data_frames: BTreeMap<VirtPageNum, FrameTracker>,
    map_type: MapType,
    map_perm: MapPermission,
}

pub type VPNRange = SimpleRange<VirtPageNum>; // 一个可迭代的类型

// ...

/// 地址空间是一系列有关联的逻辑段，这种关联一般是指这些逻辑段属于一个运行的程序（目前把一个运行的程序称为任务，后续会称为进程）
pub struct MemorySet {
    page_table: PageTable,
    areas: Vec<MapArea>, // 一系列逻辑段的集合，.bss .text .rodata .data 构成一个程序
}
```

内核也是一个程序，具有一个 `MemorySet` 叫做 `KERNEL_SPACE`
```rust
pub static ref KERNEL_SPACE: Arc<Mutex<MemorySet>> =
    Arc::new(Mutex::new(MemorySet::new_kernel()));

```


