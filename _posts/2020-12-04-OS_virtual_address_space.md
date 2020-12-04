---
title: "Virtual Address Space"
date: 2020-12-04 10:17:00
categories: OS
---



최대 4GB (archi. dep.)


## 접근 불가 영역
    ex.
    08048000 - 0804c000
    prcess에게 접근 권한이 없는 메모리 주소 영역
    유효한 주소의 이런 영역을 -> 메모리 영역이라고 함

&#160;

## segment fault
    유효하지 않은 영역에 access 시 발생

&nbsp;

## memory section types
    1) text section (RO)
    2) data section (RW)
        실행 파일의 초기값이 있는 전역 변수가 할당된 메모리
    3) zero page (BSS)
        초기값이 없는 전역 변수가 들어 있는 영역
    4) process user space stack
    5) C lib용 
        text, data, BSS sections
    6) memory 할당 file 영역
    7) 공유 메모리 구간
    8) malloc 등의 함수와 관련된 '익명' 할당 메모리


    process address space의 모든 유효 address는 한 영역에 속함 
    영역은 겹치지 않음
    
    실행 중인 process는
        stack
        object code
        global variables
        memory allocation file
        
        등 여러 구분된 memory sections들을 지님

&#2;

## Memroy Descriptor

    memory descriptor로 process의 memory address space를 표현
    
    <linuc/mm_types.h> 내

```C
    struct mm_struct {
        struct vm_area_struct   *mmap;           <-
        struct rb_root           mm_rb;


            동일 자료 구조에 대한 두가지 표현 (각각 list와 rb tree - O(lnN) time)            
            
            보통 중복 표현을 피하지만, 
            list의 경우 순차 탐색에 용이하며, rb tree는 특정 항목 탐색에 용이
            
            이렇게 같은 data를 두 가지 다른 접근 방식으로 사용하는 것 --> thread tree


        
        struct vm_area_struct   *mmap_cache;      <- 최근 사용한 메모리 영역
        ...
        pgd_t                   *pgd;            <- 전체 page list
        
        atomic_t                 mm_users;       <- 주소 공간 사용자 (process 개수 - 즉, thread 개수)
        atomic_t                 mm_count;       <- mm_struct의 참조 횟수
        
                9개의 thread가 주소 공간을 공유해도, mm_users 값은 9지만, mm_count는 1임
                mm_users 값이 0이 될 때 mm_count도 0이 됨 (이때 메모리 해제)

        ...

        struct list_head        mmlist;            /* list of all mm_structs */
        
            모든 mm_struct는 mlist를 통해서 list로 관리됨
            list 내 최초 item은 init processs의 address space를 나타내는 init_mm memory descriptor임
            mm_struct init_mm은, /kernel/mm/init-mm.c에 존재
            
                kernel/fork.c에 정의된 mmlist_lock에 의해서 동시 접근이 제한



        unsigned long   start_code;           /* start address of code */
        unsigned long   end_code;             /* final address of code */
        unsigned long   start_data;           /* start address of data */
        unsigned long   end_data;             /* final address of data */
        unsigned long   start_brk;            /* start address of heap */
        unsigned long   brk;                  /* final address of heap */
        unsigned long   start_stack;          /* start address of stack */
        unsigned long   arg_start;            /* start of arguments */
        unsigned long   arg_end;              /* end of arguments */
        unsigned long   env_start;            /* start of environment */
        unsigned long   env_end;              /* end of environment */

        unsigned long       rss;                  /* pages allocated */
        unsigned long       total_vm;             /* total number of pages */
        ...
        cpumask_t cpu_vm_mask; /* lazy TLB switch mask */
        ...
        struct core_state *core_state; /* core dump support */
        ...
   };
```


## memory descriptor allocation

    특정 task의 memory descriptor는 task의 process descriptor 내 mm 항목(mm_struct)을 통해 연결되어 있음
    
    즉, 
        current->mm 은 현재 process의 memory descriptor를 의미
    
    fork 시, 
        copy_mm 함수가 parent process의 memory descriptor를 child process로 복사
    
    mm_struct
        allocate_mm은 mm_cachep (slab cache 임)를 통해 mm_struct를 할당
        process는 보통 독자적인 mm_struct (즉, 돚가적인 process address space)를 할당 받음

    clone
        에서 CLONE_VM flag로,
        ㅂ모와 자식간 주소 공간 공유
            위 flag 설정 시, 
            tsk->mm = current->mm; 임
        
        이런 process를 thread라고 함
        
    
    process 종료 시, 
        exit_mm이 호출 (@ kernel/exit.c)
        mm_count를 줄이고, 0이 되면, 더 이상 memory descriptor 사용처가 없기에 free_mm 호출
            free_mm --> kmem_cache_free

        mm_struct 구조체가 mm_cachep slab cache로 돌아감



