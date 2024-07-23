---
title: "为 FC41D 添加 ramdump 和 trace32 离线分析调试方法"
date: 2024-07-23T02:00:18+08:00
lastmod: 2024-07-23T02:00:18+08:00
author: ["hacper"]
tags:
   - FC41D
   - freertos
   - bk7231m
   - Trace32
   - dump
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "增加 ramdump 离线调试方法，解 bug 更加得心应手。" # 文章简单描述，会展示在主页
# weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
autonumbering: true # 目录自动编号
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
searchHidden: false # 该页面可以被搜索到
showbreadcrumbs: true #顶部显示当前路径
# mermaid: true

---

想要使用 Trace32 Simulators 离线解析 ramdump，必须具备以下 3 个条件：
- 知道当前寄存器的值
- 能够获取到当前运行状态下 ram 的数据
- 能获取到当前使用的 elf 文件  


## 代码修改

修改代码，将 ram 、 tcm 的数据打印到log，按照 trace 32 的脚本格式，将寄存器数据打印到log。
新增一个 bk_dump_parser.py 脚本，从log中提取ram、tcm、寄存器数据，并写入到文件。
```diff
diff --git a/beken_freertos_sdk_release-SDK_3.0.21/beken378/app/config/sys_config_bk7231n.h b/beken_freertos_sdk_release-SDK_3.0.21/beken378/app/config/sys_config_bk7231n.h
index 601f3d63..bbbe8d4c 100644
--- a/beken_freertos_sdk_release-SDK_3.0.21/beken378/app/config/sys_config_bk7231n.h
+++ b/beken_freertos_sdk_release-SDK_3.0.21/beken378/app/config/sys_config_bk7231n.h
@@ -308,4 +308,6 @@
    #define CONFIG_MBEDTLS_SSL_OUT_CONTENT_LEN      4096
 #endif
 
+#define CFG_RAM_DUMP_ENABLE                               1
+
 #endif // _SYS_CONFIG_H_
diff --git a/beken_freertos_sdk_release-SDK_3.0.21/beken378/driver/intc/intc.c b/beken_freertos_sdk_release-SDK_3.0.21/beken378/driver/intc/intc.c
index bb17af8e..d069d95d 100644
--- a/beken_freertos_sdk_release-SDK_3.0.21/beken378/driver/intc/intc.c
+++ b/beken_freertos_sdk_release-SDK_3.0.21/beken378/driver/intc/intc.c
@@ -365,8 +365,112 @@ void bk_show_register (struct arm_registers *regs)
     }
     
     os_printf("\r\n");
+#if CFG_RAM_DUMP_ENABLE
+    os_printf("; CPU Registers:\r\n");
+    os_printf("reg_cmm_start:\r\n");
+    os_printf("register.set cpsr 0x%08x\r\n", regs->cpsr);
+    os_printf("register.set spsr 0x%08x\r\n", regs->spsr);
+
+    os_printf("register.set pc 0x%08x\r\n", regs->pc);
+
+    os_printf("register.set r0 0x%08x\r\n", regs->r0);
+    os_printf("register.set r1 0x%08x\r\n", regs->r1);
+    os_printf("register.set r2 0x%08x\r\n", regs->r2);
+    os_printf("register.set r3 0x%08x\r\n", regs->r3);
+    os_printf("register.set r4 0x%08x\r\n", regs->r4);
+    os_printf("register.set r5 0x%08x\r\n", regs->r5);
+    os_printf("register.set r6 0x%08x\r\n", regs->r6);
+    os_printf("register.set r7 0x%08x\r\n", regs->r7);
+    os_printf("register.set r8 0x%08x\r\n", regs->r8);
+    os_printf("register.set r9 0x%08x\r\n", regs->r8);
+    os_printf("register.set r10 0x%08x\r\n", regs->r8);
+    os_printf("register.set r11 0x%08x\r\n", regs->fp);
+    os_printf("register.set r12 0x%08x\r\n", regs->ip);
+
+    if((regs->spsr & 0x3f) == 0x20)
+    {
+        os_printf("register.set r13 0x%08x\r\n", regs->sp);
+        os_printf("register.set r14 0x%08x\r\n", regs->lr);
+    }
+    else if((regs->spsr & 0x3f) == 0x21)
+    {
+        reg1 = (const unsigned int *)0x400068;
+        os_printf("register.set r13 0x%08x\r\n", *(reg1+7));
+        os_printf("register.set r14 0x%08x\r\n", *(reg1+8));
+    }
+    else if((regs->spsr & 0x3f) == 0x22)
+    {
+        reg1 = (const unsigned int *)0x400044;
+        os_printf("register.set r13 0x%08x\r\n", *(reg1+7));
+        os_printf("register.set r14 0x%08x\r\n", *(reg1+8));
+    }
+    else if((regs->spsr & 0x3f) == 0x23)
+    {
+        reg1 = (const unsigned int *)0x4000d4;
+        os_printf("register.set r13 0x%08x\r\n", *(reg1+7));
+        os_printf("register.set r14 0x%08x\r\n", *(reg1+8));
+    }
+    else if((regs->spsr & 0x3f) == 0x27)
+    {
+        reg1 = (const unsigned int *)0x40008c;
+        os_printf("register.set r13 0x%08x\r\n", *(reg1+7));
+        os_printf("register.set r14 0x%08x\r\n", *(reg1+8));
+    }
+    else if((regs->spsr & 0x3f) == 0x33)
+    {
+        reg1 = (const unsigned int *)0x4000b0;
+        os_printf("register.set r13 0x%08x\r\n", *(reg1+7));
+        os_printf("register.set r14 0x%08x\r\n", *(reg1+8));
+    }
+    else if((regs->spsr & 0x3f) == 0x3f)
+    {
+        reg1 = (const unsigned int *)0x400024;
+        os_printf("register.set r13 0x%08x\r\n", *(reg1+6));
+        os_printf("register.set r14 0x%08x\r\n", *(reg1+7));
+    }
+    os_printf("\r\nreg_cmm_end:\r\n");
+    os_printf("\r\n");
+
+#endif
 
 }
+#if CFG_RAM_DUMP_ENABLE
+void bk_ram_dump(void)
+{
+    unsigned char *tcm_start = (unsigned char *)0x003F0000;
+    // unsigned char *itcm_start = (unsigned char *)0x003FF000;
+    unsigned char *ram_start = (unsigned char *)0x00400100;
+
+    bk_wdg_finalize();
+    os_printf("dump tcm:\r\n");
+    os_printf("tcm_start:\r\n");
+    for (size_t i = 0; i < (1024 *60); i++)
+    {
+        if(i%16 == 0)
+        {
+            os_printf("\r\n");
+        }
+        os_printf("%02x ", tcm_start[i]);
+        
+    }
+    os_printf("\r\ntcm_end:\r\n");
+    os_printf("\r\n");
+
+    os_printf("dump ram:\r\n");
+    os_printf("ram_start:\r\n");
+    for (size_t i = 0; i < (1024 *192-0x100); i++)
+    {
+        if(i%16 == 0)
+        {
+            os_printf("\r\n");
+        }
+        os_printf("%02x ", ram_start[i]);
+        
+    }
+    os_printf("\r\nram_end:\r\n");
+    os_printf("\r\n");
+}
+#endif
 
 void bk_trap_udef(struct arm_registers *regs)
 {
@@ -377,6 +481,9 @@ void bk_trap_udef(struct arm_registers *regs)
 #endif
     os_printf("undefined instruction\n");
     bk_show_register(regs);
+#if CFG_RAM_DUMP_ENABLE
+    bk_ram_dump();
+#endif
     bk_cpu_shutdown();
 }
 
@@ -389,6 +496,9 @@ void bk_trap_pabt(struct arm_registers *regs)
 #endif
     os_printf("prefetch abort\n");
     bk_show_register(regs);
+#if CFG_RAM_DUMP_ENABLE
+    bk_ram_dump();
+#endif
     bk_cpu_shutdown();
 }
 
@@ -401,6 +511,9 @@ void bk_trap_dabt(struct arm_registers *regs)
 #endif
     os_printf("data abort\n");
     bk_show_register(regs);
+#if CFG_RAM_DUMP_ENABLE
+    bk_ram_dump();
+#endif
     bk_cpu_shutdown();
 }
 
@@ -413,6 +526,9 @@ void bk_trap_resv(struct arm_registers *regs)
 #endif
     os_printf("not used\n");
     bk_show_register(regs);
+#if CFG_RAM_DUMP_ENABLE
+    bk_ram_dump();
+#endif
     bk_cpu_shutdown();
 }
 
diff --git a/beken_freertos_sdk_release-SDK_3.0.21/beken378/func/wlan_ui/wlan_cli.c b/beken_freertos_sdk_release-SDK_3.0.21/beken378/func/wlan_ui/wlan_cli.c
index c90de09e..50b3adb1 100644
--- a/beken_freertos_sdk_release-SDK_3.0.21/beken378/func/wlan_ui/wlan_cli.c
+++ b/beken_freertos_sdk_release-SDK_3.0.21/beken378/func/wlan_ui/wlan_cli.c
@@ -3306,6 +3306,12 @@ void pwr_Command(char *pcWriteBuffer, int xWriteBufferLen, int argc, char **argv
     rw_msg_set_power(0,pwr);
 }
 
+extern void do_undefined( void );
+
+void dump_test_Command(char *pcWriteBuffer, int xWriteBufferLen, int argc, char **argv)
+{
+    do_undefined();
+}
 static void Deep_Sleep_Command(char *pcWriteBuffer, int xWriteBufferLen, int argc, char **argv)
 {
 	PS_DEEP_CTRL_PARAM deep_sleep_param;
@@ -3974,6 +3980,9 @@ static const struct cli_command user_clis[] =
 #if (CFG_SOC_NAME == SOC_BK7221U)
     {"sec", "sec help", sec_Command },
 #endif
+#if CFG_RAM_DUMP_ENABLE
+    {"dump_test", "dump_test", dump_test_Command},
+#endif
 };
 
 
diff --git a/beken_freertos_sdk_release-SDK_3.0.21/quectel_build/scripts/bk_dump_parser.py b/beken_freertos_sdk_release-SDK_3.0.21/quectel_build/scripts/bk_dump_parser.py
new file mode 100644
index 00000000..f59289b4
--- /dev/null
+++ b/beken_freertos_sdk_release-SDK_3.0.21/quectel_build/scripts/bk_dump_parser.py
@@ -0,0 +1,91 @@
+#!/usr/bin/env python3
+
+import os
+import sys
+import optparse
+import binascii
+import re
+
+
+def parse_dump_log_file(filename):
+    reg_cmm = ""
+    tcm_data = ""
+    ram_data = ""
+    f = open(filename, "r")
+    while True:
+        line = f.readline()
+        if line == "":
+            break
+        if "reg_cmm_start:" in line:
+            while True:
+                line = f.readline()
+                if line  == "":
+                    break
+                if "reg_cmm_end:" in line:
+                    break
+                pattern = re.compile(r'register\.set [^\n]+')
+                matches = pattern.findall(line)
+                if len(matches) > 0:
+                    reg_cmm += matches[0] + "\r\n"
+        if "tcm_start:" in line:
+            while True:
+                line = f.readline()
+                if line  == "":
+                    break
+                if "tcm_end:" in line:
+                    break
+                
+                pattern = re.compile(r'(?:\[\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}\]\s*)?((?:[0-9a-fA-F]{2}\s*){16})')
+                matches = pattern.findall(line)
+                if len(matches) > 0:
+                    tcm_data += matches[0]
+                
+        if "ram_start:" in line:
+            while True:
+                line = f.readline()
+                if line  == "":
+                    break
+                if "ram_end:" in line:
+                    break
+                
+                pattern = re.compile(r'(?:\[\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}\]\s*)?((?:[0-9a-fA-F]{2}\s*){16})')
+                matches = pattern.findall(line)
+                if len(matches) > 0:
+                    ram_data += matches[0]
+            break
+    f.close()
+    return (reg_cmm, tcm_data, ram_data)
+
+def main(argv):
+    opt = optparse.OptionParser(usage="""usage %prog [options]
+    """)
+
+    opt.add_option("--log", action="store", dest="log",
+                   help="dump log file.")
+
+    opt, argv = opt.parse_args(argv)
+
+    if not opt.log:
+        print("No log file specified!")
+        return 1
+    reg_cmm, tcm_data, ram_data = parse_dump_log_file(opt.log)
+
+    if reg_cmm != "":
+        with open("reg.cmm", "w") as f:
+            f.write(reg_cmm)
+    if tcm_data != "":
+        tcm_data = tcm_data.replace(" ", "").replace("\n", "").replace("\r", "")
+        bin_data = binascii.unhexlify(tcm_data)
+        with open("tcm.bin", "wb") as f:
+            f.write(bin_data)
+
+    if ram_data != "":
+        ram_data = ram_data.replace(" ", "").replace("\n", "").replace("\r", "")
+        bin_data = binascii.unhexlify(ram_data)
+        with open("ram.bin", "wb") as f:
+            f.write(bin_data)
+
+
+
+if __name__ == '__main__':
+    sys.exit(main(sys.argv[1:]))
\ No newline at end of file
```

