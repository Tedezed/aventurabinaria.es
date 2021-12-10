---
layout: post
title: "Install AMD drivers for Linux (Debian/Ubuntu)"
date: 2020-12-10 19:26:06 -0700
comments: true
---

Download and isntall `amdgpu-install`
```
curl -SL https://repo.radeon.com/amdgpu-install/21.40.1/ubuntu/focal/amdgpu-install_21.40.1.40501-1_all.deb -o amdgpu-install_21.40.1.40501-1_all.deb
sudo apt-get install ./amdgpu-install_21.40.1.40501-1_all.deb
sudo apt-get update
```

Execute `amdgpu-install`
```
amdgpu-install --usecase=graphics,opencl --opencl=rocr,legacy --vulkan=amdvlk,pro --no-32
sudo reboot
```