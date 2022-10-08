**kvm_vcpu_fops** 对应vcpu fd，在create_vcpu_fd设置


    static const struct file_operations kvm_vcpu_fops = { 
            .release        = kvm_vcpu_release,
            .unlocked_ioctl = kvm_vcpu_ioctl,
            .mmap           = kvm_vcpu_mmap,
            .llseek         = noop_llseek,
            KVM_COMPAT(kvm_vcpu_compat_ioctl),
    };

对vcpu执行KVM_RUN后，kvm_arch_vcpu_ioctl_run检查各种参数，然后执行vcpu_run

    kvm_vcpu_ioctl
    	KVM_RUN：
    	kvm_arch_vcpu_ioctl_run(vcpu);
    		vcpu->arch.complete_userspace_io
    		kvm_x86_vcpu_pre_run
    		vcpu_run





	vcpu_run
		for(;;) {
	        if (kvm_vcpu_running(vcpu)) {
	            r = vcpu_enter_guest(vcpu);   //r>0继续执行死循环
	        } else {
	            r = vcpu_block(vcpu);
	        }
	
			if (r <= 0)
				break;
	    }



**vcpu_enter_guest**是核心函数

	vcpu_enter_guest
		for(;;) {
			//执行run函数，即vmx_vcpu_run，加载寄存器，进入non_root模式
			//在vmx_vcpu_run退出前，执行快速函数vmx_exit_handlers_fastpath
			exit_fastpath = static_call(kvm_x86_vcpu_run)(vcpu); 
			
			//如果快速函数返回值不让重新进入guest，则break
	        if (likely(exit_fastpath != EXIT_FASTPATH_REENTER_GUEST))
	        	break;
	
			if (kvm_lapic_enabled(vcpu))
				static_call_cond(kvm_x86_sync_pir_to_irr)(vcpu);
	
			if (unlikely(kvm_vcpu_exit_request(vcpu))) {
				exit_fastpath = EXIT_FASTPATH_EXIT_HANDLED;
				break;
			}
	
		}
		static_call(kvm_x86_handle_exit_irqoff)(vcpu);
		
		//执行vm_exit的服务程序，即vmx_handle_exit
		r = static_call(kvm_x86_handle_exit)(vcpu, exit_fastpath);  



快速函数

```
static fastpath_t vmx_exit_handlers_fastpath(struct kvm_vcpu *vcpu)
{
        switch (to_vmx(vcpu)->exit_reason.basic) {
        //快速函数只处理MSR_WRITE和PREEMPTION_TIMER两种情况
        case EXIT_REASON_MSR_WRITE:
                return handle_fastpath_set_msr_irqoff(vcpu);
        case EXIT_REASON_PREEMPTION_TIMER:
                return handle_fastpath_preemption_timer(vcpu);
        default:
                return EXIT_FASTPATH_NONE;
        }
}
返回值有三种类型
enum exit_fastpath_completion {
        EXIT_FASTPATH_NONE,				//fast函数未处理
        EXIT_FASTPATH_REENTER_GUEST,	//fast函数处理后，可以立即重新进入guest
        EXIT_FASTPATH_EXIT_HANDLED,		//fast函数处理后，kvm会经历全部runloop，但不会执行exit_handler
};

```





## vmx_handle_exit



    vmx_handle_exit
    	__vmx_handle_exit
    		//flush pml buffer, 导出脏数据
    		vmx_flush_pml_buffer
    		//根据不同的EXIT_REASON调用真正的handler，如handle_io，handle_invlpg
    		return kvm_vmx_exit_handlers[exit_handler_index](vcpu)
    	







## exit_reason

        vcpu->run->exit_reason = reason
    	对应结构体
    	struct kvm_vcpu——struct kvm_run——_u32 exit_reason 用于告知用户态退出原因，在很多地方进行赋值
        
        struct vcpu_vmx *vmx = to_vmx(vcpu);
        union vmx_exit_reason exit_reason = vmx->exit_reason;
        对应结构体：
        struct kvm_vcpu——struct vcpu_vmx——union vmx_exit_reason  用于kvm内部使用
        
        在vmx->exit_reason.full = vmcs_read32(VM_EXIT_REASON);中赋值





























qemu_mutex_lock_iothread