## mm_struct 구조체와 KERNEL thread

    kernel thread process의 memroy descriptor인 mm은 NULL
    
    kernel memory 접근을 위한 page table과 같이 kernel thread에 일부 data가 필요한 경우가 존재    
    kernel thread는 이전에 실행한 task의 memory descriptor를 사용
    
    process가 scheduling 될 시, process의 mm 항목이 참조하는 process address space가 load 됨
    새로운 address




    새로운 address space를 사용하도록 active_mm을 update
    kernel thread는 자신만의 address space가 없기에 mm은 NULL
    
    kernel thread가 scheduling 되면, mm 항목이 NULL 이라는 것을 확인하고 
    사용 중인 이전 process의 주소 공간을 그대로 둠
    kernel thread의 memory descriptor의 active_mm을 이전 process memory descriptor가 가리키던 곳으로 갱신
    
    이로서 kernel thread는 이전 process의 page table이 필요할 때 사용 가능
    kernel thread는 user level address space에 접근하지 않으므로 커널 memory에 해당하는 address space 정보만 사용



## virtual memory area (VMA)

```
    +----------------+ *   1 +-----------+
    | vm_area_struct |<----<>| mm_struct |
    +----------------+       +-----------+
```
        특정 process의 address space를 표현하는 memory descriptor(mm_struct)는 여러 memory area (vm_area_struct)로 구성



    vm_area_struct로 표현 <linux/mm_types.h>
    
        리눅서 커널에서 memory area는 가상 메모리 영역(VMA)라고 부르는 경우가 많음
        주어진 주소 공간에서 연속된 구간에 해당하는 단일 메모리 영역을 나타냄
    
        각 memory area를 독립적인 memory object로 간주 (area 별로 권한 및 작업 가능 범위가 다름)
    
            여러 종류의 memory ara에 대한 '추상'화된 표현
    
        memroy가 할당된 file, process의 user level stack과 같은 서로 다른 유형의 memory area를 VMA 구조체로 표현
    
            VFS에서 사용했던 객체지향 접근법과 동일

    
```C
    struct vm_area_struct {
        ...
        struct mm_struct *vm_mm;                    /* The address space we belong to. */
    
            VMA에 대당하는 mm_struct임
            VMA 별로 고유한 mm_struct를 지님
    
                두 개의 별도 process가 같은 file을 각자의 주소 공간에 할당하는 경우, 
                각자 별도의 vm_area_struct를 통해 각자의 memory 공간을 식별
    
                반면, 주소 공간 공유 경우, 
                두 thread는 모든 vm_area_struct 구조체를 공유
    
    
        unsigned long vm_start;                        /* Our start address within vm_mm. */
        unsigned long vm_end;                        /* The first byte after our end address within vm_mm. */
    

        pgprot_t vm_page_prot;                        /* Access permissions of this VMA. */    
    
        struct vm_area_struct *vm_next, *vm_prev;    VMA list          
    
        struct rb_node vm_rb;                        tree 상의 VMA node
    
    
        union....
    
    
        struct anon_vma *anon_vma;                   익명 VMA 객체
    
    
        struct vm_operations_struct *vm_ops;         관련 동작 (해당 VMA 영역에 대한 동작)
    
        struct file * vm_file;                 할당된 file (can be NULL)
    
        ...
    };
```


    각 memory descriptor는 process address space 내의 고유 구간을 나타냄
    vm_start 항목은 해당 구간의 (낮은)시작 address 
    vm_end는 구간의 마지막 주소(높은)
    
    vm_end - vm_start가 memory area의 byte length [vm_start, vm_end)



