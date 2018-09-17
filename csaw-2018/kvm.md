## kvm 500 (reversing)

### Solution

First decompile the program, and we'll see it open `/dev/kvm` and issue several `ioctl`s.
![](https://i.imgur.com/TTOCoLu.png)

Then use `strace` to inspect these calls.
```
$ strace -v ./challenge
openat(AT_FDCWD, "/dev/kvm", O_RDWR)    = 3
ioctl(3, KVM_GET_API_VERSION, 0)        = 12
ioctl(3, KVM_CREATE_VM, 0)              = 4
ioctl(4, KVM_SET_TSS_ADDR, 0xfffbd000)  = 0
mmap(NULL, 2097152, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS|MAP_NORESERVE, -1, 0) = 0x7fd00ef77000
madvise(0x7fd00ef77000, 2097152, MADV_MERGEABLE) = 0
ioctl(4, KVM_SET_USER_MEMORY_REGION, {slot=0, flags=0, guest_phys_addr=0, memory_size=2097152, userspace_addr=0x7fd00ef77000}) = 0
ioctl(4, KVM_CREATE_VCPU, 0)            = 5
ioctl(3, KVM_GET_VCPU_MMAP_SIZE, 0)     = 12288
mmap(NULL, 12288, PROT_READ|PROT_WRITE, MAP_SHARED, 5, 0) = 0x7fd00f37a000
ioctl(5, KVM_GET_SREGS, {cs={base=0xffff0000, limit=65535, selector=61440, type=11, present=1, dpl=0, db=0, s=1, l=0, g=0, avl=0}, ds={base=0, limit=65535, selector=0, type=3, present=1, dpl=0, db=0, s=1, l=0, g=0, avl=0}, es={base=0, limit=65535, selector=0, type=3, present=1, dpl=0, db=0, s=1, l=0, g=0, avl=0}, fs={base=0, limit=65535, selector=0, type=3, present=1, dpl=0, db=0, s=1, l=0, g=0, avl=0}, gs={base=0, limit=65535, selector=0, type=3, present=1, dpl=0, db=0, s=1, l=0, g=0, avl=0}, ss={base=0, limit=65535, selector=0, type=3, present=1, dpl=0, db=0, s=1, l=0, g=0, avl=0}, tr={base=0, limit=65535, selector=0, type=11, present=1, dpl=0, db=0, s=0, l=0, g=0, avl=0}, ldt={base=0, limit=65535, selector=0, type=2, present=1, dpl=0, db=0, s=0, l=0, g=0, avl=0}, gdt={base=0, limit=65535}, idt={base=0, limit=65535}, cr0=1610612752, cr2=0, cr3=0, cr4=0, cr8=0, efer=0, apic_base=0xfee00900, interrupt_bitmap=[0, 0, 0, 0]}) = 0
ioctl(5, KVM_SET_SREGS, {cs={base=0, limit=4294967295, selector=8, type=11, present=1, dpl=0, db=0, s=1, l=1, g=1, avl=0}, ds={base=0, limit=4294967295, selector=16, type=3, present=1, dpl=0, db=0, s=1, l=1, g=1, avl=0}, es={base=0, limit=4294967295, selector=16, type=3, present=1, dpl=0, db=0, s=1, l=1, g=1, avl=0}, fs={base=0, limit=4294967295, selector=16, type=3, present=1, dpl=0, db=0, s=1, l=1, g=1, avl=0}, gs={base=0, limit=4294967295, selector=16, type=3, present=1, dpl=0, db=0, s=1, l=1, g=1, avl=0}, ss={base=0, limit=4294967295, selector=16, type=3, present=1, dpl=0, db=0, s=1, l=1, g=1, avl=0}, tr={base=0, limit=65535, selector=0, type=11, present=1, dpl=0, db=0, s=0, l=0, g=0, avl=0}, ldt={base=0, limit=65535, selector=0, type=2, present=1, dpl=0, db=0, s=0, l=0, g=0, avl=0}, gdt={base=0, limit=65535}, idt={base=0, limit=65535}, cr0=2147811379, cr2=0, cr3=8192, cr4=32, cr8=0, efer=1280, apic_base=0xfee00900, interrupt_bitmap=[0, 0, 0, 0]}) = 0
ioctl(5, KVM_SET_REGS, {rax=0, rbx=0, rcx=0, rdx=0, rsi=0, rdi=0, rsp=0x200000, rbp=0, r8=0, r9=0, r10=0, r11=0, r12=0, r13=0, r14=0, r15=0, rip=0, rflags=0x2}) = 0
ioctl(5, KVM_RUN, 0)                    = 0
fstat(0, {st_dev=makedev(0, 26), st_ino=4, st_mode=S_IFCHR|0620, st_nlink=1, st_uid=1000, st_gid=5, st_blksize=1024, st_blocks=0, st_rdev=makedev(136, 1), st_atime=1537036280 /* 2018-09-16T02:31:20.548336166+0800 */, st_atime_nsec=548336166, st_mtime=1537036280 /* 2018-09-16T02:31:20.548336166+0800 */, st_mtime_nsec=548336166, st_ctime=1537021307 /* 2018-09-15T22:21:47.548336166+0800 */, st_ctime_nsec=548336166}) = 0
brk(NULL)                               = 0x55d7d8728000
brk(0x55d7d8749000)                     = 0x55d7d8749000
read(0
```

Let's first check the [KVM API](https://www.kernel.org/doc/Documentation/virtual/kvm/api.txt) and understand what it's doing:
1.  Open kvm, get fd = 3

`openat(AT_FDCWD, "/dev/kvm", O_RDWR)    = 3`
   
2.  check whether the kvm api version equals 12

`ioctl(3, KVM_GET_API_VERSION, 0)        = 12`
   
3.  creat a VM，fd = 4

`ioctl(3, KVM_CREATE_VM, 0)              = 4`
   
4.  configure the physical address of VM

`ioctl(4, KVM_SET_TSS_ADDR, 0xfffbd000)  = 0`
   
5.  allocate the memory needed by VM and set the mapping
```
mmap(NULL, 2097152, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS|MAP_NORESERVE, -1, 0) = 0x7f1cebda8000
madvise(0x7f1cebda8000, 2097152, MADV_MERGEABLE) = 0
ioctl(4, KVM_SET_USER_MEMORY_REGION, {slot=0, flags=0, guest_phys_addr=0, memory_size=2097152, userspace_addr=0x7f1cebda8000}) = 0
```

6.  create vcpu，fd = 5

`ioctl(4, KVM_CREATE_VCPU, 0)            = 5`

7.  allocate a shared memory space neede by vcpu
```
ioctl(3, KVM_GET_VCPU_MMAP_SIZE, 0)     = 12288
mmap(NULL, 12288, PROT_READ|PROT_WRITE, MAP_SHARED, 5, 0) = 0x7f1cec1ab000
```

8.  initialize special register
```
ioctl(5, KVM_SET_SREGS, {cs={base=0, limit=4294967295, selector=8, type=11, present=1, dpl=0, db=0, s=1, l=1, g=1, avl=0}, ds={base=0, limit=4294967295, selector=16, type=3, present=1, dpl=0, db=0, s=1, l=1, g=1, avl=0}, es={base=0, limit=4294967295, selector=16, type=3, present=1, dpl=0, db=0, s=1, l=1, g=1, avl=0}, fs={base=0, limit=4294967295, selector=16, type=3, present=1, dpl=0, db=0, s=1, l=1, g=1, avl=0}, gs={base=0, limit=4294967295, selector=16, type=3, present=1, dpl=0, db=0, s=1, l=1, g=1, avl=0}, ss={base=0, limit=4294967295, selector=16, type=3, present=1, dpl=0, db=0, s=1, l=1, g=1, avl=0}, tr={base=0, limit=65535, selector=0, type=11, present=1, dpl=0, db=0, s=0, l=0, g=0, avl=0}, ldt={base=0, limit=65535, selector=0, type=2, present=1, dpl=0, db=0, s=0, l=0, g=0, avl=0}, gdt={base=0, limit=65535}, idt={base=0, limit=65535}, cr0=2147811379, cr2=0, cr3=8192, cr4=32, cr8=0, efer=1280, apic_base=0xfee00900, interrupt_bitmap=[0, 0, 0, 0]}) = 0
```

9.  initialize gprs
```
ioctl(5, KVM_SET_REGS, {rax=0, rbx=0, rcx=0, rdx=0, rsi=0, rdi=0, rsp=0x200000, rbp=0, r8=0, r9=0, r10=0, r11=0, r12=0, r13=0, r14=0, r15=0, rip=0, rflags=0x2}) = 0
```
10. start running VM

`ioctl(5, KVM_RUN, 0) = 0`

After the VM starts, the program will wait for some input, a byte at once, and after each read it will call `ioctl(5, KVM_RUN, 0)` again.

Try to use frace to trace kvm execute `challenge` and enter 1234 : 
```
# echo 1 > /sys/kernel/debug/tracing/events/kvm/enable
# cat /sys/kernel/debug/tracing/trace_pipe
       challenge-24797 [003] .... 83184.290314: kvm_fpu: load
       challenge-24797 [003] .... 83184.290322: kvm_pio: pio_read at 0xe9 size 1 count 1 val 0x31
       challenge-24797 [003] d..1 83184.290326: kvm_entry: vcpu 0
       challenge-24797 [003] .... 83184.290332: kvm_exit: reason EXTERNAL_INTERRUPT rip 0x100 info 0 800000f6
       challenge-24797 [003] d..1 83184.290334: kvm_entry: vcpu 0
       challenge-24797 [003] .... 83184.290337: kvm_exit: reason IO_INSTRUCTION rip 0xff info e90008 0
       challenge-24797 [003] .... 83184.290339: kvm_fpu: unload
       challenge-24797 [003] .... 83184.290341: kvm_userspace_exit: reason KVM_EXIT_IO (2)
       challenge-24797 [003] .... 83184.290380: kvm_fpu: load
       challenge-24797 [003] .... 83184.290381: kvm_pio: pio_read at 0xe9 size 1 count 1 val 0x32
       challenge-24797 [003] d..1 83184.290382: kvm_entry: vcpu 0
       challenge-24797 [003] .... 83184.290384: kvm_exit: reason IO_INSTRUCTION rip 0xff info e90008 0
       challenge-24797 [003] .... 83184.290385: kvm_fpu: unload
       challenge-24797 [003] .... 83184.290386: kvm_userspace_exit: reason KVM_EXIT_IO (2)
       challenge-24797 [003] .... 83184.290413: kvm_fpu: load
       challenge-24797 [003] .... 83184.290413: kvm_pio: pio_read at 0xe9 size 1 count 1 val 0x33
       challenge-24797 [003] d..1 83184.290414: kvm_entry: vcpu 0
       challenge-24797 [003] .... 83184.290416: kvm_exit
```
KVM has called pio\_read() to read the bytes we just inputted, then EXIT because of io event, then RUN again by `challenge`.

Take a look at the value of `CS`, we know the VM starts its executution from address `0x0`
```c
cs={base=0,
    limit=4294967295,
    selector=8,
    type=11,
    present=1,
    dpl=0, db=0, s=1, l=1, g=1, avl=0
}
```

By `ltrace`ing the program, I found some data is copied to the guess address `0x0.`
```
$ ltrace ./challenge
memcpy(0x7f3f247a9000, "UH\211\345H\201\354\020(\0\0H\215\205\360\327\377\377\276\0(\0\0H\211\307\350\262\0\0\0\307"..., 4888) = 0x7f3f247a9000
```
These data is located at 0x202174, length = 4888
![](https://i.imgur.com/iXFqfN5.png)

Extract these bytes from the binary and disassemble it.

After some analysis, we can easily identify the functionality of some functions

![](https://i.imgur.com/B8sVigW.png)

![](https://i.imgur.com/dmzwqBO.png)

![](https://i.imgur.com/aRSj1cS.png)

![](https://i.imgur.com/mdHUQOJ.png)

But there are two strange functions left, `0x172` and `0x1e0`.

Starts from entry, it will read `0x2800` bytes inside the function `0xd1`, using instruction `in` which will trigger `pio_read()`, then do some stuff inside function `0x1e0`, a byte at once, then finally compare two strings located at `0x580` and `0x1320` with length `0x54a`, and output `Correct!` or `"Wrong!"`.

Let's dive into the function `0x1e0`:
![](https://i.imgur.com/cRBBWUI.png)

What the heck? The program soon halted after enter the function, how does it return to the main loop?

I gave input of 0x2800 bytes to the program, and grab the log from the ftrace, it shows:
```
       challenge-21643 [000] d..1 89662.281943: kvm_entry: vcpu 0
       challenge-21643 [000] .... 89662.281945: kvm_exit: reason HLT rip 0x201 info 0 0
       challenge-21643 [000] .... 89662.281954: kvm_fpu: unload
       challenge-21643 [000] .... 89662.281955: kvm_userspace_exit: reason KVM_EXIT_HLT (5)
       challenge-21643 [000] .... 89662.281959: kvm_fpu: load
       challenge-21643 [000] d..1 89662.281960: kvm_entry: vcpu 0
       challenge-21643 [000] .... 89662.281961: kvm_exit: reason HLT rip 0x341 info 0 0
       challenge-21643 [000] .... 89662.281962: kvm_fpu: unload
       challenge-21643 [000] .... 89662.281962: kvm_userspace_exit: reason KVM_EXIT_HLT (5)
       challenge-21643 [000] .... 89662.281964: kvm_fpu: load
```
It looks like after the guess halted, the program will change its rip and restart again. Let's go back to the binary

![](https://i.imgur.com/2QvF73a.png)

That's where the works are done, if the KVM exit reason is `KVM_EXIT_HTL`, it will perform some works on the register and rerun the VM, but the decompiler can't properly analyze the operation.

Again, using strace, I can inspect all calls with their arguments:
```
$ strace -vo log ./challenge < input
$ less log
...
ioctl(5, KVM_RUN, 0)                    = 0
ioctl(5, KVM_GET_REGS, {rax=0x3493310d, rbx=0, rcx=0x1fffe7, rdx=0xe9, rsi=0x1300, rdi=0xffffffff, rsp=0x1fd778, rbp=0x1fd7d8, r8=0, r9=0, r10=0, r11=0, r12=0, r13=0, r14=0, r15=0, rip=0x202, rflags=0x6}) = 0
ioctl(5, KVM_SET_REGS, {rax=0x3493310d, rbx=0, rcx=0x1fffe7, rdx=0xe9, rsi=0x1300, rdi=0xffffffff, rsp=0x1fd778, rbp=0x1fd7d8, r8=0, r9=0, r10=0, r11=0, r12=0, r13=0, r14=0, r15=0, rip=0x32c, rflags=0x6}) = 0
ioctl(5, KVM_RUN, 0)                    = 0
ioctl(5, KVM_GET_REGS, {rax=0x5de72dd, rbx=0, rcx=0x5de72dd, rdx=0xffffffff, rsi=0x1300, rdi=0xffffffff, rsp=0x1fd778, rbp=0x1fd7d8, r8=0, r9=0, r10=0, r11=0, r12=0, r13=0, r14=0, r15=0, rip=0x342, rflags=0x46}) = 0
ioctl(5, KVM_SET_REGS, {rax=0x5de72dd, rbx=0, rcx=0x5de72dd, rdx=0xffffffff, rsi=0x1300, rdi=0xffffffff, rsp=0x1fd778, rbp=0x1fd7d8, r8=0, r9=0, r10=0, r11=0, r12=0, r13=0, r14=0, r15=0, rip=0x347, rflags=0x46}) = 0
ioctl(5, KVM_RUN, 0)                    = 0
...
ioctl(5, KVM_GET_REGS, {rax=0x968630d0, rbx=0, rcx=0x5de72dd, rdx=0x30, rsi=0xae0, rdi=0x30, rsp=0x1fd628, rbp=0x1fd688, r8=0, r9=0, r10=0, r11=0, r12=0, r13=0, r14=0, r15=0, rip=0x342, rflags=0x13}) = 0
ioctl(5, KVM_SET_REGS, {rax=0x968630d0, rbx=0, rcx=0x5de72dd, rdx=0x30, rsi=0xae0, rdi=0x30, rsp=0x1fd628, rbp=0x1fd688, r8=0, r9=0, r10=0, r11=0, r12=0, r13=0, r14=0, r15=0, rip=0x400, rflags=0x13}) = 0
ioctl(5, KVM_RUN, 0)                    = 0
ioctl(5, KVM_GET_REGS, {rax=0xef5bdd13, rbx=0, rcx=0x8f6e2804, rdx=0xae0, rsi=0x30, rdi=0x61, rsp=0x1fd628, rbp=0x1fd688, r8=0, r9=0, r10=0, r11=0, r12=0, r13=0, r14=0, r15=0, rip=0x41d, rflags=0x97}) = 0
ioctl(5, KVM_SET_REGS, {rax=0xef5bdd13, rbx=0, rcx=0x8f6e2804, rdx=0xae0, rsi=0x30, rdi=0x61, rsp=0x1fd628, rbp=0x1fd688, r8=0, r9=0, r10=0, r11=0, r12=0, r13=0, r14=0, r15=0, rip=0x435, rflags=0x97}) = 0
ioctl(5, KVM_RUN, 0)                    = 0
ioctl(5, KVM_GET_REGS, {rax=0x5f291a64, rbx=0, rcx=0x8f6e2804, rdx=0xae0, rsi=0x30, rdi=0x61, rsp=0x1fd628, rbp=0x1fd688, r8=0, r9=0, r10=0, r11=0, r12=0, r13=0, r14=0, r15=0, rip=0x43b, rflags=0x97}) = 0
ioctl(5, KVM_SET_REGS, {rax=0x5f291a64, rbx=0, rcx=0x8f6e2804, rdx=0xae0, rsi=0x30, rdi=0x61, rsp=0x1fd628, rbp=0x1fd688, r8=0, r9=0, r10=0, r11=0, r12=0, r13=0, r14=0, r15=0, rip=0x441, rflags=0x97}) = 0
ioctl(5, KVM_RUN, 0)                    = 0
ioctl(5, KVM_GET_REGS, {rax=0xc50b6060, rbx=0, rcx=0x8f6e2804, rdx=0xae0, rsi=0x30, rdi=0x61, rsp=0x1fd628, rbp=0x1fd688, r8=0, r9=0, r10=0, r11=0, r12=0, r13=0, r14=0, r15=0, rip=0x44e, rflags=0x97}) = 0
ioctl(5, KVM_SET_REGS, {rax=0xc50b6060, rbx=0, rcx=0x8f6e2804, rdx=0xae0, rsi=0x30, rdi=0x61, rsp=0x1fd628, rbp=0x1fd688, r8=0, r9=0, r10=0, r11=0, r12=0, r13=0, r14=0, r15=0, rip=0x454, rflags=0x97}) = 0
ioctl(5, KVM_RUN, 0)                    = 0
```
Now, it's clear that the program will only change the rip according to the value of rax.
The change is performed by checking a table in the binary, and it's stored at `0x2020A0`

![](https://i.imgur.com/VMWCtL4.png)
![](https://i.imgur.com/V1gS61R.png)

The mapping is:

```
rax        =>  rip
0x3493310d => 0x32c
0x5de72dd  => 0x347
0x968630d0 => 0x400
0xef5bdd13 => 0x435
0x5f291a64 => 0x441
0xc50b6060 => 0x454
0x8aeef509 => 0x389
0x9d1fe433 => 0x3ed
0x54a15b03 => 0x376
0x8f6e2804 => 0x422
0x59c33d0f => 0x3e1
0x64d8a529 => 0x3b8
0x5de72dd  => 0x347
0xfc2ff49f => 0x3ce
```

After the `nop` slide under the first `hlt` inside `0x1e0`, there is more code with `hlt`:
![](https://i.imgur.com/itMwHDG.png)

We can now manually chain the blocks together using the table, and finally use our brain to decompile it:

```c
int8_t *rbp, *rsp;
int64_t rdi, rsi, rdx, rcx, rax, rbx;
int8_t *ebp, *esp;
int32_t edi, esi, edx, ecx, eax, ebx;

uint32_t shft = 0;
uint8_t *cmpbuf_p = (uint8_t *)0x1320;

void func172(uint32_t arg) {
  *cmpbuf_p |= (uint8_t) (arg << shft);
  shft += 1;
  if (shft == 8) {
    shft = 0;
  }
  if (shft == 0) {
    cmpbuf_p += 1;
  }
}

int func1e0(char chr, uint8_t *addr) {
  if (*(int32_t *)addr == -1) { // 0x347
    if (func1e0(chr,
                (uint8_t *)(*(uint64_t *)(addr + 8)))
        == 1) { // 0x376
      func172(0);
      return 1;
    }
    if (func1e0(chr,
                (uint8_t *)(*(uint64_t *)(addr + 16)))
        == 1) { // 0x3b8
      func172(1);
      return 1;
    }
    return 0; // 0x3ce
  } // 0x400
  if (*(char *)(addr) == chr) { // 0x422
    return 1;
  }
  return 0; // 0x435
}
```

Now the things are much clearer, `func1e0` is always called with `addr = 0x1300` at the outmost loop, then it will check if the addr conatins byte `0xff`, if true, then recursively check `addr+8` and `addr+16`.

Look at offset 0x1300, it is actually some kind of address chaining, which finally link to a character.
```
$ od -t x8 -Ax dump | less
...
000ae0 0000000000000030 0000000000000000
000af0 0000000000000000 0000000000000000
...
000c60 00000000000000ff 0000000000000ae0
000c70 0000000000000c40 0000000000000000
...
001260 0000000000000020 0000000000000000
001270 0000000000000000 0000000000000000
001280 00000000000000ff 0000000000001240
001290 0000000000001260 0000000000000000
0012a0 00000000000000ff 0000000000001160
0012b0 0000000000001280 0000000000000000
0012c0 00000000000000ff 0000000000000fc0
0012d0 00000000000012a0 0000000000000000
0012e0 00000000000000ff 0000000000000c60
0012f0 00000000000012c0 0000000000000000
001300 00000000000000ff 00000000000012e0
001310 0000000000003b30
```

For example, if we keep followed `addr + 8` we'll get `0x1300 -> 0x12e0 -> 0xc60 -> 0xae0 -> 0x30`

Slightly modify the function make it read the memory then dump the chracters in a tree:
```
0x1300 ->
        0x12e0 ->
                0xc60 ->
                        0xae0 -> 30(0)
                        0xc40 ->
                                0xbc0 ->
                                        0xb80 ->
                                                0xb00 -> 77(w)
                                                0xb60 ->
                                                        0xb20 -> 7d(})
                                                        0xb40 -> 35(5)
                                        0xba0 -> 6f(o)
                                0xc20 ->
                                        0xbe0 -> 31(1)
                                        0xc00 -> 74(t)
                0x12c0 ->
                        0xfc0 ->
                                0xd40 ->
                                        0xcc0 ->
                                                0xc80 -> 33(3)
                                                0xca0 -> 37(7)
                                        0xd20 ->
                                                0xce0 -> 66(f)
                                                0xd00 -> 69(i)
                                0xfa0 ->
                                        0xe20 ->
                                                0xda0 ->
                                                        0xd60 -> 3f(?)
                                                        0xd80 -> 63(c)
                                                0xe00 ->
                                                        0xdc0 -> 62(b)
                                                        0xde0 -> 64(d)
                                        0xf80 ->
                                                0xe80 ->
                                                        0xe40 -> 67(g)
                                                        0xe60 -> 72(r)
                                                0xf60 ->
                                                        0xee0 ->
                                                                0xea0 -> a(\n)
                                                                0xec0 -> 2e(.)
                                                        0xf40 ->
                                                                0xf00 -> 32(2)
                                                                0xf20 -> 65(e)
                        0x12a0 ->
                                0x1160 ->
                                        0x10e0 ->
                                                0x10a0 ->
                                                        0x1020 ->
                                                                0xfe0 -> 6d(m)
                                                                0x1000 -> 6e(n)
                                                        0x1080 ->
                                                                0x1040 -> 78(x)
                                                                0x1060 -> 7b({)
                                                0x10c0 -> 61(a)
                                        0x1140 ->
                                                0x1100 -> 73(s)
                                                0x1120 -> 36(6)
                                0x1280 ->
                                        0x1240 ->
                                                0x11c0 ->
                                                        0x1180 -> 34(4)
                                                        0x11a0 -> 68(h)
                                                0x1220 ->
                                                        0x11e0 -> 6c(l)
                                        0x1260 -> 20( )
        0x3b30 -> 0()
```

There are apparently some chracters we want!! `f`, `l`, `a`, `g`, `{`, `}`.

Each time the function returns with 1, it will call `func172()`. `func172()` will shift some bytes from `0x1320`, which is the buffer later will be compared with `0x580`. Take a deeper look at this function, if `arg == 0` then nothing changes, but if `arg == 1`, the `shft`-th bit from right is set at the current byte, once `shft` reaches 8, it's wrapped to 0 and move to next byte.

So the input chrarcter will determine what bits are set, isn't it a huffman encoding?
We can thus modify our function again to print out the encoding table, each time it recurse with `addr+8` is a `0` and `addr+16` is a `1`, remember the bit is set after the recusive call returns, so the code must be reversed.

```
{
  '0' : '000',
  'w' : '000100',
  '}' : '0100100',
  '5' : '1100100',
  'o' : '10100',
  '1' : '01100',
  't' : '11100',
  '3' : '000010',
  '7' : '100010',
  'f' : '010010',
  'i' : '110010',
  '?' : '0001010',
  'c' : '1001010',
  'b' : '0101010',
  'd' : '1101010',
  'g' : '0011010',
  'r' : '1011010',
  '.' : '10111010',
  '2' : '01111010',
  'e' : '11111010',
  'm' : '00000110',
  'n' : '10000110',
  'x' : '01000110',
  '{' : '11000110',
  'a' : '100110',
  's' : '010110',
  '6' : '110110',
  '4' : '0001110',
  'h' : '1001110',
  'l' : '0101110',
  'u' : '1101110',
  ' ' : '11110',
  '' : '1',
}
```

Extracted the bytes from 0x580 with length 0x54a, translate to binary string and decode with the table. Unfortunately, since the codes are reversed, the property that codes don't have common prefix now isn't hold anymore. I get something like
```
flag.txt 000660o0017i000017i0000000000711000600t766n1602 0 tuar   oshi oshiefo{11fd 0culd 0ao1  100 ctf tea012s 1 obfuscat0s boi0o0 011101011111111
```
It's almost there, but I can't figure out which character with common prefix to choose, finally, I try to manually replace the binary string based on some known words, like `flag{`, `ctf`, `tea`, `obfuscat`, etc. And I finally get the flag
```
flag.txt11111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111000000000000
1101101101100001110100000000001100100010110010000010000000000110010001011001000001000000000000000000000000000100
0100110010110000001000001000011101101100000100001110000100010110110110110100001100011001101100000111101011111000
0111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111011100101
1011100100110101101011110111101111001010001011010011101100101111111111111111111111111111110010100010110100111011
0010111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111
1111111111111111111111111111111111111111111111111111111111111111111111111111flag{who would win?  100 ctf teams o
r 1 obfuscat3d boi?}0011101011111111
```

Here is the script I use for decryption:
```python
#!/usr/bin/env python3

table = {
    '0': '000',
    'w': '000100',
    '}': '0100100',
    '5': '1100100',
    'o': '10100',
    '1': '01100',
    't': '11100',
    '3': '000010',
    '7': '100010',
    'f': '010010',
    'i': '110010',
    '?': '0001010',
    'c': '1001010',
    'b': '0101010',
    'd': '1101010',
    'g': '0011010',
    'r': '1011010',
#    '\n': '00111010',
    '.': '10111010',
    '2': '01111010',
    'e': '11111010',
    'm': '00000110',
    'n': '10000110',
    'x': '01000110',
    '{': '11000110',
    'a': '100110',
    's': '010110',
    '6': '110110',
    '4': '0001110',
    'h': '1001110',
    'l': '0101110',
    'u': '1101110',
    ' ': '11110',
#    '': '1',
}

data = None
flag = ""
with open("nativecode", "rb") as f:
    f.seek(0x580)
    data = f.read(0x54a)

data = data[:146]

binstr = ""
for b in data:
    binstr += bin(b)[2:].rjust(8, '0')[::-1]
"""
for i in range(2000):
    matched = []
    for char, code in table.items():
        if binstr.startswith(code):
            matched.append((code, char))
    if len(matched) == 0:
        binstr = binstr[1:]
        continue
    code, char = min(matched, key=lambda t: t[0])
    binstr = binstr.replace(code, '', 1)
    flag += char
    print(flag, binstr)
"""


def rep(a, b):
    s = ""
    for c in b:
        s += table[c]
    return a.replace(s, b)


#print(binstr)
binstr = rep(binstr, "flag.txt")
binstr = rep(binstr, "flag{")
binstr = rep(binstr, "would ")
binstr = rep(binstr, "who ")
binstr = rep(binstr, "or ")
binstr = rep(binstr, "win?  ")
binstr = rep(binstr, "obfuscat3d ")
binstr = rep(binstr, " ctf ")
binstr = rep(binstr, "teams ")
binstr = rep(binstr, "1 ")
binstr = rep(binstr, "100")
binstr = rep(binstr, "boi?")
binstr = rep(binstr, "}")
print(binstr)
```


### Flag
```
flag{who would win?  100 ctf teams or 1 obfuscat3d boi?}
```
