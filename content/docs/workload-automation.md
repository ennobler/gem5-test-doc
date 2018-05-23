---
title: "Workload Automation"
date: 2018-05-13T18:31:30-04:00
draft: false
weight: 100
---

{{% notice warning %}}
It's not clear if this is still up-to-date
{{% /notice %}}

To run workloads in gem5, we advise you to use Workload Automation (WA).
This framework allows you to run workloads automatically on Android and
Linux platforms. More information can be found here:
<https://github.com/ARM-software/workload-automation>

## What do I need?

To use WA together with gem5 you will need to make changes to the host
system and the guest system. These changes will enable 9P over virtio,
which will allow for files to be transported into the simulation.

### Host system requirements

On the host system side this means that diod needs to be present. To
install diod on Ubuntu you can use the following command:

`sudo apt-get install diod`

### Guest system requirements

To enable this in the guest system we need to ensure the support is
built into the kernel. To do this the following configuration options
need to be set:

`CONFIG_NETWORK_FILESYSTEMS=y`
`CONFIG_NET_9P=y`
`CONFIG_NET_9P_VIRTIO=y`
`CONFIG_9P_FS=y`
`CONFIG_9P_FS_POSIX_ACL=y`
`CONFIG_9P_FS_SECURITY=y`
`CONFIG_VIRTIO_BLK=y`

The guest system also needs to have an m5 binary, which can be found in
the gem5 repository under util/m5. The m5 binary is used for extracting
files out of the simulation, checkpointing and simulation control (e.g.
stats dumps, and exiting gem5). It is important that this file can be
found on the path. For checkpointing to work correctly two things need
to be ensured when taking the checkpoint:

1.  The virtio device is included in the system
2.  No part of the host file system is mounted by the virtio device

During boot the operating system will initialize all the drivers
corresponding with which devices are present in the system. If the
virtio device is not found, it cannot be used later, thus the deivce
needs to be present during boot (1). Unfortunately, when checkpointing
the system it is impossible (well, very hard) to give guarantees about
the preservation of the state. You may add or remove files from the host
system in between taking and resuming from the checkpoint. For this
reason we advise to only mount the host system after restoring from the
checkpoint (2).

## Using gem5 with Workload Automation

WA uses agendas to specify the experiments that need to be run. An
agenda is a YAML file that describes the configuration of the device,
the workloads to be run, and which results to extract. It can be seen as
a recipe of how to recreate an experiment. Here is an example of what an
agenda looks like for gem5:

```
config:
    device: gem5_linux
    device_config:
        gem5_args: "configs/example/fs.py"
        gem5_vio_args: "--workload-automation-vio={}"
        username: root
        temp_dir: "/tmp"
        checkpoint: True
        run_delay: 10
    reboot_policy: never
    result_processors: [~sqlite]
    instrumentation: [~cpufreq]

workloads:
  - id: dhrystone
    workload_name: dhrystone
    iterations: 1
```

### Required YAML entries for gem5

  - gem5_args
    Specify the simulation to be run. This should be the same as you
    would specify for a stand-alone gem5 run.
  - gem5_vio_args
    This parameter enables the virtio device in the simulated system. As
    a minimal requirement you need to ensure that the root parameter of
    the virtio device is exposed to the command line. Please set this to
    ‘{}’ in the agenda as it will later be set to the used directory by
    WA.

### Optional YAML entries

  - gem5_binary
    Specify which gem5 to execute. This option is useful for executing
    gem5 in a non-standard location, or in debug mode.
  - temp_dir
    Temporary directory used for file transferring into gem5. This
    directory will be created and removed by WA.
  - checkpoint
    When this is set to ‘True’, WA creates a checkpoint once the system
    is booted. This checkpoint can then later be used by WA to avoid
    boot time.
  - run_delay
    This parameter sets the time that the system should sleep prior to
    running workloads or taking checkpoints.
  - username
    This is a Linux-specific parameter, as most Linux systems require a
    login. If a login prompt is found, this field will be used as
    username.
  - password
    As above, but for passwords

