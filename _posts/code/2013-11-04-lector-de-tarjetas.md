---
layout: post
title:  Lector de tarjetas
excerpt: Recauchutando el kelmer para hacer funcionar el lector de SD del ThinkPad
date:   2013-11-04
categories: code
tags: [linux, kernel]
---
Debido a un problema -posiblemente relacionado con la tarjeta de video, aunque ya ni me acuerdo- que me dejaba las X cuajadas, un día probé a instalar un kernel más moderno que el que viene de serie con Debian 7. Para mi sorpresa, el problema desapareció y, desde entonces, tengo la costumbre de ir actualizando el kernel en cuanto sale.

Pero, un buen día, al tratar de utilizar el lector de tarjetas de mi ThinkPad, observé con natural desagrado que no funcionaba. Se trata de este dispositivo:

```shell
$ lspci | grep Card
04:00.0 Unassigned class [ff00]: Realtek Semiconductor Co., Ltd. RTS5209 PCI Express Card Reader (rev 01)
```

Tras un poco de investigación, resulta que el módulo que lo soporta (rts_pstor) no está incluido en el kernel. Afortunadamente, el fabricante (Realtek) pone a disposición un driver GPL en su sitio web: [realtek.com]

Todo iba muy bien hasta que, a partir de determinada versión (¿3.4?), ocurre esto:

```shell
luis@e330:~/src/rts_pstor$ make
sed "s/RTSX_MK_TIME/`date +%y.%m.%d.%H.%M`/" timestamp.in &gt; timestamp.h
cp -f ./define.release ./define.h
make -C /lib/modules/3.12.0/build/ SUBDIRS=/home/luis/src/rts_pstor modules
make[1]: Entering directory `/home/luis/src/linux-3.12'
CC [M] /home/luis/src/rts_pstor/rtsx.o
/home/luis/src/rts_pstor/rtsx.c:275:2: error: unknown field ‘proc_info’ specified in initializer
/home/luis/src/rts_pstor/rtsx.c:275:2: warning: initialization from incompatible pointer type [enabled by default]
/home/luis/src/rts_pstor/rtsx.c:275:2: warning: (near initialization for ‘rtsx_host_template.proc_dir’) [enabled by default]
/home/luis/src/rts_pstor/rtsx.c:916:22: error: expected ‘=’, ‘,’, ‘;’, ‘asm’ or ‘__attribute__’ before ‘rtsx_probe’
/home/luis/src/rts_pstor/rtsx.c:1080:23: error: expected ‘=’, ‘,’, ‘;’, ‘asm’ or ‘__attribute__’ before ‘rtsx_remove’
/home/luis/src/rts_pstor/rtsx.c:1106:11: error: ‘rtsx_probe’ undeclared here (not in a function)
/home/luis/src/rts_pstor/rtsx.c:1107:2: error: implicit declaration of function ‘__devexit_p’ [-Werror=implicit-function-declaration]
/home/luis/src/rts_pstor/rtsx.c:1107:24: error: ‘rtsx_remove’ undeclared here (not in a function)
/home/luis/src/rts_pstor/rtsx.c:485:12: warning: ‘rtsx_control_thread’ defined but not used [-Wunused-function]
/home/luis/src/rts_pstor/rtsx.c:596:12: warning: ‘rtsx_polling_thread’ defined but not used [-Wunused-function]
/home/luis/src/rts_pstor/rtsx.c:745:13: warning: ‘quiesce_and_remove_host’ defined but not used [-Wunused-function]
/home/luis/src/rts_pstor/rtsx.c:780:13: warning: ‘release_everything’ defined but not used [-Wunused-function]
/home/luis/src/rts_pstor/rtsx.c:790:12: warning: ‘rtsx_scan_thread’ defined but not used [-Wunused-function]
/home/luis/src/rts_pstor/rtsx.c:816:13: warning: ‘rtsx_init_options’ defined but not used [-Wunused-function]
cc1: some warnings being treated as errors
make[2]: *** [/home/luis/src/rts_pstor/rtsx.o] Error 1
make[1]: *** [_module_/home/luis/src/rts_pstor] Error 2
make[1]: Leaving directory `/home/luis/src/linux-3.12'
make: *** [default] Error 2
```