## VMA flag
```
    vm_area_struct
     +- ...
     +- unsigned long vm_flags
```

    <linux/mm.h>    
    VMA flag는 hardware가 아닌 kernel의 동작 지정


    VM_READ,
    VM_WRITE
    VM_EXEC
    VM_SHARED           page will be shared
                        memory region이 여러 process가 공유함을 의미
                        shared mapping
                        (개별 할당인 경우 private mapping)
    ...
    VM_IO               mmap 호출 시 설정 됨 (메모리 영역이 process의 core dump에 들어가지 않게 함)
    VM_SEQ_READ         sequentially accessible page
                        app이 해당 역에 대해 순차적 읽기 동작을 수행한다고 kerenl에 알림
                        (커널이 해당 vm_area_struct 영역에 대해서 미리 읽기 기능을 사용할 수 있음)

    VM_RAND_READ        VM_SEQ_READ의 반대
    ...
    VM_RESERVED         memory region이 swap 되지 않도록 하는 역할 (device driver 할당에서 사용)
    
    
    ex.
        object code는 VM_READ, VM_EXEC는 할당해야 함, 
                      VM_WRITE는 할당하면 안 됨

        object data는 VM_READ, VM_WRITE 할당, VM_EXEC는 할당 X


## VMA 동작
    vm_area_struct 구조체의 vm_ops 항목
        : 커널이 해당 memory region을 조작히는 방법

    
    
    vm_operations_struct
     +- void (*open)(struct vm_area_struct * area);
     +- void (*close)(struct vm_area_struct * area);
     +- int (*fault)(struct vm_area_struct *vma, struct vm_fault *vmf);
     |
     +- int (*page_mkwrite)(struct vm_area_struct *vma, struct vm_fault *vmf);
     +- int (*access)(struct vm_area_struct *vma, unsigned long addr, void *buf, int len, int write);
     |
     +- NUMA ops
     |
     +- int (*remap_pages)(struct vm_area_struct *vma, unsigned long addr, unsigned long size, pgoff_t pgoff);


    int fault
        : physical memory가 없는 page에 access 시page fault handler가 호출하는 함수
    
    int page_mkwrite(...)
        : read only page를 writeable하게 변경 시 page fault handler가 호출하는 함수

    int access(...)
        : get_user_pages() 호출 failure 시 access_process_vm에서 호출하는 함수



