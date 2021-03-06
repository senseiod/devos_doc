# 简单的UEFI引导程序

首先我们利用Rust创建一个项目：

```text
$ cargo new boot
```

由于我们是创建一个全新的OS，因此我们不能使用windows平台、linux平台、mac平台的api以及Rust的std库。

虽然没有一些标准库的支持，不过我们可以依赖别人写好的第三方库，比如[uefi](https://crates.io/crates/uefi)，我们可以尝试使用构建一个完整的引导程序。

让我们打开`boot/Cargo.toml`,在`[dependencies]` 下加入`uefi = "0.4.2"`,就像这样:

```text
[package]
name = "boot"
version = "0.1.0"
authors = ["Steve Moyan <a22137299217@gmail.com>"]
edition = "2018"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
uefi = "0.4.2"
log = "0.4.8"
```

[uefi](https://crates.io/crates/uefi) 是一个UEFI支持库，他将UEFI的一些方法包装了起来，用于构建UEFI引导

log是一个打印库，用于支持类似于`println!`方法的库

让我们编写`src/main.rs`文件

```text
#![no_std]
#![no_main]
#![feature(abi_efiapi)]

#[macro_use]
use log::*;

use uefi::prelude::*;

#[entry]
fn efi_main(handle: Handle, system_table: SystemTable<Boot>) -> Status {
    info!("Hello UEFI!");
    loop{}
}
```

`#![no_std]`用于告诉Rust编译器，我们不需要std的支持。相反，我们只使用core库

`#![no_main]`则用于告知Rust编译器，我们不需要main方法，我们会自己构建一个主函数用于启动

`#![feature(abi_efiapi)]`用于告知Rust编译器，我们需要使用nightly里的abi\_efiapi特性，因为这个特性还不稳定，因此需要手动开启该特性

`#[macro_use]`可以导入log库中的宏函数

`#[entry]`则是告诉Rust编译器，这便是我们的主函数入口

```text
efi_main(handle: Handle, system_table: SystemTable<Boot>) -> Status {
    info!("Hello UEFI!");
    loop{}
}
```

 这段代码其实和使用TianoCore EDK 2 的库时是类似的，让我们放上C版的

```text
#include <Uefi.h>

EFI_STATUS EFIAPI UefiMain(IN EFI_HANDLE Handle,IN EFI_SYSTEM_TABLE *System_Table)
{
    SystemTable->ConOut->OutputString(SystemTable->ConOut,L"Hello World\n");
    return EFI_SUCCESS;
}
```

handle和SystemTable是主函数的两个参数，其中的参数handle是这个UEFI引导程序被加载到内存后生成的image对象的句柄，而参数system\_table则是程序与UEFI固件的交互通道，通过它可以使用UEFI提供的各种服务。

利用log库我们可以直接使用`info!` 打印参数。

让我们尝试编译一下,注意，这里无法使用cargo build,因为cargo build还没有交叉编译，无法为我们构建无主机的系统和UEFI，而cargo xbuild则可以使用

```text
$ cargo xbuild
```

这里有一个警告，它告诉我们我们正在为当前的主机目标生成程序，要求我们自行添加目标系统

```text
WARNING: You're currently building for the host system. This is likely an error and will cause build scripts of dependencies to break.

To build for the target system either pass a `--target` argument or set the build.target configuration key in a `.cargo/config` file.
```

这是什么意思呢？这个意思就好比你需要构建一个系统，但是你得给编译器讲你用的什么CPU、构建为什么格式、使用什么ABI，因此我们需要编写一个目标三元组来告诉编译器，我们在给什么平台构建程序

接下来我们创建一个.cargo文件夹，文件夹下有一个叫做config的文件，编辑该文件

```text
[build]
target = "x86_64-unknown-uefi"
```

然后再尝试编译，没想到还是编译失败

```text
error: `#[panic_handler]` function required, but not found

error: aborting due to previous error

error: could not compile `boot`.
```

接下来我们导入uefi-services在\[dependencies\]下

```text
//  Cargo.toml

uefi-services = "0.2.0"
```

 接下来编写main.rs文件

```text
#[entry]
fn efi_main(handle: Handle, system_table: SystemTable<Boot>) -> Status {
    uefi_services::init(&system_table).expect_success("failed to initialize utilities");
    info!("Hello UEFI!");
    loop {}
}
```

再次编译

```text
cargo xbuild
```

```text
 Finished dev [unoptimized + debuginfo] target(s) in 0.02s
```

终于编译成功，让我们看看target/x86\_64-unknown-uefi/debug/文件夹下面，出现了一个boot.efi，这便是我们编写的第一个UEFI 引导程序！

**QEMU运行**

// TODO: 上传OVMF.fd

在boot文件夹下创建一个名叫Makefile的文件，然后添加以下内容

```text
.PHONY: all build qemu

all: build qemu

# 编译UEFI
build:
	@cargo xbuild
	@make esp_mkdir

# 创建一个ESP文件夹
esp_mkdir:
	@mkdir -p build/pc/esp/EFI/Boot
	@cp target/x86_64-unknown-uefi/debug/boot.efi build/pc/esp/EFI/Boot/BootX64.efi

# QEMU运行x86_64
qemu:
	@qemu-system-x86_64 \
    -drive if=pflash,format=raw,file=OVMF.fd,readonly=on \
    -drive format=raw,file=fat:rw:build/pc/esp \
    -m 2048 \
    -nographic \
    -no-fd-bootchk \
    -smp 2

```

运行结果

```text
BdsDxe: failed to load Boot0001 "UEFI QEMU DVD-ROM QM00003 " from PciRoot(0x0)/Pci(0x1,0x1)/Ata(Secondary,Master,0x0): Not Found
BdsDxe: loading Boot0002 "UEFI QEMU HARDDISK QM00001 " from PciRoot(0x0)/Pci(0x1,0x1)/Ata(Primary,Master,0x0)
BdsDxe: starting Boot0002 "UEFI QEMU HARDDISK QM00001 " from PciRoot(0x0)/Pci(0x1,0x1)/Ata(Primary,Master,0x0)
INFO: Hello UEFI!
```

可以看到，我们的UEFI程序成功运行了！



