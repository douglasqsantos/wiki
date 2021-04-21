# Resetting the Root Password from the Boot Loader on CentOS/RedHat 8

One task that every system administrator should be able to accomplish is resetting a lost root password. If the administrator is still logged in, either as an unprivileged user but with full sudo access, or as root, this task is trivial. When the administrator is not logged in, this task becomes slightly more involved.

Several methods exist to set a new root password. A system administrator could, for example, boot the system using a Live CD, mount the root file system from there, and edit /etc/shadow. In this section, we explore a method that does not require the use of external media.

**NOTE:** On Red Hat Enterprise Linux 6 and earlier, administrators can boot the system into runlevel 1 to get a root prompt. The closest analogs to runlevel 1 on a Red Hat Enterprise Linux 8 machine are the rescue and emergency targets, both of which require the root password to log in.

On Red Hat Enterprise Linux 8, it is possible to have the scripts that run from the initramfs pause at certain points, provide a root shell, and then continue when that shell exits. This is mostly meant for debugging, but you can also use this method to reset a lost root password.

To access that root shell, follow these steps:

- Reboot the system.
- Interrupt the boot loader countdown by pressing any key, except **Enter**.
- Move the cursor to the kernel entry to boot.
- Press e to edit the selected entry.
- Move the cursor to the kernel command line (the line that starts with Linux).
- Append rd.break. With that option, the system breaks just before the system hands control from the initramfs to the actual system.
- Press Ctrl+x to boot with the changes.

At this point, the system presents a root shell, with the actual root file system on the disk mounted read-only on /sysroot. Because troubleshooting often requires modification to the root file system, you need to change the root file system to read/write. The following step shows how the **remount,rw** option to the mount command remounts the file system with the new option (rw) set.

**NOTE:** *Prebuilt images may place multiple console= arguments to the kernel to support a wide array of implementation scenarios. Those console= arguments indicate the devices to use for console output. The caveat with rd.break is that even though the system sends the kernel messages to all the consoles, the prompt ultimately uses whichever console is given last. If you do not get your prompt, you may want to temporarily reorder the console= arguments when you edit the kernel command line from the boot loader.*

**IMPORTANT**: *The system has not yet enabled SELinux, so any file you create does not have an SELinux context. Some tools, such as the **passwd** command, first create a temporary file, then move it in place of the file they are intended to edit, effectively creating a new file without an SELinux context. For this reason, when you use the **passwd** command with rd.break, the /etc/shadow file does not get an SELinux context.*

To reset the root password from this point, use the following procedure:

1. Remount /sysroot as read/write.

```bash
mount -o remount,rw /sysroot
```

2. Switch into a chroot jail, where /sysroot is treated as the root of the file-system tree.

```bash
chroot /sysroot
```

3. Set a new root password.

```bash
passwd root
```

4. Make sure that all unlabeled files, including /etc/shadow at this point, get relabeled during boot.

```bash
touch /.autorelabel
```

5. Type **exit** twice. The first command exits the **chroot** jail, and the second command exits the **initramfs** debug shell.

At this point, the system continues booting, performs a full SELinux relabel, and then reboots again.