## memory area list & tree (memory area thread tree)

    
        struct mm_struct {
            struct vm_area_struct * mmap;        /* list of VMAs */
            struct rb_root mm_rb;
            struct vm_area_struct * mmap_cache;    /* last find_vma result */
            
            ...
            

    mmap과 mm_rb는 thread tree 자료 구조임
    
        mmap
            : 모든 memory area object를 하나의 linked list로 보관
            
            vm_area_struct의 vm_next 항목을 통해 list 형태로 연결 됨
            오름착순 정렬
            첫 번째 memory area는 mmap pointer가 가키리는 vm_area_sturct
            마지막 구조체는 NULL

        mm_Rb
            : 모든 memory area 객체를 RB-tree로 연결
            mm_rb가 가리키는 곳이 RB-tree의 root
            address space에 속하는  각 vm_area_struct 구조체는 vm_rb 항목을 통해서 연결됨

            cf.
            RB-tree는
                왼편이 우측보다 작은 값
                각 노드는 Red, Back의 상태를 지님
             
                red의 child는 black
                root에서 left로 가는 모든 경로는 같은 개수의 black node가 있음
                root node는 항상 red node
                
                검색, 삽입, 삭제가 O(log(n))   <-- O(logN)을 보장한다는 것이 특징 (FBT는 보장은 못함)


## Actual memory area
    process의 address space와 memory area 내부
    
        /proc
        pmap utility
        
        를 통해서 살펴 볼 수 있음


    ex.
        특정 app의 address space (mm_struct의 각 vm_area_struct) 살펴보기
        
        
            text area
            data area
            BSS  area
            
        shared lib. link 시 각 shared lib이 사용하는 text/data/BSS area가 존재
        
            그리고
            process의 stack area

        
        /proc/<pid>/maps
        
            00e80000-00faf000       r-xp    00000000 03:01 208530  /lib/tls/libc-2.5.1.so
            ...
            
            시작-끝  권한  offset  majkor:minor  inode  file


        pmap 1426
        
            ... 더 보기 편하게 출력
            
            4001e000 (4KB)  rw-p    (00:00 0)                   <-- ZI
            bfffe000 (8KB)  rwxp    (00:00 0)       [ stack ]   <-- process stack


    실행코드이기에 text area는 read/executable
    반면 data area와 BSS area는 read/write 가능
    
        1340KB 중 40KB만 writable area
        
        늘 write가 불가능한 영역은 모든 process가 하나만 사용한다.
        (즉, write 발생 시 memory 영역을 복사 하는 CoW 정책에 의함)
        

        위에서 ZI는 file이 할당되어 있지 않은 장치 00:00과 0번 inode memory area이다.



## memroy area handling
    커널은 주어진 VMA에 대해서 주소 존재 여부 확인과 같은 동작을 빈번히 수행함    
    mmap의 기본 동작을 구성


    struct vm_area_struct *
    find_vma(sruct mm_struct *mm, unsigned long addr);
    
        주어진 주소공간(mm)에 대해 vm_end 항목 값이 addr 보다 큰 첫 번째 memory area를 찾음
        addr이 들어 있거나 addr 보다 큰 주소로 시작하는 첫 번째 memory area를 찾음
        
            해당 vm_area_struct를 리턴
            (VMA는 addr 보다 큰 주소이기에 addr이 VMA에 반드시 들어 있음을 보장하지는 않음)

        find_vma의 return은 memory descriptor(mm)의 mmap_cache 항목 안에 cache 됨
            VMA에 대한 동작이 수행 되었다는 의미이므로, cache의 temporal 특성에 의해 hit rate는 30~40%

        주어진 주소가 cache에 없으면 memory descriptor(mm)에 속한 memory area(vma_area_struct)를 탐색
            RB-tree를 통해 탐색


            struct vm_area_struct *find_vma(struct mm_struct *mm, unsigned long addr)
            {
                struct vm_area_struct *vma = NULL;
                
                vma = ACCESS_ONCE(mm->mmap_cache);
                if (!(vma && vma->vm_end > addr && vma->vm_start <= addr)) {  <-- cache에 없으면, 
                    ...
                    
                    rb_node = mm->mm_rb.rb_node;
                    vma = NULL;
                    
                    while (rb_node) {
                        struct vm_area_struct *vma_tmp;
                        vma_tmp = rb_entry(rb_node, struct vm_area_struct, vm_rb);
                        
                        if (vma_tmp->vm_end > addr) {
                            vma = vma_tmp;
                            
                            if (vma_tmp->vm_start <= addr)
                                break;
                            rb_node = rb_node->rb_left;
                        } else {
                            rb_node = rb_node->rb_right;
                        }
                    }
                    
                    if (vma)
                        mm->mmap_cache = vma;           <-- 방금 찾은 것은 cache에 keep (hit rate 30~40%)            
                }
            
                return vma;
            }



    struct vm_area_struct *
    find_vma_prev(struct mm_struct *mm, unsigned long addr,
        struct vm_area_struct **pprev)
        
        
        find_vma의 동작과 동일하나 addr 이전의 마지막 VMA를 반환하는 점이 다름
        mm/mmap.c file에 정의되어 있음, <linux/mm.h>에 선언
        
        pprev 인자에는 addr 앞 쪽의 VMA pointer가 저장됨


    static inline struct vm_area_struct *
    find_vma_intersection(struct mm_struct * mm, unsigned long start_addr, unsigned long end_addr)
    
        지정한 주소 범위와 중첩되는 첫 번째 VMA를 리턴
    
        struct vm_area_struct * vma = find_vma(mm,start_addr);
        if (vma && end_addr <= vma->vm_start)
            vma = NULL;
        return vma;



## mmap() & do_mmap(): address range generation


    do_mmap
        : do_mmap은 process의 address space에 address range를 추가
        기존 memory area의 확장 or 생성 시 기존 address space에 address range를 추가


        커널이 연속된 주소 범위를 새로 만들 때 사용
        (단 생성된 주소 범위가 기존 주소 범위와 인접하면 병합해서 합침 -> 즉, 생성이라고 부를 수 없음)
        
        합병 불가능 시, 
            1) vm_area_cachep slab cache에서 새로운 vm_area_struct 구조체를 할당,
            2) vma_link 호출
                새로운 VMA을 address space의 linked list와 RB-tree에 추가
            3) memory descriptor (mm_struct)의 total_vm 항목 갱신
            
            그리고 
            새로 생성된 주소 범위의 초기 주소 값을 리턴


    do_mmap 함수의 동작은 mmap system call을 통해 user space에 제공됨
        mmap system call 호출은 다음과 같이 정의

            unsigned long sys_mmap2(unsigned long addr, size_t len,
                        unsigned long prot, unsigned long flags,
                        unsigned long fd, unsigned long pgoff)
            {
                return do_mmap2(addr, len, prot, flags, fd, pgoff, PAGE_SHIFT-12);
            }
            
            
            이 system call은 mmap의 두 번째 변화형이라 mmap2임
            워래 mmap은 마지막 인자로 byte unit offset을 사용했음
            
            mmap2는 page unit offset 사용
                offset이 더 커져서 더 큰 file 할당 가능
                

        
    do_mmap(file *file, addr, len, prot, flga, offset)
    
        offset 위치에 len 길이만큼 주소 범위를 할당
        file에 NULL, offset에 0 지정 경우
            anynymous mapping
        
        file과 offset이 지정된 경우
            file-backed mapping

        prot
            : page 접근 권한 지정
                PROT_READ       : VM_READ에 해당
                PROT_WRITE      : VM_WRITE
                PROT_EXEC       : VM_EXEC
                PROT_NONE       : page에 접근 불가

        flag
            : VMA flag에 해당하는 flag
            
                MAP_SHARED      : 공유 가능 allocation
                MAP_PRIVATE     : 공유 불가
                MAP_FIXED       : 새로 생성되는 주소 range는 지정한 addr 주소에서 시작
                MAP_ANYNYMOUS   : 파일 지정 할당이 아닌 익명 할당
                ....