### Patch for the gem5 simulator

The patch below is required to complete the integration with gem5. It
adds a VirtIO device to the default full-system simulation script, and
exposes it via an argument to the
script.

```patch
diff --git a/configs/common/Options.py b/configs/common/Options.py
--- a/configs/common/Options.py
+++ b/configs/common/Options.py
@@ -351,3 +351,8 @@
     parser.add_option("--command-line-file", action="store",
                       default=None, type="string",
                       help="File with a template for the kernel command line")
+
+    # Workload Automation options
+    parser.add_option("--workload-automation-vio", action="store", type="string",
+                      default=None, help="Enable the Virtio 9P device and set "
+                      "the path to use. Required to use Workload Automation")
diff --git a/configs/example/fs.py b/configs/example/fs.py
--- a/configs/example/fs.py
+++ b/configs/example/fs.py
@@ -225,6 +225,25 @@
             not options.fast_forward:
             CpuConfig.config_etrace(TestCPUClass, test_sys.cpu, options)

+        if buildEnv['TARGET_ISA'] != "arm" and options.workload_automation_vio:
+            warn("Ignoring --workload-automation-vio. It is unsupported on "
+                 "non-ARM systems.")
+        else:
+            from m5.objects import PciVirtIO, VirtIO9PDiod
+            viopci = PciVirtIO(pci_bus=0, pci_dev=test_sys.realview._num_pci_dev,
+                               pci_func=0, InterruptPin=1,
+                               InterruptLine=test_sys.realview._num_pci_int_line)
+
+            test_sys.realview._num_pci_dev = test_sys.realview._num_pci_dev + 1
+            test_sys.realview._num_pci_int_line = test_sys.realview._num_pci_int_line + 1
+
+            viopci.vio = VirtIO9PDiod()
+            viopci.vio.root = options.workload_automation_vio
+
+            test_sys.realview.viopci = viopci
+            test_sys.realview.viopci.dma = test_sys.iobus.slave
+            test_sys.realview.viopci.pio = test_sys.iobus.master
+
         CacheConfig.config_cache(options, test_sys)

         MemConfig.config_mem(options, test_sys)
diff --git a/src/dev/arm/RealView.py b/src/dev/arm/RealView.py
--- a/src/dev/arm/RealView.py
+++ b/src/dev/arm/RealView.py
@@ -270,6 +270,8 @@
     cxx_header = "dev/arm/realview.hh"
     system = Param.System(Parent.any, "system")
     _mem_regions = [(Addr(0), Addr('256MB'))]
+    _num_pci_dev = 0
+    _num_pci_int_line = 0

     def _on_chip_devices(self):
         return []
@@ -615,10 +617,18 @@

     # Attach any PCI devices that are supported
     def attachPciDevices(self):
-        self.ethernet = IGbE_e1000(pci_bus=0, pci_dev=0, pci_func=0,
-                                   InterruptLine=1, InterruptPin=1)
-        self.ide = IdeController(disks = [], pci_bus=0, pci_dev=1, pci_func=0,
-                                 InterruptLine=2, InterruptPin=2)
+        self.ethernet = IGbE_e1000(pci_bus=0, pci_dev=self._num_pci_dev,
+                                   pci_func=0,
+                                   InterruptLine=self._num_pci_int_line,
+                                   InterruptPin=1)
+        self._num_pci_dev = self._num_pci_dev + 1
+        self._num_pci_int_line = self._num_pci_int_line + 1
+        self.ide = IdeController(disks = [], pci_bus=0,
+                                 pci_dev=self._num_pci_dev, pci_func=0,
+                                 InterruptLine=self._num_pci_int_line,
+                                 InterruptPin=2)
+        self._num_pci_dev = self._num_pci_dev + 1
+        self._num_pci_int_line = self._num_pci_int_line + 1

     def enableMSIX(self):
         self.gic = Pl390(dist_addr=0x2C001000, cpu_addr=0x2C002000, it_lines=512)
```
