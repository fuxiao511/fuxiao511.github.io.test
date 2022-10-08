## kvm

```
handle_io
	kvm_fast_pio
		kvm_fast_pio_out
			emulator_pio_out  //拷贝数据到vcpu结构体中，执行kernel_pio

```