## munmap() & do_munmap(): address range free

    do_munmap
     : 지정한 process address space에서 address range를 제거
     (즉, VMA를 제거)
     
     
        int do_munmap(struct mm_struct *mm, start, len)

    int munmap(void *start, len)


    이 system call은 mm//mmap.c file에 정의
    단순히 do_munmap()의 wrapper
    
    
    
        armlinkage long sys_munmap(unsigned long addr, size_t len)
        {
            ...
            struct mm_struct *mm;
            
            mm = current->mm;
            down_write(&mm->mmap_sem);
            ret = do_munmap(mm, addr, len);
            up_write(&mm>mmap_sem);
            return ret;
        }
    
        
## Page table
    virtual address는, physical page table entry (page-frame)이 할당된 virtual page table entry를 사용
    
    page table에 access는 CPU가 내부적으로 알아서, 
        virtual address --> logical address --> linear address --> physical address
        
            각 단계별 page table들이 level 별로 존재
    
    Linux page table
        3 level로 구성        
        H/W가 3 level을 지원하지 않아도 Linux kernel은 3 단계를 사용
        
        세 단계를 사용하는 것은 일종의 '최대 공약수' 같은 개념
        필요에 의해 compiler optimization을 통해 kernel page table을 단순화 시킬 수 있음
        
    PGD (Page Global Directory)
        최상위 page table은 (page global directory인)pgd_t 형 배열
        
            pgd_t는 대부분 unsigned long

        PGD는 PMD (Page Middle Directory)를 가리킴
        
            pmd_t
            
        PMD는 PTE (Page Table Entry)를 가리킴
        
            pte_t

 ```
                                                                            +-------+
                              +-------+              +-------+              | pte_t |          +------+
    +-----------+             | pgd_t |              | pmd_t |              +-------+    +---> | page | 
    | mm_struct |------+      +-------+              +-------+              | pte_t |    |     +---|--+
    +-----------+      |      | pgd_t |       +----->| pmd_t |--------+     +-------+    |         |
                       |      +-------+       |      +-------+        |     | pte_t |    |         |
                       +----->| pgd_t | ------+      | pmd_t |        |     +-------+    |         v
                              +-------+              +-------+        |     | pte_t |    |      page-frame
                              | pgd_t |              | pmd_t |        |     +-------+    |
                              +-------+              +-------+        +---->| pte_t |----+
                              | pgd_t |              | pmd_t |              +-------+
                              +-------+              +-------+              
```

     TLB 사용
        TLB hit 시 TLB entry 사용
        아니면 위의 단계를 수행해서 physical page-frame 값 획득
