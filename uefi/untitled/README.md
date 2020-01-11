# UEFI简介

**Bootstrap\(启动程序\)**

从PC诞生之起，人们就在解决一个问题：硬件需要启动软件，但是启动硬件又需要利用软件来启动，于是机智的人们就发明了一个很小的启动固件和软件，名曰:BIOS。

首先电脑开机，然后首先CPU加电，CPU就会去启动BIOS，BIOS\(base input/output system\)会完成自己的自检，接下来BIOS会将系统文件从文件中读取放在内存中让CPU启动。这里原理我不会讲得很复杂，只是将个大概。BIOS在启动好系统后基本上就完成自己的使命了\(但是BIOS还是会帮助系统完成内存初始化、信息读取等\)。这便是BIOS的基本作用了

**UEFI  BIOS和 Legacy BIOS**

为了不引起争议，我们在这里将Bootstrap分为两种：**UEFI即UEFI BIOS**、**BIOS即Legacy  BIOS**，**这里缩短名称方便理解**。UEFI作为新型的OS启动引导方式，它的特性和功能都要比BIOS好上不少，比如UEFI是由C语言编写的，可读性要比BIOS好上不少，而且UEFI启动速度快、拓展性好等，并且UEFI可以读取大于4TB的硬盘，这是BIOS无法完成的事，原因在于UEFI使用的磁盘分区表为GPT，而BIOS使用的磁盘分区表为MBR。如果你使用的磁盘大小大于或等于4TB，它默认使用的就是GPT分区表

由于UEFI可拓展性强，未来可能会导致UEFI变得臃肿，变得越来越难管理，因此目前第三代bootstrap 正在尝试构建一个轻量化的bootstrap [https://github.com/intel/ModernFW](https://github.com/intel/ModernFW)

由于我们是使用UEFI来启动，因此我们不再赘述UEFI和BIOS的更多区别和细节，具体可以自行了解，以拓宽知识

**UEFI的启动**

**首先你的电脑需要支持UEFI，其实从目前的市场来看，基本上现代的主板都是支持UEFI的，可能有些UEFI的版本不同**

**其次你可能需要一个U盘用于测试自己编写的UEFI和系统，这个下一个章节我们可能需要讲到。可以尝试自己将自己的U盘转换为GPT分区表，然后将U盘格式化为fat32。或者看我们下一篇的教程**

\*\*\*\*