Trace32 Simulators 环境的搭建参考之前的文章：[Trace 32 离线 dump 分析环境搭建方法](https://hacperme.com/posts/series/dump/20230401_t32_setup/)，在这个基础上新增 Trace32 脚本，用于加载和解析 ramdump 文件。

```diff
diff --git a/T32_beken_7231.bat b/T32_beken_7231.bat
new file mode 100644
index 0000000..2b69bb0
--- /dev/null
+++ b/T32_beken_7231.bat
@@ -0,0 +1,5 @@
+@echo off
+
+echo Load Trace32
+
+start t32marm.exe -c config.t32, bk7231.cmm &
\ No newline at end of file
diff --git a/bk7231.cmm b/bk7231.cmm
new file mode 100644
index 0000000..4152765
--- /dev/null
+++ b/bk7231.cmm
@@ -0,0 +1,27 @@
+; ARM load script
+screen.on
+SYStem.RESet
+SYStem.CPU ARM968E
+SYStem.Up
+
+&psd=os.psd()
+&dump_dir="bkdump"
+&ap_elf_file="beken7231_bsp.elf"
+
+DATA.LOAD.ELF &dump_dir\&ap_elf_file /NOCLEAR
+
+;DATA.LOAD.BINARY &dump_dir\itcm.bin 0x003FF000 /NOCLEAR
+DATA.LOAD.BINARY &dump_dir\tcm.bin 0x003F0000 /NOCLEAR
+DATA.LOAD.BINARY &dump_dir\ram.bin 0x00400100 /NOCLEAR
+
+; CPU Registers:
+do &dump_dir\reg.cmm
+
+
+MENU.REPROGRAM &psd\freertos\freertos.men
+TASK.CONFIG &psd\freertos\freertos.t32
+
+B::Var.Frame /Locals /Caller
+B::Register
+B::Data.List
+
```

## 使用示例

dump log：
```
=~=~=~=~=~=~=~=~=~=~=~= MobaXterm log 2024.07.23 15:34:18 =~=~=~=~=~=~=~=~=~=~=~=
BK7231n_1.0.8
[2024-07-23 15:34:18]  CPSR:0x000000D3
[2024-07-23 15:34:18]  R0:0x00800000
[2024-07-23 15:34:18]  R1:0x00000000
[2024-07-23 15:34:18]  R2:0x00000003
[2024-07-23 15:34:18]  R3:0x000000A5
[2024-07-23 15:34:18]  R4:0x00401170
[2024-07-23 15:34:18]  R13:0x00401AD4
[2024-07-23 15:34:18]  R14(LR):0x000011D4
[2024-07-23 15:34:18]  ST:0x00000000
[2024-07-23 15:34:18]  [I/FAL] Fal(V0.4.0)success
[2024-07-23 15:34:18]  [I/OTA] RT-Thread OTA package(V0.2.4) initialize success.
[2024-07-23 15:34:18]  
[2024-07-23 15:34:18]  
[2024-07-23 15:34:18]  go os_addr(0x10000)..........
[2024-07-23 15:34:18]  j
[2024-07-23 15:34:18]  prvHeapInit-start addr:0x40aa90, size:152944
[2024-07-23 15:34:18]  [Flash]id:0xeb6015
[2024-07-23 15:34:18]  --write status reg:4004,2--
[2024-07-23 15:34:18]  [Flash]init over
... ...
[2024-07-23 15:34:22]  
[2024-07-23 15:34:22]  configuring interface uap (with Static IP)[THD]dhcp-server:[tcb]413e18 [stack]413988-413e08:1152:2
[2024-07-23 15:34:22]  def netif is no ap's netif, sending boardcast or no-subnet ip packets may failed
[2024-07-23 15:34:22]  init_xtal:0, delta:-1, last_xtal:0
[2024-07-23 15:34:22]  
[2024-07-23 15:34:25]  # dum
[2024-07-23 15:34:28]  [ATSVR]msg type:7
[2024-07-23 15:34:28]  p_test
[2024-07-23 15:34:31]  
[2024-07-23 15:34:31]  undefined instruction
[2024-07-23 15:34:31]  Current regs:
[2024-07-23 15:34:31]  r00:0x004157d0 r01:0x00000800 r02:0x00000001 r03:0x004157d0
[2024-07-23 15:34:31]  r04:0x0003c851 r05:0x003f03d8 r06:0x003f0398 r07:0x003f03d8
[2024-07-23 15:34:31]  r08:0x08080808 r09:0x09090909 r10:0x10101010
[2024-07-23 15:34:31]  fp :0x11111111 ip :0x00000070
[2024-07-23 15:34:31]  sp :0x004014f0 lr :0x000102c4 pc :0x000102c4
[2024-07-23 15:34:31]  SPSR:0x0000003f
[2024-07-23 15:34:31]  CPSR:0x0000009b
[2024-07-23 15:34:31]  
[2024-07-23 15:34:31]  separate regs:
[2024-07-23 15:34:31]  SYS:cpsr r8-r14
[2024-07-23 15:34:31]  0x0000009f
[2024-07-23 15:34:31]  0x08080808
[2024-07-23 15:34:31]  0x09090909
[2024-07-23 15:34:31]  0x10101010
[2024-07-23 15:34:31]  0x11111111
[2024-07-23 15:34:31]  0x00000070
[2024-07-23 15:34:31]  0x00416bc0
[2024-07-23 15:34:31]  0x0003c857
[2024-07-23 15:34:31]  IRQ:cpsr spsr r8-r14
[2024-07-23 15:34:31]  0x00000092
[2024-07-23 15:34:31]  0x6000003f
[2024-07-23 15:34:31]  0x08080808
[2024-07-23 15:34:31]  0x09090909
[2024-07-23 15:34:31]  0x10101010
[2024-07-23 15:34:31]  0x11111111
[2024-07-23 15:34:31]  0x00000070
[2024-07-23 15:34:31]  0x00402e28
[2024-07-23 15:34:31]  0x00054cde
[2024-07-23 15:34:31]  FIR:cpsr spsr r8-r14
[2024-07-23 15:34:31]  0x00000091
[2024-07-23 15:34:31]  0x6000009f
[2024-07-23 15:34:31]  0x00000000
[2024-07-23 15:34:31]  0x00000000
[2024-07-23 15:34:31]  0x00000000
[2024-07-23 15:34:31]  0x00000000
[2024-07-23 15:34:31]  0x00000008
[2024-07-23 15:34:31]  0x00401e38
[2024-07-23 15:34:31]  0x003fff34
[2024-07-23 15:34:31]  ABT:cpsr spsr r8-r14
[2024-07-23 15:34:31]  0x00000097
[2024-07-23 15:34:31]  0x00000010
[2024-07-23 15:34:31]  0x08080808
[2024-07-23 15:34:31]  0x09090909
[2024-07-23 15:34:31]  0x10101010
[2024-07-23 15:34:31]  0x11111111
[2024-07-23 15:34:31]  0x00000070
[2024-07-23 15:34:31]  0x00401538
[2024-07-23 15:34:31]  0xc7109030
[2024-07-23 15:34:31]  UND:cpsr spsr r8-r14
[2024-07-23 15:34:31]  0x0000009b
[2024-07-23 15:34:31]  0x0000003f
[2024-07-23 15:34:31]  0x08080808
[2024-07-23 15:34:31]  0x09090909
[2024-07-23 15:34:31]  0x10101010
[2024-07-23 15:34:31]  0x11111111
[2024-07-23 15:34:31]  0x00000070
[2024-07-23 15:34:31]  0x004014e8
[2024-07-23 15:34:31]  0x000102c4
[2024-07-23 15:34:31]  SVC:cpsr spsr r8-r14
[2024-07-23 15:34:31]  0x00000093
[2024-07-23 15:34:31]  0x6000001f
[2024-07-23 15:34:31]  0x08080808
[2024-07-23 15:34:31]  0x09090909
[2024-07-23 15:34:31]  0x10101010
[2024-07-23 15:34:31]  0x11111111
[2024-07-23 15:34:31]  0x00000070
[2024-07-23 15:34:31]  0x00403208
[2024-07-23 15:34:31]  0x0007e69c
[2024-07-23 15:34:31]  
[2024-07-23 15:34:31]  ; CPU Registers:
[2024-07-23 15:34:31]  reg_cmm_start:
[2024-07-23 15:34:31]  register.set cpsr 0x0000009b
[2024-07-23 15:34:31]  register.set spsr 0x0000003f
[2024-07-23 15:34:31]  register.set pc 0x000102c4
[2024-07-23 15:34:31]  register.set r0 0x004157d0
[2024-07-23 15:34:31]  register.set r1 0x00000800
[2024-07-23 15:34:31]  register.set r2 0x00000001
[2024-07-23 15:34:31]  register.set r3 0x004157d0
[2024-07-23 15:34:31]  register.set r4 0x0003c851
[2024-07-23 15:34:31]  register.set r5 0x003f03d8
[2024-07-23 15:34:31]  register.set r6 0x003f0398
[2024-07-23 15:34:31]  register.set r7 0x003f03d8
[2024-07-23 15:34:31]  register.set r8 0x08080808
[2024-07-23 15:34:31]  register.set r9 0x08080808
[2024-07-23 15:34:31]  register.set r10 0x08080808
[2024-07-23 15:34:31]  register.set r11 0x11111111
[2024-07-23 15:34:31]  register.set r12 0x00000070
[2024-07-23 15:34:31]  register.set r13 0x00416bc0
[2024-07-23 15:34:31]  register.set r14 0x0003c857
[2024-07-23 15:34:31]  
[2024-07-23 15:34:31]  reg_cmm_end:
[2024-07-23 15:34:31]  
[2024-07-23 15:34:31]  dump tcm:
[2024-07-23 15:34:31]  tcm_start:
[2024-07-23 15:34:31]  
[2024-07-23 15:34:31]  00 00 00 00 02 00 00 00 00 00 00 00 6a 84 28 05 
[2024-07-23 15:34:31]  89 c1 09 fe 00 88 a2 cf e4 23 00 00 59 96 c8 65 
[2024-07-23 15:34:31]  aa 30 c0 ba 44 df 7a 72 6b 82 26 52 8a 08 a8 28 
[2024-07-23 15:34:31]  7f 07 1e 00 ff dd 63 80 ce 8a 75 a9 aa 2e 78 86 
[2024-07-23 15:34:31]  39 03 f9 dd f0 bc 01 da 00 80 00 00 30 01 c0 d8 
[2024-07-23 15:34:31]  00 00 00 00 95 21 39 47 cc 5e 30 7b 90 90 88 88 
[2024-07-23 15:34:31]  90 90 90 90 98 a0 94 a8 a8 d0 b8 e0 b4 e8 b0 e8 
[2024-07-23 15:34:31]  ae e8 ae e8 ae e8 ad e7 ac e8 ac e7 a7 ff f8 10 
[2024-07-23 15:34:31]  00 00 00 00 05 00 01 00 00 80 00 00 02 00 00 00 
[2024-07-23 15:34:31]  00 00 00 00 18 00 00 00 51 01 00 00 00 00 00 00 
[2024-07-23 15:34:31]  00 00 00 00 09 00 00 00 00 00 00 f0 30 00 00 00 
[2024-07-23 15:34:31]  01 00 01 00 e0 00 01 00 70 00 01 00 01 00 01 00 
[2024-07-23 15:34:31]  05 00 01 00 39 03 f9 dd 02 00 00 00 2c 01 00 00 
[2024-07-23 15:34:31]  a7 ff f8 00 a7 ff f8 00 01 02 ba 00 95 00 00 00 
[2024-07-23 15:34:31]  00 00 00 00 08 01 57 03 00 02 00 00 00 08 00 18 
[2024-07-23 15:34:31]  00 20 c0 2c 01 00 00 00 2b 08 a9 07 ff 0f a0 0f 
[2024-07-23 15:34:31]  01 08 42 08 00 88 00 c4 fa 56 74 00 00 00 00 00 
[2024-07-23 15:34:31]  00 00 00 00 65 e0 b2 80 77 db 77 00 74 d8 74 00 
[2024-07-23 15:34:31]  64 00 00 80 00 00 00 00 28 36 00 00 00 00 00 00 
[2024-07-23 15:34:31]  00 00 00 00 00 00 00 00 00 00 00 00 ff 00 00 00 
[2024-07-23 15:34:31]  00 00 00 00 00 00 00 00 31 b1 b1 b1 b1 b1 b1 b1 
[2024-07-23 15:34:31]  b1 b1 b1 b1 31 b1 94 94 94 94 94 93 93 93 93 93 
[2024-07-23 15:34:31]  92 92 92 92 92 91 91 91 91 10 90 90 90 90 90 90 
[2024-07-23 15:34:31]  90 90 90 90 90 90 90 90 90 90 90 90 90 90 18 98 
[2024-07-23 15:34:31]  98 98 98 98 98 98 98 98 98 98 18 98 2d ad ad ad 
[2024-07-23 15:34:31]  ad ad ad ad ad ad ad ad 2d ad 00 00 0f 0d 0d 01 
... ...

[2024-07-23 15:34:48]  aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa 
[2024-07-23 15:34:48]  aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa 
[2024-07-23 15:34:48]  aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa 
[2024-07-23 15:34:48]  aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa 
[2024-07-23 15:34:48]  aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa 
[2024-07-23 15:34:48]  aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa 
[2024-07-23 15:34:48]  tcm_end:
[2024-07-23 15:34:48]  
[2024-07-23 15:34:48]  dump ram:
[2024-07-23 15:34:48]  ram_start:
[2024-07-23 15:34:48]  
[2024-07-23 15:34:48]  89 0f 07 00 51 0f 07 00 d0 07 00 00 00 10 00 00 
[2024-07-23 15:34:48]  c8 47 8c 42 00 48 00 c8 47 8c 42 00 49 00 00 00 
[2024-07-23 15:34:48]  00 08 00 00 00 00 00 00 00 00 00 00 79 f3 3f 00 
[2024-07-23 15:34:48]  4d f5 3f 00 71 f6 3f 00 c1 6c 01 00 1d 74 01 00 
... ...

[2024-07-23 15:35:41]  dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd 
[2024-07-23 15:35:41]  dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd 
[2024-07-23 15:35:41]  dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd 
[2024-07-23 15:35:41]  dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd 
[2024-07-23 15:35:41]  dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd 
[2024-07-23 15:35:41]  dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd 
[2024-07-23 15:35:41]  dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd 
[2024-07-23 15:35:41]  dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd 
[2024-07-23 15:35:41]  dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd 
[2024-07-23 15:35:41]  dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd 
[2024-07-23 15:35:41]  dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd 
[2024-07-23 15:35:41]  dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd dd 
[2024-07-23 15:35:41]  dd dd dd dd dd dd dd dd 00 00 00 00 00 00 00 00 
[2024-07-23 15:35:41]  ram_end:
[2024-07-23 15:35:41]  
[2024-07-23 15:35:41]  shutdown...
[2024-07-23 15:35:41]  
```

将 dumplog、elf 文件放到 bkdump 目录, 执行脚本 bk_dump_parser.py 解析 dumplog。

```bash
T32\bkdump on  master [?] via 🐍 v2.7.18
❯ py -3 .\bk_dump_parser.py --log .\dumplog.log
write reg.cmm
write tcm.bin
write ram.bin

T32\bkdump on  master [?] via 🐍 v2.7.18
❯ ls


    目录: C:\Users\x\Desktop\T32\bkdump


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         2024/7/23     15:33       10147160 beken7231_bsp.elf
-a----         2024/7/23     15:33        2372336 beken7231_bsp.map
-a----         2024/7/23     17:03           2922 bk_dump_parser.py
-a----         2024/7/23     15:35        1194721 dumplog.log
-a----         2024/7/23     19:35         196352 ram.bin
-a----         2024/7/23     19:35            531 reg.cmm
-a----         2024/7/23     19:35          61440 tcm.bin
```

点击 T32_beken_7231.bat 启动 Trace32
![](https://jsd.cdn.zzko.cn/gh/hacperme/picx-images-hosting@master/20240723/image.4916862rkk.webp)

查看任务调用栈
![](https://jsd.cdn.zzko.cn/gh/hacperme/picx-images-hosting@master/20240723/image.9nzoqlj1nz.webp)

查看任务列表
![](https://jsd.cdn.zzko.cn/gh/hacperme/picx-images-hosting@master/20240723/image.6m3spditri.webp)

切换任务上下文
![](https://jsd.cdn.zzko.cn/gh/hacperme/picx-images-hosting@master/20240723/image.26ldk47hmb.webp)