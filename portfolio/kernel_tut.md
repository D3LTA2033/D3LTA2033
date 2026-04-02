# Comprehensive Kernel Architecture and Implementation Tutorial

## Table of Contents
1. [Introduction to Operating System Kernels](#introduction)
2. [Kernel Architecture Overview](#architecture)
3. [Process Management](#process)
4. [Memory Management](#memory)
5. [File Systems](#filesystem)
6. [Device Drivers](#drivers)
7. [Networking Stack](#networking)
8. [Security and Permissions](#security)
9. [Inter-Process Communication](#ipc)
10. [Kernel Synchronization](#sync)
11. [System Calls Interface](#syscalls)
12. [Implementation Examples](#examples)

---

## Introduction to Operating System Kernels {#introduction}

### What is a Kernel?

The kernel is the core component of an operating system that bridges the gap between hardware and software. It manages system resources, provides services to applications, and enforces security policies. The kernel runs in privileged mode (ring 0 in x86 architecture) with direct access to all hardware resources.

### Kernel Responsibilities

1. **Process Management**: Creating, scheduling, and terminating processes
2. **Memory Management**: Virtual memory, paging, and physical memory allocation
3. **Device Management**: Abstracting hardware through device drivers
4. **File System Management**: Providing file abstractions and storage management
5. **Network Stack**: Implementing TCP/IP and other network protocols
6. **Security**: Enforcing access controls and permissions
7. **System Calls**: Providing interface for user-space applications

### Kernel Types

#### Monolithic Kernels
- All kernel services run in kernel space
- Examples: Linux, Windows (hybrid), BSD
- Pros: High performance, direct function calls
- Cons: Less modular, stability concerns

#### Microkernels
- Minimal kernel with only essential services
- Other services run in user space
- Examples: Minix, QNX, L4
- Pros: More modular, better isolation
- Cons: Performance overhead from message passing

#### Hybrid Kernels
- Combination of monolithic and microkernel approaches
- Examples: Windows NT, XNU (macOS)
- Balance between performance and modularity

---

## Kernel Architecture Overview {#architecture}

### Kernel Space vs User Space

The processor operates in different privilege levels:

```
┌─────────────────────────────────────────┐
│              User Space                  │ ← Ring 3 (Unprivileged)
│  ┌─────────────────────────────────────┐ │
│  │        Applications                 │ │
│  └─────────────────────────────────────┘ │
│  ┌─────────────────────────────────────┐ │
│  │      System Libraries               │ │
│  └─────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│              Kernel Space                │ ← Ring 0 (Privileged)
│  ┌─────────────────────────────────────┐ │
│  │         System Calls                │ │
│  └─────────────────────────────────────┘ │
│  ┌─────────────────────────────────────┐ │
│  │      Core Kernel Services            │ │
│  └─────────────────────────────────────┘ │
│  ┌─────────────────────────────────────┐ │
│  │        Device Drivers               │ │
│  └─────────────────────────────────────┘ │
│  ┌─────────────────────────────────────┐ │
│  │         Hardware                    │ │
│  └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

### Boot Process

1. **BIOS/UEFI Initialization**
   - Hardware detection and initialization
   - Load bootloader from storage device

2. **Bootloader Execution**
   - Load kernel image into memory
   - Setup initial execution environment
   - Jump to kernel entry point

3. **Kernel Initialization**
   - Initialize core subsystems
   - Setup interrupt handlers
   - Initialize memory management
   - Mount root filesystem
   - Start init process (PID 1)

### Kernel Data Structures

#### Task Control Block (TCB)
```c
struct task_struct {
    // Process identification
    pid_t pid;
    pid_t tgid;
    uid_t uid;
    gid_t gid;
    
    // Scheduling information
    int prio;
    int static_prio;
    int normal_prio;
    struct sched_entity se;
    
    // Memory management
    struct mm_struct *mm;
    struct mm_struct *active_mm;
    
    // File system information
    struct fs_struct *fs;
    struct files_struct *files;
    
    // Signal handling
    struct signal_struct *signal;
    struct sighand_struct *sighand;
    
    // Execution state
    unsigned long flags;
    int exit_state;
    int exit_code;
    
    // CPU state
    struct thread_info thread_info;
    struct pt_regs *regs;
    
    // List management
    struct list_head tasks;
    struct list_head run_list;
};
```

---

## Process Management {#process}

### Process States

```
┌───────────┐    fork()    ┌───────────┐
│   NEW     │ ──────────→ │   READY   │
└───────────┘              └───────────┘
     │                        │
     │                        ▼ scheduler
     │                   ┌───────────┐
     │                   │  RUNNING   │
     │                   └───────────┘
     │                        │
     │                        ▼
     │                   ┌───────────┐
     └────────────────── │  BLOCKED   │
     (I/O wait)         └───────────┘
                            │
                            ▼
                        ┌───────────┐
                        │ TERMINATED│
                        └───────────┘
```

### Process Creation (fork())

1. **Allocate Process Descriptor**
   - Create new `task_struct`
   - Initialize basic fields
   - Assign PID

2. **Copy Memory Space**
   - Create new page table
   - Copy-on-write for efficiency
   - Setup virtual memory areas

3. **Initialize Resources**
   - Copy file descriptors
   - Inherit signal handlers
   - Setup execution context

4. **Add to Scheduler**
   - Set process state to READY
   - Add to run queue
   - Wake up scheduler

### Process Scheduling

#### Scheduling Algorithms

**Completely Fair Scheduler (CFS)**
- Virtual runtime based scheduling
- Red-black tree for ordered tasks
- O(log n) insertion and removal

```c
struct sched_entity {
    struct load_weight load;
    unsigned long run_weight;
    u64 exec_start;
    u64 sum_exec_runtime;
    u64 vruntime;
    u64 prev_sum_exec_runtime;
    
    // RB-tree node
    struct rb_node run_node;
    struct list_head group_node;
};
```

**Real-time Scheduling**
- FIFO and Round Robin policies
- Fixed priorities (0-99)
- Preemptive scheduling

#### Context Switch

```assembly
; x86_64 context switch assembly
switch_to:
    ; Save current process state
    pushq   %rbp
    movq    %rsp, %rbp
    pushq   %r15
    pushq   %r14
    pushq   %r13
    pushq   %r12
    pushq   %rbx
    
    ; Save old stack pointer
    movq    %rsp, (old_thread)
    
    ; Load new stack pointer
    movq    (new_thread), %rsp
    
    ; Restore new process state
    popq    %rbx
    popq    %r12
    popq    %r13
    popq    %r14
    popq    %r15
    popq    %rbp
    ret
```

---

## Memory Management {#memory}

### Virtual Memory Architecture

```
┌─────────────────────────────────────────┐
│           Virtual Address Space         │
├─────────────────────────────────────────┤
│  ┌─────────────────────────────────────┐ │
│  │         Kernel Space                │ │ ← High addresses
│  │  ┌─────────────────────────────────┐ │ │
│  │  │      Kernel Code & Data         │ │ │
│  │  └─────────────────────────────────┘ │ │
│  └─────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│  ┌─────────────────────────────────────┐ │
│  │         User Space                   │ │
│  │  ┌─────────────────────────────────┐ │ │
│  │  │         Stack                    │ │ │
│  │  └─────────────────────────────────┘ │ │
│  │  ┌─────────────────────────────────┐ │ │
│  │  │         Heap                    │ │ │
│  │  └─────────────────────────────────┘ │ │
│  │  ┌─────────────────────────────────┐ │ │
│  │  │       BSS/Data/Text             │ │ │
│  │  └─────────────────────────────────┘ │ │
│  └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

### Page Tables

#### Multi-level Page Table Structure

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│ PML4E   │───▶│ PDPTE   │───▶│ PDE     │───▶│ PTE     │
│ (512)   │    │ (512)   │    │ (512)   │    │ (512)   │
└─────────┘    └─────────┘    └─────────┘    └─────────┘
     │              │              │              │
     ▼              ▼              ▼              ▼
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│512 GB   │    │  1 GB   │    │  2 MB   │    │  4 KB   │
│Region   │    │ Region  │    │ Region  │    │ Page    │
└─────────┘    └─────────┘    └─────────┘    └─────────┘
```

#### Page Table Entry Format (x86_64)

```c
typedef struct {
    uint64_t present : 1;        // Page is present in memory
    uint64_t rw : 1;             // Read/write permission
    uint64_t user : 1;           // User/supervisor mode
    uint64_t pwt : 1;            // Page-level write-through
    uint64_t pcd : 1;            // Page-level cache disable
    uint64_t accessed : 1;       // Page was accessed
    uint64_t dirty : 1;          // Page was written to
    uint64_t pat : 1;            // Page attribute table
    uint64_t global : 1;         // Global page (TLB)
    uint64_t avail : 3;          // Available for software use
    uint64_t frame : 40;         // Physical frame address
    uint64_t avail2 : 11;        // More available bits
    uint64_t nx : 1;             // No-execute bit
} pte_t;
```

### Memory Allocation

#### Buddy System Allocator

```c
struct zone {
    // Free area lists for different orders
    struct free_area free_area[MAX_ORDER];
    
    // Zone statistics
    unsigned long pages_scanned;
    unsigned long flags;
    
    // Zone boundaries
    unsigned long zone_start_pfn;
    unsigned long spanned_pages;
    unsigned long present_pages;
    
    // Watermarks for memory pressure
    unsigned long pages_min;
    unsigned long pages_low;
    unsigned long pages_high;
};

struct free_area {
    struct list_head free_list;
    unsigned long nr_free;
};
```

#### Slab Allocator

```c
struct kmem_cache {
    // Cache metadata
    unsigned int size;           // Object size
    unsigned int align;         // Object alignment
    unsigned int flags;         // Cache flags
    
    // Slab management
    struct list_head slabs_full;
    struct list_head slabs_partial;
    struct list_head slabs_free;
    
    // Object management
    void (*ctor)(void *obj);    // Constructor
    void (*dtor)(void *obj);    // Destructor
    
    // Statistics
    unsigned long num;          // Number of objects
    unsigned long active_objs;   // Active objects
    unsigned long num_pages;    // Pages allocated
};
```

---

## File Systems {#filesystem}

### Virtual File System (VFS) Architecture

```
┌─────────────────────────────────────────┐
│           Application Layer              │
├─────────────────────────────────────────┤
│            System Calls                 │
│  (open, read, write, close, etc.)      │
├─────────────────────────────────────────┤
│          Virtual File System            │
│  ┌─────────────────────────────────────┐ │
│  │         File Operations            │ │
│  │  (inode_operations, file_ops)      │ │
│  └─────────────────────────────────────┘ │
│  ┌─────────────────────────────────────┐ │
│  │        Path Resolution             │ │
│  │  (dentry, vfsmount)                │ │
│  └─────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│          File System Types              │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │  ext4   │ │  NTFS   │ │  Btrfs  │   │
│  └─────────┘ └─────────┘ └─────────┘   │
├─────────────────────────────────────────┤
│           Block Device Layer            │
│  ┌─────────────────────────────────────┐ │
│  │      Block I/O Scheduler            │ │
│  │  (CFQ, deadline, noop)             │ │
│  └─────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│            Device Drivers               │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │   HDD   │ │   SSD   │ │   RAM   │   │
│  └─────────┘ └─────────┘ └─────────┘   │
└─────────────────────────────────────────┘
```

### Key VFS Objects

#### Inode Structure
```c
struct inode {
    // Inode identification
    umode_t i_mode;              // File type and permissions
    uid_t i_uid;                 // Owner user ID
    gid_t i_gid;                 // Owner group ID
    dev_t i_rdev;                // Device number
    
    // File information
    loff_t i_size;               // File size in bytes
    struct timespec i_atime;    // Access time
    struct timespec i_mtime;    // Modification time
    struct timespec i_ctime;    // Creation time
    
    // Reference counting
    atomic_t i_count;            // Reference count
    atomic_t i_writecount;       // Writers count
    
    // Operations
    const struct inode_operations *i_op;
    const struct file_operations *i_fop;
    
    // File system specific data
    struct super_block *i_sb;
    void *i_private;
    
    // Hash and list management
    struct hlist_node i_hash;
    struct list_head i_list;
    struct list_head i_sb_list;
};
```

#### File Structure
```c
struct file {
    // File identification
    struct path f_path;
    struct inode *f_inode;
    
    // File operations
    const struct file_operations *f_op;
    
    // File position
    loff_t f_pos;
    loff_t f_version;
    
    // Flags and mode
    unsigned int f_flags;
    unsigned int f_mode;
    
    // Locking
    spinlock_t f_lock;
    atomic_long_t f_count;
    
    // Private data
    void *private_data;
    
    // Read-ahead
    struct fown_struct f_owner;
    unsigned int f_uid, f_gid;
};
```

### File System Operations

#### File Operations Structure
```c
struct file_operations {
    struct module *owner;
    loff_t (*llseek) (struct file *, loff_t, int);
    ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
    ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
    ssize_t (*read_iter) (struct kiocb *, struct iov_iter *);
    ssize_t (*write_iter) (struct kiocb *, struct iov_iter *);
    int (*iterate) (struct file *, struct dir_context *);
    int (*iterate_shared) (struct file *, struct dir_context *);
    unsigned int (*poll) (struct file *, struct poll_table_struct *);
    long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);
    long (*compat_ioctl) (struct file *, unsigned int, unsigned long);
    int (*mmap) (struct file *, struct vm_area_struct *);
    unsigned long mmap_supported_flags;
    int (*open) (struct inode *, struct file *);
    int (*flush) (struct file *, fl_owner_t id);
    int (*release) (struct inode *, struct file *);
    int (*fsync) (struct file *, loff_t, loff_t, int datasync);
    int (*fasync) (int, struct file *, int);
    int (*lock) (struct file *, int, struct file_lock *);
    ssize_t (*sendpage) (struct file *, struct page *, int, size_t, loff_t *, int);
    unsigned long (*get_unmapped_area)(struct file *, unsigned long, unsigned long, unsigned long);
    int (*check_flags)(int);
    int (*flock) (struct file *, int, struct file_lock *);
    ssize_t (*splice_write)(struct pipe_inode_info *, struct file *, loff_t *, size_t, unsigned int);
    ssize_t (*splice_read)(struct file *, loff_t *, struct pipe_inode_info *, size_t, unsigned int);
    int (*setlease)(struct file *, long, struct file_lock **, void **);
    long (*fallocate)(struct file *file, int mode, loff_t offset, loff_t len);
    void (*show_fdinfo)(struct seq_file *m, struct file *f);
    ssize_t (*copy_file_range)(struct file *, loff_t, struct file *, loff_t, size_t, unsigned int);
    loff_t (*remap_file_range)(struct file *file_in, loff_t pos_in, struct file *file_out, loff_t pos_out, loff_t len, unsigned int remap_flags);
    int (*fadvise)(struct file *, loff_t, loff_t, unsigned int);
};
```

---

## Device Drivers {#drivers}

### Driver Architecture

```
┌─────────────────────────────────────────┐
│           Application Layer              │
├─────────────────────────────────────────┤
│            System Calls                 │
│  (open, read, write, ioctl, mmap)       │
├─────────────────────────────────────────┤
│           Virtual File System            │
├─────────────────────────────────────────┤
│          Device Driver Layer             │
│  ┌─────────────────────────────────────┐ │
│  │      Character Drivers              │ │
│  │  (serial, console, input devices)   │ │
│  └─────────────────────────────────────┘ │
│  ┌─────────────────────────────────────┐ │
│  │      Block Drivers                 │ │
│  │  (HDD, SSD, CD-ROM, RAM disk)     │ │
│  └─────────────────────────────────────┘ │
│  ┌─────────────────────────────────────┐ │
│  │      Network Drivers               │ │
│  │  (Ethernet, Wi-Fi, Bluetooth)      │ │
│  └─────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│            Bus Drivers                  │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │   PCI   │ │   USB   │ │  I2C    │   │
│  └─────────┘ └─────────┘ └─────────┘   │
├─────────────────────────────────────────┤
│              Hardware                   │
└─────────────────────────────────────────┘
```

### Character Device Driver Example

```c
// Character device driver structure
struct char_dev {
    struct cdev cdev;           // Character device structure
    struct class *class;       // Device class
    dev_t dev_num;              // Device number
    char *buffer;               // Device buffer
    size_t size;                // Buffer size
    struct mutex lock;          // Access lock
};

// File operations
static ssize_t char_read(struct file *filp, char __user *buf, 
                        size_t count, loff_t *f_pos) {
    struct char_dev *dev = filp->private_data;
    ssize_t ret = 0;
    
    mutex_lock(&dev->lock);
    
    if (*f_pos >= dev->size) {
        goto out;  // EOF
    }
    
    if (*f_pos + count > dev->size) {
        count = dev->size - *f_pos;
    }
    
    if (copy_to_user(buf, dev->buffer + *f_pos, count)) {
        ret = -EFAULT;
        goto out;
    }
    
    *f_pos += count;
    ret = count;
    
out:
    mutex_unlock(&dev->lock);
    return ret;
}

static ssize_t char_write(struct file *filp, const char __user *buf,
                         size_t count, loff_t *f_pos) {
    struct char_dev *dev = filp->private_data;
    ssize_t ret = 0;
    
    mutex_lock(&dev->lock);
    
    if (*f_pos >= dev->size) {
        ret = -ENOSPC;
        goto out;
    }
    
    if (*f_pos + count > dev->size) {
        count = dev->size - *f_pos;
    }
    
    if (copy_from_user(dev->buffer + *f_pos, buf, count)) {
        ret = -EFAULT;
        goto out;
    }
    
    *f_pos += count;
    ret = count;
    
out:
    mutex_unlock(&dev->lock);
    return ret;
}

static const struct file_operations char_fops = {
    .owner = THIS_MODULE,
    .read = char_read,
    .write = char_write,
    .open = char_open,
    .release = char_release,
};

// Module initialization
static int __init char_init(void) {
    int ret;
    
    // Allocate character device
    ret = alloc_chrdev_region(&char_dev.dev_num, 0, 1, "char_dev");
    if (ret < 0) {
        return ret;
    }
    
    // Initialize cdev
    cdev_init(&char_dev.cdev, &char_fops);
    char_dev.cdev.owner = THIS_MODULE;
    
    // Add character device
    ret = cdev_add(&char_dev.cdev, char_dev.dev_num, 1);
    if (ret < 0) {
        unregister_chrdev_region(char_dev.dev_num, 1);
        return ret;
    }
    
    // Create device class
    char_dev.class = class_create(THIS_MODULE, "char_class");
    if (IS_ERR(char_dev.class)) {
        cdev_del(&char_dev.cdev);
        unregister_chrdev_region(char_dev.dev_num, 1);
        return PTR_ERR(char_dev.class);
    }
    
    // Create device file
    device_create(char_dev.class, NULL, char_dev.dev_num, NULL, "char_dev");
    
    // Allocate buffer
    char_dev.buffer = kmalloc(PAGE_SIZE, GFP_KERNEL);
    char_dev.size = PAGE_SIZE;
    mutex_init(&char_dev.lock);
    
    printk(KERN_INFO "Character device loaded\n");
    return 0;
}

module_init(char_init);
```

### Block Device Driver

```c
// Block device structure
struct block_dev {
    struct gendisk *gd;         // Generic disk
    struct request_queue *queue; // Request queue
    spinlock_t lock;            // Queue lock
    void *data;                 // Device data
    int size;                   // Device size (sectors)
};

// Request processing
static void block_request(struct request_queue *q) {
    struct block_dev *dev = q->queuedata;
    struct request *req;
    
    while ((req = blk_fetch_request(q)) != NULL) {
        if (req->cmd_type != REQ_TYPE_FS) {
            __blk_end_request_all(req, -EIO);
            continue;
        }
        
        // Process the request
        block_transfer(dev, blk_rq_pos(req), blk_rq_cur_sectors(req),
                       bio_data(req->bio), rq_data_dir(req));
        
        // Complete the request
        __blk_end_request_all(req, 0);
    }
}

// Block transfer function
static void block_transfer(struct block_dev *dev, sector_t sector,
                          unsigned long nsect, char *buffer, int write) {
    unsigned long offset = sector * SECTOR_SIZE;
    unsigned long nbytes = nsect * SECTOR_SIZE;
    
    if (write) {
        memcpy(dev->data + offset, buffer, nbytes);
    } else {
        memcpy(buffer, dev->data + offset, nbytes);
    }
}
```

---

## Networking Stack {#networking}

### Network Protocol Stack Architecture

```
┌─────────────────────────────────────────┐
│           Application Layer              │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │   HTTP  │ │   FTP   │ │   SSH   │   │
│  └─────────┘ └─────────┘ └─────────┘   │
├─────────────────────────────────────────┤
│           Transport Layer               │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │   TCP   │ │   UDP   │ │   SCTP  │   │
│  └─────────┘ └─────────┘ └─────────┘   │
├─────────────────────────────────────────┤
│            Network Layer                │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │   IPv4  │ │   IPv6  │ │   ARP   │   │
│  └─────────┘ └─────────┘ └─────────┘   │
├─────────────────────────────────────────┤
│            Data Link Layer              │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │Ethernet │ │  802.11 │ │  PPP    │   │
│  └─────────┘ └─────────┘ └─────────┘   │
├─────────────────────────────────────────┤
│            Physical Layer               │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │  10/100 │ │   WiFi  │ │  Fiber  │   │
│  └─────────┘ └─────────┘ └─────────┘   │
└─────────────────────────────────────────┘
```

### Socket Implementation

```c
// Socket structure
struct socket {
    socket_state state;        // Socket state
    short type;                 // Socket type (SOCK_STREAM, etc.)
    unsigned long flags;        // Socket flags
    struct socket_wq wq;        // Wait queue
    
    struct file *file;          // Associated file
    struct sock *sk;            // Network layer socket
    const struct proto_ops *ops; // Protocol operations
};

// Network layer socket
struct sock {
    // Protocol family
    struct sock_common __sk_common;
    
    // Socket state and configuration
    unsigned char sk_state;
    unsigned char sk_protocol;
    unsigned short sk_type;
    
    // Memory and buffers
    atomic_t sk_rmem_alloc;
    atomic_t sk_wmem_alloc;
    struct sk_buff_head sk_receive_queue;
    struct sk_buff_head sk_write_queue;
    
    // Connection information
    struct sock *sk_peer;
    struct inet_connection_sock *icsk;
    
    // Callbacks and timers
    void (*sk_state_change)(struct sock *sk);
    void (*sk_data_ready)(struct sock *sk);
    void (*sk_write_space)(struct sock *sk);
    
    // Statistics
    struct sock_exterr_kern sk_err;
    u32 sk_ack_backlog;
    u32 sk_max_ack_backlog;
};

// Socket buffer (sk_buff)
struct sk_buff {
    struct sk_buff *next;       // Next buffer in list
    struct sk_buff *prev;       // Previous buffer in list
    
    // Network layer pointers
    struct sock *sk;            // Owning socket
    struct net_device *dev;     // Device for TX/RX
    
    // Data pointers
    char *head;                 // Head of buffer
    char *data;                 // Start of data
    char *tail;                 // End of data
    char *end;                  // End of buffer
    
    // Length information
    unsigned int len;           // Length of data
    unsigned int data_len;      // Data length
    unsigned int truesize;      // True size
    
    // Protocol headers
    __u16 mac_header;          // MAC header offset
    __u16 network_header;      // Network header offset
    __u16 transport_header;     // Transport header offset
    
    // Packet information
    __u32 priority;             // Packet priority
    __u8 pkt_type:5;            // Packet type
    __u8 ip_summed:2;           // Checksum status
    __u8 cloned:1;              // Is cloned
    __u8 nohdr:1;               // No headers
    __u8 nfctinfo:3;            // Connection tracking
    __u8 pkt_scratch:3;         // Scratch area
    __u8 nf_trace:1;            // Netfilter trace
    
    // Timestamp
    ktime_t tstamp;             // Receive timestamp
};
```

### Network Device Driver

```c
// Network device structure
struct net_device {
    char name[IFNAMSIZ];        // Interface name
    struct hlist_node name_hlist;
    char *ifalias;              // Interface alias
    
    // Device information
    unsigned long mem_end;      // Shared memory end
    unsigned long mem_start;    // Shared memory start
    unsigned long base_addr;    // Device I/O address
    unsigned int irq;           // Device IRQ number
    
    // Hardware and protocol headers
    unsigned char dev_addr[ETH_ALEN]; // Hardware address
    unsigned char broadcast[ETH_ALEN]; // Broadcast address
    unsigned char perm_addr[ETH_ALEN]; // Permanent address
    
    // Device operations
    const struct net_device_ops *netdev_ops;
    const struct ethtool_ops *ethtool_ops;
    
    // State and flags
    unsigned long flags;        // Interface flags
    unsigned long state;        // Device state
    unsigned long priv_flags;   // Private flags
    
    // Statistics
    struct rtnl_link_stats64 stats64;
    
    // Queue management
    struct netdev_queue *tx_queue;
    struct netdev_queue *rx_queue;
    
    // Private data
    void *priv;                 // Driver private data
    void *ml_priv;              // Multicast list private
};

// Network device operations
static const struct net_device_ops netdev_ops = {
    .ndo_open = netdev_open,
    .ndo_stop = netdev_close,
    .ndo_start_xmit = netdev_xmit,
    .ndo_set_mac_address = netdev_set_mac,
    .ndo_get_stats64 = netdev_get_stats64,
    .ndo_change_mtu = netdev_change_mtu,
};

// Packet transmission
static netdev_tx_t netdev_xmit(struct sk_buff *skb, struct net_device *dev) {
    struct netdev_priv *priv = netdev_priv(dev);
    unsigned int len = skb->len;
    
    // Add hardware headers if needed
    if (skb_cow_head(skb, 0)) {
        dev_kfree_skb(skb);
        return NETDEV_TX_OK;
    }
    
    // Queue packet for transmission
    spin_lock(&priv->tx_lock);
    
    if (priv->tx_queue_len >= TX_QUEUE_SIZE) {
        // Queue full, drop packet
        dev_kfree_skb(skb);
        priv->stats.tx_dropped++;
    } else {
        // Add to transmit queue
        list_add_tail(&skb->list, &priv->tx_queue);
        priv->tx_queue_len++;
        
        // Start transmission
        netdev_start_transmission(dev);
        
        priv->stats.tx_packets++;
        priv->stats.tx_bytes += len;
    }
    
    spin_unlock(&priv->tx_lock);
    
    return NETDEV_TX_OK;
}

// Packet reception
static void netdev_rx(struct net_device *dev, struct sk_buff *skb) {
    struct netdev_priv *priv = netdev_priv(dev);
    
    // Update statistics
    priv->stats.rx_packets++;
    priv->stats.rx_bytes += skb->len;
    
    // Pass packet to network stack
    skb->protocol = eth_type_trans(skb, dev);
    netif_rx(skb);
}
```

---

## Security and Permissions {#security}

### Security Architecture

```
┌─────────────────────────────────────────┐
│           Application Layer              │
├─────────────────────────────────────────┤
│            System Calls                 │
│  (access, open, chmod, setuid, etc.)    │
├─────────────────────────────────────────┤
│          Security Modules               │
│  ┌─────────────────────────────────────┐ │
│  │       Linux Security Module       │ │
│  │  (SELinux, AppArmor, Smack, etc.) │ │
│  └─────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│          POSIX Capabilities             │
│  ┌─────────────────────────────────────┐ │
│  │     Capability Sets               │ │
│  │  (Inheritable, Permitted, Effective)│ │
│  └─────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│            Access Control               │
│  ┌─────────────────────────────────────┐ │
│  │      Discretionary Access Control  │ │
│  │  (Owner, Group, Other permissions) │ │
│  └─────────────────────────────────────┘ │
│  ┌─────────────────────────────────────┐ │
│  │      Mandatory Access Control       │ │
│  │  (Security labels, contexts)       │ │
│  └─────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│            Kernel Hooks                 │
│  ┌─────────────────────────────────────┐ │
│  │      Security Hooks                │ │
│  │  (inode_permission, file_permission, │ │
│  │   task_setuid, socket_bind, etc.)   │ │
│  └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

### Linux Security Module (LSM) Framework

```c
// Security hook structure
struct security_hook_heads {
    struct list_head inode_create_security;
    struct list_head inode_link;
    struct list_head inode_unlink;
    struct list_head inode_symlink;
    struct list_head inode_mkdir;
    struct list_head inode_rmdir;
    struct list_head inode_mknod;
    struct list_head inode_rename;
    struct list_head inode_readlink;
    struct list_head inode_follow_link;
    struct list_head inode_permission;
    struct list_head inode_setattr;
    struct list_head inode_getattr;
    struct list_head inode_setxattr;
    struct list_head inode_getxattr;
    struct list_head inode_listxattr;
    struct list_head inode_removexattr;
    struct list_head file_permission;
    struct list_head file_alloc_security;
    struct list_head file_free_security;
    struct list_head file_ioctl;
    struct list_head file_mmap;
    struct list_head file_mprotect;
    struct list_head file_lock;
    struct list_head file_fcntl;
    struct list_head file_set_fowner;
    struct list_head file_send_sigiotask;
    struct list_head file_receive;
    // ... many more hooks
};

// Security operations
struct security_operations {
    char *name;
    int (*ptrace_access_check)(struct task_struct *child, unsigned int mode);
    int (*ptrace_may_access)(struct task_struct *child, unsigned int mode);
    int (*capget)(struct task_struct *target, kernel_cap_t *effective,
                  kernel_cap_t *inheritable, kernel_cap_t *permitted);
    int (*capset)(struct cred *new, const struct cred *old,
                  const kernel_cap_t *effective,
                  const kernel_cap_t *inheritable,
                  const kernel_cap_t *permitted);
    int (*acct)(struct file *file);
    int (*sysctl)(struct ctl_table *table, int op);
    int (*quotactl)(int cmds, int type, int id, struct super_block *sb);
    int (*quota_on)(struct dentry *dentry);
    int (*syslog)(int type);
    int (*settime)(struct timespec64 *ts, struct timezone *tz);
    int (*vm_enough_memory)(struct mm_struct *mm, long pages);
    // ... many more operations
};
```

### SELinux Implementation

```c
// SELinux security context
struct selinux_security_context {
    u32 user;                   // SELinux user
    u32 role;                   // SELinux role
    u32 type;                   // SELinux type
    u32 level;                  // MLS level
};

// SELinux access vector
struct av_decision {
    u32 allowed;                // Allowed permissions
    u32 decided;                // Decided permissions
    u32 auditallow;             // Audit allowed
    u32 auditdeny;              // Audit denied
    u32 seqno;                  // Sequence number
};

// SELinux policy database
struct selinux_policy {
    struct policydb policydb;   // Policy database
    struct sidtab sidtab;       // SID table
    struct selinux_mapping *map; // Mapping table
    u32 latest_granting;        // Latest granting SID
    u32 handle_unknown;         // Handle unknown setting
};

// Access check function
int selinux_permission(struct inode *inode, struct inode *dir,
                       const struct qstr *name, u32 requested) {
    struct selinux_security_context *isec = inode->i_security;
    struct selinux_security_context *dsec = dir->i_security;
    struct av_decision avd;
    u32 sid, tsid;
    int rc;
    
    // Get SIDs
    sid = isec->sid;
    tsid = dsec->sid;
    
    // Check permission
    rc = security_compute_av(sid, tsid, SECCLASS_FILE, requested, &avd);
    if (rc)
        return rc;
    
    // Check if permission is granted
    if (!(avd.allowed & requested)) {
        // Permission denied
        return -EACCES;
    }
    
    return 0;
}
```

---

## Inter-Process Communication {#ipc}

### IPC Mechanisms

```
┌─────────────────────────────────────────┐
│           Application Layer              │
├─────────────────────────────────────────┤
│            IPC Interfaces                │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │  Pipes  │ │  Sockets│ │Messages │   │
│  └─────────┘ └─────────┘ └─────────┘   │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │Signals  │ │Shared   │ │Semaphores│   │
│  │         │ │Memory   │ │         │   │
│  └─────────┘ └─────────┘ └─────────┘   │
├─────────────────────────────────────────┤
│            IPC Implementation             │
│  ┌─────────────────────────────────────┐ │
│  │      IPC Namespace                 │ │
│  │  (System V IPC, POSIX IPC)         │ │
│  └─────────────────────────────────────┘ │
│  ┌─────────────────────────────────────┐ │
│  │      Message Queues                │ │
│  │  (msgsnd, msgrcv, msgctl)         │ │
│  └─────────────────────────────────────┘ │
│  ┌─────────────────────────────────────┐ │
│  │      Shared Memory                 │ │
│  │  (shmat, shmdt, shmctl)           │ │
│  └─────────────────────────────────────┘ │
│  ┌─────────────────────────────────────┐ │
│  │      Semaphores                    │ │
│  │  (semop, semctl, semget)          │ │
│  └─────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│            Kernel IPC Layer              │
│  ┌─────────────────────────────────────┐ │
│  │      IPC Resource Management       │ │
│  │  (ipc_ids, ipc_namespace)          │ │
│  └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

### Message Queues

```c
// Message queue structure
struct msg_queue {
    struct kern_ipc_perm q_perm;  // IPC permissions
    time_t q_stime;               // Last send time
    time_t q_rtime;               // Last receive time
    time_t q_ctime;               // Creation time
    unsigned long q_cbytes;       // Current bytes
    unsigned long q_qnum;         // Number of messages
    unsigned long q_qbytes;       // Max bytes
    pid_t q_lspid;                // Last sender PID
    pid_t q_lrpid;                // Last receiver PID
    
    struct list_head q_messages;  // Message list
    struct list_head q_receivers;  // Receiver list
    struct list_head q_senders;    // Sender list
    
    struct msg_msg *q_lastmsg;    // Last message
    int q_notify;                 // Notification
};

// Message structure
struct msg_msg {
    struct list_head m_list;      // List links
    long m_type;                  // Message type
    size_t m_ts;                   // Text size
    struct msg_msgseg *next;      // Next segment
};

// Send message
long do_msgsnd(int msqid, long mtype, void __user *mtext,
               size_t msgsz, int msgflg) {
    struct msg_queue *msq;
    struct msg_msg *msg;
    int err;
    
    // Find message queue
    msq = msg_lock_check(msqid);
    if (IS_ERR(msq))
        return PTR_ERR(msq);
    
    // Allocate message
    msg = load_msg(mtext, msgsz);
    if (IS_ERR(msg)) {
        err = PTR_ERR(msg);
        goto out_unlock;
    }
    
    msg->m_type = mtype;
    
    // Add to queue
    list_add_tail(&msg->m_list, &msq->q_messages);
    msq->q_cbytes += msgsz;
    msq->q_qnum++;
    msq->q_lspid = task_tgid_vnr(current);
    msq->q_stime = get_seconds();
    
    // Wake up receivers
    wake_up(&msq->q_receivers);
    
    err = 0;
    
out_unlock:
    msg_unlock(msq);
    return err;
}
```

### Shared Memory

```c
// Shared memory segment
struct shmid_kernel {
    struct kern_ipc_perm shm_perm; // IPC permissions
    struct file *shm_file;          // Associated file
    unsigned long shm_nattch;       // Number of attaches
    unsigned long shm_segsz;        // Segment size
    time_t shm_atim;                // Last attach time
    time_t shm_dtim;                // Last detach time
    time_t shm_ctim;                // Creation time
    pid_t shm_cprid;                // Creator PID
    pid_t shm_lprid;                // Last operation PID
    
    struct user_struct *mlock_user; // User for locking
};

// Attach shared memory
long do_shmat(int shmid, char __user *shmaddr, int shmflg,
              ulong *raddr) {
    struct shmid_kernel *shp;
    unsigned long addr;
    unsigned long size;
    struct file *file;
    int err;
    
    // Find shared memory segment
    shp = shm_lock_check(shmid);
    if (IS_ERR(shp))
        return PTR_ERR(shp);
    
    // Get file and size
    file = shp->shm_file;
    size = i_size_read(file->f_path.dentry->d_inode);
    
    // Determine address
    if (shmaddr) {
        if (shmflg & SHM_RND)
            addr = (unsigned long)shmaddr & ~(SHMLBA - 1);
        else
            addr = (unsigned long)shmaddr;
    } else {
        if (shmflg & SHM_REMAP)
            addr = 0;
        else
            addr = get_unmapped_area(file->f_op->get_unmapped_area,
                                   file, addr, size, 0, 0);
    }
    
    // Map into process address space
    err = do_mmap_pgoff(file, addr, size, PROT_READ | PROT_WRITE,
                       MAP_SHARED | MAP_FIXED, 0);
    
    if (IS_ERR((void *)err)) {
        shm_unlock(shp);
        return err;
    }
    
    *raddr = err;
    shp->shm_nattch++;
    shp->shm_lprid = task_tgid_vnr(current);
    shp->shm_atim = get_seconds();
    
    shm_unlock(shp);
    return 0;
}
```

---

## Kernel Synchronization {#sync}

### Synchronization Primitives

```
┌─────────────────────────────────────────┐
│           Synchronization               │
├─────────────────────────────────────────┤
│  ┌─────────────────────────────────────┐ │
│  │         Spinlocks                  │ │
│  │  (spin_lock, spin_unlock)          │ │
│  │  - Busy waiting                    │ │
│  │  - Short critical sections         │ │
│  │  - Non-preemptive                  │ │
│  └─────────────────────────────────────┘ │
│  ┌─────────────────────────────────────┐ │
│  │         Mutexes                    │ │
│  │  (mutex_lock, mutex_unlock)        │ │
│  │  - Sleep waiting                   │ │
│  │  - Long critical sections          │ │
│  │  - Preemptive                      │ │
│  └─────────────────────────────────────┘ │
│  ┌─────────────────────────────────────┐ │
│  │         Semaphores                 │ │
│  │  (down, up)                       │ │
│  │  - Resource counting               │ │
│  │  - Producer-consumer               │ │
│  └─────────────────────────────────────┘ │
│  ┌─────────────────────────────────────┐ │
│  │      Read-Copy Update (RCU)        │ │
│  │  (rcu_read_lock, synchronize_rcu)  │ │
│  │  - Read-mostly workloads           │ │
│  │  - Lock-free reads                │ │
│  └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

### Spinlock Implementation

```c
// Spinlock structure
typedef struct {
    volatile unsigned int lock;    // Lock state
} spinlock_t;

#define SPINLOCK_UNLOCKED 0
#define SPINLOCK_LOCKED   1

// Acquire spinlock
static inline void spin_lock(spinlock_t *lock) {
    unsigned long flags;
    
    // Disable preemption and interrupts
    raw_spin_lock_irqsave(&lock->raw_lock, flags);
    
    // Try to acquire lock
    while (unlikely(test_and_set_bit_lock(0, &lock->lock))) {
        // Lock is busy, spin
        while (lock->lock) {
            cpu_relax();
        }
    }
}

// Release spinlock
static inline void spin_unlock(spinlock_t *lock) {
    // Release lock
    clear_bit_unlock(0, &lock->lock);
    
    // Enable preemption and interrupts
    raw_spin_unlock_irqrestore(&lock->raw_lock, flags);
}

// Try to acquire spinlock (non-blocking)
static inline int spin_trylock(spinlock_t *lock) {
    unsigned long flags;
    
    raw_spin_lock_irqsave(&lock->raw_lock, flags);
    
    if (test_and_set_bit_lock(0, &lock->lock)) {
        // Lock is busy
        raw_spin_unlock_irqrestore(&lock->raw_lock, flags);
        return 0;
    }
    
    return 1;
}
```

### Mutex Implementation

```c
// Mutex structure
struct mutex {
    atomic_long_t owner;           // Owner task
    struct list_head wait_list;     // Wait queue
    spinlock_t wait_lock;          // Wait queue lock
    void *magic;                   // Magic number
#ifdef CONFIG_DEBUG_MUTEXES
    struct task_struct *task;      // Debugging
#endif
};

// Acquire mutex
void __sched mutex_lock(struct mutex *lock) {
    might_sleep();
    
    // Fast path - try to acquire
    if (__mutex_fastpath_lock(&lock->owner, __mutex_lock_slowpath))
        return;
    
    // Slow path - need to wait
    __mutex_lock_slowpath(lock);
}

// Slow path lock acquisition
void __mutex_lock_slowpath(struct mutex *lock) {
    struct task_struct *task = current;
    
    // Add to wait queue
    spin_lock_mutex(&lock->wait_lock, flags);
    list_add_tail(&waiter.list, &lock->wait_list);
    waiter.task = task;
    spin_unlock_mutex(&lock->wait_lock, flags);
    
    // Wait until we get the lock
    for (;;) {
        // Check if we can acquire
        if (__mutex_trylock(lock)) {
            // Got the lock
            spin_lock_mutex(&lock->wait_lock, flags);
            list_del(&waiter.list);
            spin_unlock_mutex(&lock->wait_lock, flags);
            return;
        }
        
        // Schedule out
        set_current_state(TASK_UNINTERRUPTIBLE);
        schedule();
        set_current_state(TASK_RUNNING);
    }
}

// Release mutex
void __sched mutex_unlock(struct mutex *lock) {
    // Fast path release
    if (__mutex_fastpath_unlock(&lock->owner, __mutex_unlock_slowpath))
        return;
    
    // Slow path - wake up waiters
    __mutex_unlock_slowpath(lock);
}
```

### RCU Implementation

```c
// RCU read-side critical section
#define rcu_read_lock() preempt_disable()
#define rcu_read_unlock() preempt_enable()

// RCU callback structure
struct rcu_head {
    struct rcu_head *next;         // Next callback
    void (*func)(struct rcu_head *head); // Callback function
};

// Register RCU callback
void call_rcu(struct rcu_head *head, void (*func)(struct rcu_head *rcu)) {
    unsigned long flags;
    
    // Add to callback list
    spin_lock_irqsave(&rcu_ctrlblk.lock, flags);
    head->func = func;
    head->next = rcu_ctrlblk.next;
    rcu_ctrlblk.next = head;
    spin_unlock_irqrestore(&rcu_ctrlblk.lock, flags);
}

// Synchronize RCU
void synchronize_rcu(void) {
    // Wait for all pre-existing readers to complete
    synchronize_sched();
    
    // Process callbacks
    rcu_process_callbacks();
}

// Process RCU callbacks
static void rcu_process_callbacks(void) {
    struct rcu_head *head;
    unsigned long flags;
    
    // Process callback list
    spin_lock_irqsave(&rcu_ctrlblk.lock, flags);
    while ((head = rcu_ctrlblk.next) != NULL) {
        rcu_ctrlblk.next = head->next;
        spin_unlock_irqrestore(&rcu_ctrlblk.lock, flags);
        
        // Call callback
        head->func(head);
        
        spin_lock_irqsave(&rcu_ctrlblk.lock, flags);
    }
    spin_unlock_irqrestore(&rcu_ctrlblk.lock, flags);
}
```

---

## System Calls Interface {#syscalls}

### System Call Architecture

```
┌─────────────────────────────────────────┐
│           User Application              │
├─────────────────────────────────────────┤
│            System Call                  │
│  (syscall instruction, libc wrapper)    │
├─────────────────────────────────────────┤
│           System Call Table             │
│  ┌─────────────────────────────────────┐ │
│  │      System Call Dispatch          │ │
│  │  (sys_call_table, entry.S)         │ │
│  └─────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│           Kernel Implementation          │
│  ┌─────────────────────────────────────┐ │
│  │      System Call Handler           │ │
│  │  (sys_open, sys_read, sys_write)   │ │
│  └─────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│            Kernel Services               │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │  VFS    │ │  Memory │ │Network  │   │
│  └─────────┘ └─────────┘ └─────────┘   │
└─────────────────────────────────────────┘
```

### System Call Entry/Exit

```assembly
# x86_64 system call entry (entry_64.S)
ENTRY(system_call)
    # Save user registers
    swapgs
    movq %rsp, %gs:pda_oldrsp
    movq %gs:pda_kernelstack, %rsp
    
    # Stack frame
    pushq %rcx
    pushq %r11
    pushq %gs:pda_oldrsp
    
    # Check for syscall number
    cmpq $__NR_syscalls_max, %rax
    jae bad_syscall
    
    # Get system call table entry
    movq %rax, %rcx
    shrq $3, %rcx
    movq sys_call_table(, %rcx, 8), %r10
    
    # Check if system call exists
    testq %r10, %r10
    jz bad_syscall
    
    # Call system call
    call *%r10
    
    # Return path
    movq %rax, %gs:pda_retval
    jmp return_from_syscall

bad_syscall:
    movq $-ENOSYS, %rax
    jmp return_from_syscall

return_from_syscall:
    # Restore user registers
    movq %gs:pda_oldrsp, %rsp
    popq %gs:pda_oldrsp
    popq %r11
    popq %rcx
    swapgs
    
    # Return to user space
    sysretq
```

### System Call Implementation

```c
// System call table
const sys_call_ptr_t sys_call_table[__NR_syscalls_max] = {
    [0] = sys_read,
    [1] = sys_write,
    [2] = sys_open,
    [3] = sys_close,
    [4] = sys_stat,
    [5] = sys_fstat,
    [6] = sys_lstat,
    [7] = sys_poll,
    [8] = sys_lseek,
    [9] = sys_mmap,
    [10] = sys_mprotect,
    [11] = sys_munmap,
    [12] = sys_brk,
    // ... many more system calls
};

// System call: open()
SYSCALL_DEFINE3(open, const char __user *, filename, int, flags, umode_t, mode) {
    long ret;
    
    // Copy filename from user space
    char *kname = getname(filename);
    if (IS_ERR(kname))
        return PTR_ERR(kname);
    
    // Open file
    ret = do_sys_open(AT_FDCWD, kname, flags, mode);
    
    // Free copied filename
    putname(kname);
    
    return ret;
}

// System call: read()
SYSCALL_DEFINE3(read, unsigned int, fd, char __user *, buf, size_t, count) {
    struct fd f = fdget_pos(fd);
    ssize_t ret = -EBADF;
    
    if (!f.file)
        return -EBADF;
    
    // Check file permissions
    if (!(f.file->f_mode & FMODE_READ))
        goto out;
    
    // Perform read operation
    ret = vfs_read(f.file, buf, count, &f.file->f_pos);
    
out:
    fdput_pos(f);
    return ret;
}

// System call: write()
SYSCALL_DEFINE3(write, unsigned int, fd, const char __user *, buf,
                size_t, count) {
    struct fd f = fdget_pos(fd);
    ssize_t ret = -EBADF;
    
    if (!f.file)
        return -EBADF;
    
    // Check file permissions
    if (!(f.file->f_mode & FMODE_WRITE))
        goto out;
    
    // Perform write operation
    ret = vfs_write(f.file, buf, count, &f.file->f_pos);
    
out:
    fdput_pos(f);
    return ret;
}
```

### User Space to Kernel Space Data Transfer

```c
// Copy data from user space to kernel space
static inline long copy_from_user(void *to, const void __user *from, unsigned long n) {
    if (access_ok(VERIFY_READ, from, n)) {
        // Use optimized copy if available
        if (__builtin_constant_p(n)) {
            unsigned long ret;
            __copy_from_user(to, from, n, ret);
            return ret;
        } else {
            return __copy_from_user(to, from, n);
        }
    }
    
    // Access violation
    return n;
}

// Copy data from kernel space to user space
static inline long copy_to_user(void __user *to, const void *from, unsigned long n) {
    if (access_ok(VERIFY_WRITE, to, n)) {
        // Use optimized copy if available
        if (__builtin_constant_p(n)) {
            unsigned long ret;
            __copy_to_user(to, from, n, ret);
            return ret;
        } else {
            return __copy_to_user(to, from, n);
        }
    }
    
    // Access violation
    return n;
}

// Verify user space access
static inline int access_ok(int type, const void __user *addr, unsigned long size) {
    // Check if address range is in user space
    if (addr + size < addr)
        return 0;  // Overflow
    
    if (addr + size > current_thread_info()->addr_limit.seg)
        return 0;  // Beyond user space limit
    
    return 1;
}
```

---

## Implementation Examples {#examples}

### Complete Kernel Module Example

```c
/*
 * Complete Kernel Module Example
 * 
 * This module demonstrates:
 * - Character device driver
 * - Process management
 * - Memory management
 * - Timer implementation
 * - Proc filesystem interface
 * - Sysfs interface
 */

#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/device.h>
#include <linux/uaccess.h>
#include <linux/slab.h>
#include <linux/mutex.h>
#include <linux/timer.h>
#include <linux/proc_fs.h>
#include <linux/seq_file.h>
#include <linux/sysfs.h>
#include <linux/kobject.h>

#define DEVICE_NAME "kernel_tut"
#define CLASS_NAME  "kernel_tut"
#define BUFFER_SIZE PAGE_SIZE

// Module parameters
static int debug_enable = 0;
module_param(debug_enable, int, 0644);
MODULE_PARM_DESC(debug_enable, "Enable debug output");

static char *device_name = DEVICE_NAME;
module_param(device_name, charp, 0644);
MODULE_PARM_DESC(device_name, "Device name");

// Device structure
struct kernel_tut_dev {
    struct cdev cdev;           // Character device
    struct class *class;         // Device class
    dev_t dev_num;               // Device number
    
    // Data buffer
    char *buffer;                // Device buffer
    size_t size;                 // Buffer size
    size_t count;                // Data count
    
    // Synchronization
    struct mutex lock;           // Access lock
    wait_queue_head_t wait_queue; // Wait queue
    
    // Statistics
    unsigned long reads;         // Read operations
    unsigned long writes;        // Write operations
    unsigned long errors;        // Error count
    
    // Timer
    struct timer_list timer;     // Periodic timer
    unsigned long timer_interval; // Timer interval (jiffies)
    
    // Process tracking
    pid_t last_pid;              // Last accessing process
    char last_comm[TASK_COMM_LEN]; // Last process name
};

static struct kernel_tut_dev *dev;

// Forward declarations
static int kernel_tut_open(struct inode *inode, struct file *file);
static int kernel_tut_release(struct inode *inode, struct file *file);
static ssize_t kernel_tut_read(struct file *file, char __user *buf,
                               size_t count, loff_t *f_pos);
static ssize_t kernel_tut_write(struct file *file, const char __user *buf,
                                size_t count, loff_t *f_pos);
static long kernel_tut_ioctl(struct file *file, unsigned int cmd,
                            unsigned long arg);

// File operations
static const struct file_operations kernel_tut_fops = {
    .owner = THIS_MODULE,
    .open = kernel_tut_open,
    .release = kernel_tut_release,
    .read = kernel_tut_read,
    .write = kernel_tut_write,
    .unlocked_ioctl = kernel_tut_ioctl,
    .llseek = default_llseek,
};

// Timer callback
static void kernel_tut_timer_callback(struct timer_list *t) {
    struct kernel_tut_dev *dev = from_timer(dev, t, timer);
    
    // Update statistics
    dev->timer_interval = msecs_to_jiffies(1000);
    
    // Restart timer
    mod_timer(&dev->timer, jiffies + dev->timer_interval);
    
    if (debug_enable) {
        printk(KERN_DEBUG "%s: Timer triggered\n", device_name);
    }
}

// Device open
static int kernel_tut_open(struct inode *inode, struct file *file) {
    struct kernel_tut_dev *dev = container_of(inode->i_cdev,
                                             struct kernel_tut_dev, cdev);
    
    // Store device pointer
    file->private_data = dev;
    
    // Track accessing process
    dev->last_pid = current->pid;
    strncpy(dev->last_comm, current->comm, TASK_COMM_LEN - 1);
    dev->last_comm[TASK_COMM_LEN - 1] = '\0';
    
    if (debug_enable) {
        printk(KERN_DEBUG "%s: Opened by %s (pid %d)\n",
               device_name, dev->last_comm, dev->last_pid);
    }
    
    return 0;
}

// Device release
static int kernel_tut_release(struct inode *inode, struct file *file) {
    struct kernel_tut_dev *dev = file->private_data;
    
    if (debug_enable) {
        printk(KERN_DEBUG "%s: Closed by %s (pid %d)\n",
               device_name, current->comm, current->pid);
    }
    
    return 0;
}

// Device read
static ssize_t kernel_tut_read(struct file *file, char __user *buf,
                               size_t count, loff_t *f_pos) {
    struct kernel_tut_dev *dev = file->private_data;
    ssize_t ret = 0;
    
    mutex_lock(&dev->lock);
    
    // Check if we have data
    if (*f_pos >= dev->count) {
        if (file->f_flags & O_NONBLOCK) {
            ret = -EAGAIN;
            goto out;
        }
        
        // Wait for data
        mutex_unlock(&dev->lock);
        if (wait_event_interruptible(dev->wait_queue, 
                                     dev->count > *f_pos)) {
            return -ERESTARTSYS;
        }
        mutex_lock(&dev->lock);
    }
    
    // Calculate bytes to read
    if (*f_pos + count > dev->count) {
        count = dev->count - *f_pos;
    }
    
    // Copy to user
    if (copy_to_user(buf, dev->buffer + *f_pos, count)) {
        ret = -EFAULT;
        dev->errors++;
        goto out;
    }
    
    // Update position and statistics
    *f_pos += count;
    dev->reads++;
    ret = count;
    
    if (debug_enable) {
        printk(KERN_DEBUG "%s: Read %zd bytes\n", device_name, count);
    }
    
out:
    mutex_unlock(&dev->lock);
    return ret;
}

// Device write
static ssize_t kernel_tut_write(struct file *file, const char __user *buf,
                                size_t count, loff_t *f_pos) {
    struct kernel_tut_dev *dev = file->private_data;
    ssize_t ret = 0;
    
    mutex_lock(&dev->lock);
    
    // Check buffer space
    if (*f_pos + count > dev->size) {
        if (*f_pos >= dev->size) {
            ret = -ENOSPC;
            goto out;
        }
        count = dev->size - *f_pos;
    }
    
    // Copy from user
    if (copy_from_user(dev->buffer + *f_pos, buf, count)) {
        ret = -EFAULT;
        dev->errors++;
        goto out;
    }
    
    // Update position, count, and statistics
    *f_pos += count;
    if (*f_pos > dev->count) {
        dev->count = *f_pos;
    }
    dev->writes++;
    ret = count;
    
    // Wake up waiting readers
    wake_up_interruptible(&dev->wait_queue);
    
    if (debug_enable) {
        printk(KERN_DEBUG "%s: Wrote %zd bytes\n", device_name, count);
    }
    
out:
    mutex_unlock(&dev->lock);
    return ret;
}

// Device ioctl
static long kernel_tut_ioctl(struct file *file, unsigned int cmd,
                            unsigned long arg) {
    struct kernel_tut_dev *dev = file->private_data;
    long ret = 0;
    
    switch (cmd) {
    case 0: // Reset device
        mutex_lock(&dev->lock);
        dev->count = 0;
        memset(dev->buffer, 0, dev->size);
        mutex_unlock(&dev->lock);
        break;
        
    case 1: // Get statistics
        if (copy_to_user((void __user *)arg, &dev->reads, sizeof(unsigned long)))
            ret = -EFAULT;
        break;
        
    default:
        ret = -ENOTTY;
    }
    
    return ret;
}

// Proc filesystem interface
static int kernel_tut_proc_show(struct seq_file *m, void *v) {
    seq_printf(m, "Kernel Tutorial Device Statistics\n");
    seq_printf(m, "================================\n");
    seq_printf(m, "Device name: %s\n", device_name);
    seq_printf(m, "Buffer size: %zu bytes\n", dev->size);
    seq_printf(m, "Data count: %zu bytes\n", dev->count);
    seq_printf(m, "Read operations: %lu\n", dev->reads);
    seq_printf(m, "Write operations: %lu\n", dev->writes);
    seq_printf(m, "Error count: %lu\n", dev->errors);
    seq_printf(m, "Last process: %s (pid %d)\n", dev->last_comm, dev->last_pid);
    seq_printf(m, "Timer interval: %lu jiffies\n", dev->timer_interval);
    seq_printf(m, "Debug enabled: %s\n", debug_enable ? "yes" : "no");
    return 0;
}

static int kernel_tut_proc_open(struct inode *inode, struct file *file) {
    return single_open(file, kernel_tut_proc_show, NULL);
}

static const struct proc_ops kernel_tut_proc_ops = {
    .proc_open = kernel_tut_proc_open,
    .proc_read = seq_read,
    .proc_lseek = seq_lseek,
    .proc_release = single_release,
};

// Sysfs interface
static ssize_t debug_show(struct kobject *kobj, struct kobj_attribute *attr,
                         char *buf) {
    return sprintf(buf, "%d\n", debug_enable);
}

static ssize_t debug_store(struct kobject *kobj, struct kobj_attribute *attr,
                          const char *buf, size_t count) {
    int val;
    
    if (kstrtoint(buf, 10, &val))
        return -EINVAL;
    
    debug_enable = val ? 1 : 0;
    
    return count;
}

static ssize_t statistics_show(struct kobject *kobj, struct kobj_attribute *attr,
                             char *buf) {
    return sprintf(buf, "reads:%lu writes:%lu errors:%lu\n",
                   dev->reads, dev->writes, dev->errors);
}

static struct kobj_attribute debug_attribute =
    __ATTR(debug, 0644, debug_show, debug_store);

static struct kobj_attribute statistics_attribute =
    __ATTR_RO(statistics);

static struct attribute *kernel_tut_attrs[] = {
    &debug_attribute.attr,
    &statistics_attribute.attr,
    NULL,
};

static struct attribute_group kernel_tut_attr_group = {
    .attrs = kernel_tut_attrs,
};

static struct kobject *kernel_tut_kobj;

// Module initialization
static int __init kernel_tut_init(void) {
    int ret;
    
    printk(KERN_INFO "%s: Initializing kernel tutorial module\n", device_name);
    
    // Allocate device structure
    dev = kzalloc(sizeof(struct kernel_tut_dev), GFP_KERNEL);
    if (!dev) {
        ret = -ENOMEM;
        goto err_alloc;
    }
    
    // Allocate buffer
    dev->buffer = kzalloc(BUFFER_SIZE, GFP_KERNEL);
    if (!dev->buffer) {
        ret = -ENOMEM;
        goto err_buffer;
    }
    dev->size = BUFFER_SIZE;
    
    // Initialize mutex and wait queue
    mutex_init(&dev->lock);
    init_waitqueue_head(&dev->wait_queue);
    
    // Allocate device number
    ret = alloc_chrdev_region(&dev->dev_num, 0, 1, device_name);
    if (ret < 0) {
        goto err_chrdev;
    }
    
    // Initialize cdev
    cdev_init(&dev->cdev, &kernel_tut_fops);
    dev->cdev.owner = THIS_MODULE;
    
    // Add character device
    ret = cdev_add(&dev->cdev, dev->dev_num, 1);
    if (ret < 0) {
        goto err_cdev;
    }
    
    // Create device class
    dev->class = class_create(THIS_MODULE, CLASS_NAME);
    if (IS_ERR(dev->class)) {
        ret = PTR_ERR(dev->class);
        goto err_class;
    }
    
    // Create device file
    device_create(dev->class, NULL, dev->dev_num, NULL, "%s", device_name);
    
    // Initialize timer
    timer_setup(&dev->timer, kernel_tut_timer_callback, 0);
    dev->timer_interval = msecs_to_jiffies(1000);
    mod_timer(&dev->timer, jiffies + dev->timer_interval);
    
    // Create proc entry
    proc_create(device_name, 0, NULL, &kernel_tut_proc_ops);
    
    // Create sysfs entries
    kernel_tut_kobj = kobject_create_and_add("kernel_tut", kernel_kobj);
    if (!kernel_tut_kobj) {
        ret = -ENOMEM;
        goto err_sysfs;
    }
    
    ret = sysfs_create_group(kernel_tut_kobj, &kernel_tut_attr_group);
    if (ret) {
        goto err_sysfs_group;
    }
    
    printk(KERN_INFO "%s: Module loaded successfully\n", device_name);
    printk(KERN_INFO "%s: Device major: %d, minor: %d\n",
           device_name, MAJOR(dev->dev_num), MINOR(dev->dev_num));
    
    return 0;
    
err_sysfs_group:
    kobject_put(kernel_tut_kobj);
err_sysfs:
    remove_proc_entry(device_name, NULL);
    device_destroy(dev->class, dev->dev_num);
    del_timer_sync(&dev->timer);
    class_destroy(dev->class);
err_class:
    cdev_del(&dev->cdev);
err_cdev:
    unregister_chrdev_region(dev->dev_num, 1);
err_chrdev:
    kfree(dev->buffer);
err_buffer:
    kfree(dev);
err_alloc:
    return ret;
}

// Module cleanup
static void __exit kernel_tut_exit(void) {
    printk(KERN_INFO "%s: Unloading kernel tutorial module\n", device_name);
    
    // Remove sysfs entries
    sysfs_remove_group(kernel_tut_kobj, &kernel_tut_attr_group);
    kobject_put(kernel_tut_kobj);
    
    // Remove proc entry
    remove_proc_entry(device_name, NULL);
    
    // Delete timer
    del_timer_sync(&dev->timer);
    
    // Remove device
    device_destroy(dev->class, dev->dev_num);
    class_destroy(dev->class);
    cdev_del(&dev->cdev);
    unregister_chrdev_region(dev->dev_num, 1);
    
    // Free resources
    kfree(dev->buffer);
    kfree(dev);
    
    printk(KERN_INFO "%s: Module unloaded successfully\n", device_name);
}

module_init(kernel_tut_init);
module_exit(kernel_tut_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Kernel Tutorial");
MODULE_DESCRIPTION("Comprehensive kernel module tutorial");
MODULE_VERSION("1.0");
```

---

## Conclusion

This comprehensive kernel tutorial covers the fundamental aspects of operating system kernel design and implementation. We've explored:

1. **Kernel Architecture**: Understanding the different kernel types and their trade-offs
2. **Process Management**: How processes are created, scheduled, and managed
3. **Memory Management**: Virtual memory, paging, and memory allocation
4. **File Systems**: VFS layer and file system operations
5. **Device Drivers**: Character and block device implementation
6. **Networking Stack**: Socket implementation and network drivers
7. **Security**: Access control and security frameworks
8. **IPC**: Inter-process communication mechanisms
9. **Synchronization**: Locks, mutexes, and RCU
10. **System Calls**: Interface between user and kernel space

The complete kernel module example demonstrates how these concepts come together in a real implementation. Understanding these fundamentals is essential for anyone working with operating systems, whether you're developing drivers, debugging kernel issues, or designing new kernel features.

Remember that kernel programming requires careful attention to:
- Memory management and allocation
- Proper synchronization to avoid race conditions
- Error handling and cleanup
- Security considerations
- Performance optimization

Always test kernel code thoroughly and use proper debugging techniques to ensure system stability and reliability.