El interfaz proc_info está obsoleto y ha sido eliminado. Este es el parche que hay que aplicar, eliminando las referencias al interfaz:

```shell
--- rtsx.c      2011-01-11 10:11:07.000000000 +0100
+++ ../ok_rts_pstor/rtsx.c      2013-08-19 10:29:18.639278988 +0200
@@ -138,38 +138,6 @@ static int slave_configure(struct scsi_d
#define SPRINTF(args...) \
        do { if (pos &lt; buffer+length) pos += sprintf(pos, ## args); } while (0)   -static int proc_info (struct Scsi_Host *host, char *buffer, -               char **start, off_t offset, int length, int inout) -{ -       char *pos = buffer; - -        -       if (inout) -               return length; - -        -       SPRINTF("   Host scsi%d: %s\n", host-&gt;host_no, CR_DRIVER_NAME);
-
-
-       SPRINTF("       Vendor: Realtek Corp.\n");
-       SPRINTF("      Product: PCIE Card Reader\n");
-       SPRINTF("      Version: %s\n", DRIVER_VERSION);
-       SPRINTF("        Build: %s\n", DRIVER_MAKE_TIME);
-
-       /*
-        * Calculate start of next buffer, and return value.
-        */
-       *start = buffer + offset;
-
-       if ((pos - buffer) &lt; offset)
-               return (0);
-       else if ((pos - buffer - offset) &lt; length) -               return (pos - buffer - offset); -       else -               return (length); -} - -    static int queuecommand_lck(struct scsi_cmnd *srb,                         void (*done)(struct scsi_cmnd *)) @@ -272,7 +240,7 @@ struct scsi_host_template rtsx_host_temp                  .name =                         CR_DRIVER_NAME,         .proc_name =                    CR_DRIVER_NAME, -       .proc_info =                    proc_info, +       /* .proc_info =                 proc_info, */         .info =                         host_info,            @@ -913,7 +881,7 @@ static void rtsx_init_options(struct rts         chip-&gt;s3_pwr_off_delay = 1000;
}
 
-static int __devinit rtsx_probe(struct pci_dev *pci, const struct pci_device_id *pci_id)
+static int  rtsx_probe(struct pci_dev *pci, const struct pci_device_id *pci_id)
{
        struct Scsi_Host *host;
        struct rtsx_dev *dev;
@@ -1077,7 +1045,7 @@ errout:
}
 
-static void __devexit rtsx_remove(struct pci_dev *pci)
+static void  rtsx_remove(struct pci_dev *pci)
{
        struct rtsx_dev *dev = (struct rtsx_dev *)pci_get_drvdata(pci);
 
@@ -1104,7 +1072,7 @@ static struct pci_driver driver = {
        .name = CR_DRIVER_NAME,
        .id_table = rtsx_ids,
        .probe = rtsx_probe,
-       .remove = __devexit_p(rtsx_remove),
+       .remove = (rtsx_remove),
#ifdef CONFIG_PM
        .suspend = rtsx_suspend,
        .resume = rtsx_resume,
```

A partir de ahí, el procedimiento habitual:

```shell
$ make clean
$ make
$ sudo make install
$ sudo depmod -a
```

Y listo:

```shell
$ /sbin/modinfo rts_pstor
filename: /lib/modules/3.12.0/kernel/drivers/scsi/rts_pstor.ko
version: v1.10
license: GPL
description: Realtek PCI-Express card reader driver
srcversion: BCAF3F6C893DC8A26AD19FB
alias: pci:v000010ECd00005288sv*sd*bcFFsc*i*
alias: pci:v000010ECd00005209sv*sd*bcFFsc*i*
alias: pci:v000010ECd00005208sv*sd*bcFFsc*i*
depends: scsi_mod
vermagic: 3.12.0 SMP mod_unload modversions
parm: delay_use:seconds to delay before using a new device (uint)
parm: ss_en:enable selective suspend (int)
parm: ss_interval:Interval to enter ss state in seconds (int)
parm: auto_delink_en:enable auto delink (int)
parm: aspm_l0s_l1_en:enable device aspm (byte)
parm: msi_en:enable msi (int)
```

[realtek.com]: http://www.realtek.com/downloads/downloadsView.aspx?Langid=1&PNid=15&PFid=25&Level=4&Conn=3&DownTypeID=3&GetDown=false
