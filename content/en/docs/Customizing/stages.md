
---
title: "Stages"
linkTitle: "Stages"
weight: 1
date: 2017-01-05
description: >
  Configure the system in the various stages: boot, initramfs, fs, network, reconcile
---

Cloud-init files in `/system/oem`, `/oem` and `/usr/local/oem` are applied in 5 different phases: `boot`, `network`, `fs`, `initramfs` and `reconcile`. All the available cloud-init keywords can be used in each stage. Additionally, it's possible also to hook before or after a stage has run, each one has a specific stage which is possible to run steps: `boot.after`, `network.before`, `fs.after` etc.

Multiple stages can be specified in a single cloud-init file.

{{% alert title="Note" %}}
When a cOS derivative boots it creates sentinel files in order to allow to execute cloud-init steps programmaticaly.

- `/run/cos/recovery_mode` is being created when booting from the recovery partition
- `/run/cos/live_mode` is created when booting from the LiveCD

To execute a block using the sentinel files you can specify: `if: '[ -f "/run/cos/..." ]'`, see the examples below.
{{% /alert %}}

#### rootfs

This is the earliest stage, running before switching root, just right after the
root is mounted in `/sysroot` and before applying the immutable rootfs configuration.
This stage is executed over initrd root, no chroot is applied.

Example:
```yaml
name: "Set persistent devices"
stage:
  rootfs:
    - name: "Layout configuration"
      environment_file: /run/cos/cos-layout.env
      environment:
        VOLUMES: "LABEL=COS_OEM:/oem LABEL=COS_PERSISTENT:/usr/local"
        OVERLAY: "tmpfs:25%"
```

#### initramfs

This is still an early stage, running before switching root. Here you can apply radical changes to the booting setup of `cOS`.
Despite this is executed before switching root this exection runs chrooted into the target root after the immutable rootfs is set up and ready.

Example:
```yaml
name: "Run something on initramfs"
stages:
   initramfs:
     - name: "Setting"
       if: '[ ! -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in active or passive
          touch /etc/something_important
     - name: "Setting"
       if: '[ -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in recovery mode
```

#### boot

This stage is executed after initramfs has switched root, during the `systemd` bootup process.

Example:
```yaml
name: "Run something on boot"
stages:
   boot:
     - name: "Setting"
       if: '[ ! -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in active or passive
     - name: "Setting"
       if: '[ -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in recovery mode
```

#### fs

This stage is executed when fs is mounted and is guaranteed to have access to `COS_STATE` and `COS_PERSISTENT`.

Example:
```yaml
name: "Run something on boot"
stages:
   fs:
     - name: "Setting"
       if: '[ ! -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          touch /usr/local/something
     - name: "Setting"
       if: '[ -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in recovery mode
```


#### network

This stage is executed when network is available

Example:
```yaml
name: "Run something on boot"
stages:
   network:
     - name: "Setting"
       if: '[ ! -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Network is available, do something..
```

#### reconcile

This stage is executed `5m` after boot and periodically each `60m`.

Example:
```yaml
name: "Run something on boot"
stages:
   reconcile:
     - name: "Setting"
       if: '[ ! -f "/run/sentinel" ]'
       commands:
       - |
          touch /run/sentinel
```

#### after-install

This stage is executed after installation of the OS has ended (after calling `cos-install`).

Example:
```yaml
name: "Run something after installation"
stages:
   after-install:
     - name: "Setting"
       if: '[ ! -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in active or passive
     - name: "Setting"
       if: '[ -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in recovery mode
```


#### after-upgrade

This stage is executed after upgrade of the OS has ended (after calling `cos-upgrade`).

Example:
```yaml
name: "Run something after upgrade"
stages:
   after-upgrade:
     - name: "Setting"
       if: '[ ! -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in active or passive
     - name: "Setting"
       if: '[ -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in recovery mode
```

#### after-reset

This stage is executed after reset of the OS has ended (after calling `cos-reset`).

Example:
```yaml
name: "Run something after reset"
stages:
   after-reset:
     - name: "Setting"
       if: '[ ! -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in active or passive
     - name: "Setting"
       if: '[ -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in recovery mode
```

#### after-deploy

This stage is executed after deployment of the OS has ended (after calling `cos-deploy`).

Example:
```yaml
name: "Run something after deployment"
stages:
   after-deploy:
     - name: "Setting"
       if: '[ ! -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in active or passive
     - name: "Setting"
       if: '[ -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in recovery mode
```


#### before-install

This stage is executed before installation (executed during `cos-install`).

Example:
```yaml
name: "Run something before installation"
stages:
   before-install:
     - name: "Setting"
       if: '[ ! -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in active or passive
     - name: "Setting"
       if: '[ -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in recovery mode
```


#### before-upgrade

This stage is executed before upgrade of the OS (executed during `cos-upgrade`).

Example:
```yaml
name: "Run something before upgrade"
stages:
   before-upgrade:
     - name: "Setting"
       if: '[ ! -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in active or passive
     - name: "Setting"
       if: '[ -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in recovery mode
```

#### before-reset

This stage is executed before reset of the OS (executed during `cos-reset`).

Example:
```yaml
name: "Run something before reset"
stages:
   before-reset:
     - name: "Setting"
       if: '[ ! -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in active or passive
     - name: "Setting"
       if: '[ -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in recovery mode
```

#### before-deploy

This stage is executed before deployment of the OS has ended (executed during `cos-deploy`).

Example:
```yaml
name: "Run something before deployment"
stages:
   before-deploy:
     - name: "Setting"
       if: '[ ! -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in active or passive
     - name: "Setting"
       if: '[ -f "/run/cos/recovery_mode" ]'
       commands:
       - |
          # Run something when we are booting in recovery mode
```
