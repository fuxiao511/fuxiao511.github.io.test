---
layout: post
title:  handle_io
date:   2022-10-03 16:11:00 +0800
categories: linux
tags: [linux, cgroup]
typora-root-url: ../
---
* content
{:toc}

## kvm

```
handle_io
	kvm_fast_pio
		kvm_fast_pio_out
			emulator_pio_out  //拷贝数据到vcpu结构体中，执行kernel_pio

```

