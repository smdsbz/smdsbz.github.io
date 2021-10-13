---
layout: article
title: SSH into a Vagrant virtual machine on Windows
key: ssh-into-a-vagrant-virtual-machine-on-windows
tags: SSH Vagrant VirtualMachine Windows
---

<!-- more -->

After creating the vm with `vagrant up`, `vagrant ssh` should bring you into
the vm.

However, if you'd like to use the default OpenSSH client shipped with Windows,
you may encounter `Permission denied` error. This is not caused by an incorrect
private key, but rather a file access permission that is _too open_ (you may
check this by passing `-vvv` when calling `ssh`).

To address this issue, simply navigate to `.vagrant/machines/<vm name>/<provider>/private_key`,
and remove role "Everyone" from `Properties > Security > Group or User Name` (
`属性 > 安全 > 组或用户名`). Now your default SSH client (and therefore VS Code
Remote-SSH) will work!
