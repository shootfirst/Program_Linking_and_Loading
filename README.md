# Program_Linking_and_Loading
## 程序的装载与链接学习笔记

### elf64文件格式与相关数据结构

  参考#include <elf.h>，该头文件通常在/usr/include/elf.h
  
  - elf文件头
  
      * 数据结构：
      
          typedef struct
          {
          unsigned char	e_ident[EI_NIDENT];	/* Magic number and other info */
          Elf64_Half	e_type;			/* Object file type */
          Elf64_Half	e_machine;		/* Architecture */
          Elf64_Word	e_version;		/* Object file version */
          Elf64_Addr	e_entry;		/* Entry point virtual address */
          Elf64_Off	e_phoff;		/* Program header table file offset */
          Elf64_Off	e_shoff;		/* Section header table file offset */
          Elf64_Word	e_flags;		/* Processor-specific flags */
          Elf64_Half	e_ehsize;		/* ELF header size in bytes */
          Elf64_Half	e_phentsize;		/* Program header table entry size */
          Elf64_Half	e_phnum;		/* Program header table entry count */
          Elf64_Half	e_shentsize;		/* Section header table entry size */
          Elf64_Half	e_shnum;		/* Section header table entry count */
          Elf64_Half	e_shstrndx;		/* Section header string table index */
          } Elf64_Ehdr;
      
      * 字段解析：
      
          + e_ident  16字节
          
          存储魔数及其他相关信息。最开始的4个字节是所有ELF文件都必须相同的标识码，分别为0x7F、0x45、0x4c、0x46，第一个字节对应ascii码的DEL控制符，而后三字节刚好是ELF三个字母
          的ascii码，几乎所有可执行文件开始都是魔数，一般这种魔数用于确定可执行文件类型，以确定下一步操作。如果魔数不正确，将会拒绝执行该文件。第五个字节是用来表示文件位数，0x00表
          示无效文件，0x01表示是32位的，0x02表示是64位的。第六位表示字节序，0x00表示无效格式，0x01表示小端，0x02表示大端。第七位表示elf文件版本，一般是1，表示1.2版本，elf自1.2版
          本后再也没有更新。最后9位保留，一般填0。
          
          + e_type    2字节
          
          此字段表名文件类型：
            - ET_NONE    0        位置文件类型
            - ET_REL     1        可重定位文件
            - ET_EXEC    2        可执行文件
            - ET_DYN     3        动态链接库文件
            - ET_CORE    4        core文件（目前不支持，但保留此关键字）
            - ET_LOPROC  0xff00   特特定处理器文件扩展下边界
            - ET_HIPROC  0xffff   特特定处理器文件扩展上边界
            
          ET_LOPROC ~ ET_HIPROC (0xff00 ~ 0xffff)这一范围内的文件类型是为特定处理器而保留的，如果需要为某种处理器专门设定文件格式，可以从这一范围内选取一个做为标识。在以上已定义
          范围外的文件类型均为保留类型，留做以后可能的扩展。
          
          + e_machine  2字节
          
          此字段表示处理器体系结构：
            - EM_NONE        0 未知体系结构
            - EM_M32         1 AT&T WE 32100 
            - EM_SPARC       2 SPARC 
            - EM_386         3 Intel Architecture 
            - EM_68K         4 Motorola 68000 
            - EM_88K         5 Motorola 88000 
            - EM_860         7 Intel 80860 
            - EM_MIPS        8 MIPS RS3000 Big-Endian 
            - EM_MIPS_RS4-BE 10 MIPS RS4000 Big-Endian 
            - RESERVED       11 ~ 16
            
          在以上已定义范围外的处理器类型均为保留的，在需要的时候将分配给新出现的处理器使用。这里所定义的这些处理器类型名字都以 ET_为前缀，后接处理器名。在 ELF 文件中，处理器厂商可
          以定义一些自有的常量，这类常量的命名必须要遵守一定的规则，只要合乎这一规则，这些名字就都是合法的，ELF 文件解析器就需要能够识别他们。这种规则一般是以某种前缀（比如 DT_或             PT_）开头，加上处理器的名字（比如 M32），再加上_SPECIAL 后缀构成，比如DT_M32_SPECIAL。
          
          + e_version    4字节
          
          此字段表名文件版本号，0表示非法版本号，1表示当前版本号，以后出现新版本，则1成为历史版本号。
          
          + e_entry      8字节
          
          此字段表示进程（程序）入口虚拟地址，实际地址取决于操作系统分配，对于可执行程序文件来说，当 ELF 文件完成加载之后，程序将从这里开始运行；而对于其它文件来说，这个值应该是 0。
          
          e_phoff      8字节
          
          此字段表示程序头表(program header table)开始处在文件中的偏移量。如果没有程序头表，该值应设为 0。
          
          e_shoff      8字节
          
          此字段指明节头表(section header table)开始处在文件中的偏移量。如果没有节头表，该值应设为 0。
          
          e_flags      4字节
          
          此字段含有处理器特定的标志位。标志的名字符合”EF_machine_flag”的格式。对于 Intel 架构的处理器来说，它没有定义任何标志位，所以 e_flags 应该为0。
          
          e_ehsize     2字节
          
          表示elf文件头，即此数据结构大小，单位字节。
          
          e_phentsize  2字节
          
          程序头表每一项的大小，单位字节。
          
          e_phnum      2字节
          
          程序头表项数。
          
          e_shentsize  2字节
          
          节头表每一项大小，单位字节。
          
          e_shnum      2字节
          
          节头表项数。
          
          e_shstrndx   2字节
          
          节头表中与节名字表相对应的表项的索引。如果文件没有节名字表，此值应设置为 SHN_UNDEF。
          
    - 节头表
    
        * 介绍：
        
        * 数据结构：
           
           typedef struct
           {
             Elf64_Word	sh_name;		/* Section name (string tbl index) */
             Elf64_Word	sh_type;		/* Section type */
             Elf64_Xword	sh_flags;		/* Section flags */
             Elf64_Addr	sh_addr;		/* Section virtual addr at execution */
             Elf64_Off	sh_offset;		/* Section file offset */
             Elf64_Xword	sh_size;		/* Section size in bytes */
             Elf64_Word	sh_link;		/* Link to another section */
             Elf64_Word	sh_info;		/* Additional section information */
             Elf64_Xword	sh_addralign;		/* Section alignment */
             Elf64_Xword	sh_entsize;		/* Entry size if section holds table */
           } Elf64_Shdr;
         
         * 字段解析：
         
         
                     
          
    - 程序头表

        * 介绍：

          一个可执行文件或共享目标文件的程序头表(program header table)是一个数组，数组中的每一个元素称为“程序头(program header)”，每一个程序头描述了一个“段(segment)”或者一块用
          于准备执行程序的信息。一个目标文件中的“段(segment)”包含一个或者多个“节(section)”。程序头只对可执行文件或共享目标文件有意义，对于其它类型的目标文件，该信息可以忽略。在目
          标文件的文件头(elf header)中，e_phentsize 和 e_phnum 成员指定了程序头的大小。
    
        * 数据结构：
          typedef struct
          {
            Elf64_Word	p_type;			/* Segment type */
            Elf64_Word	p_flags;		/* Segment flags */
            Elf64_Off	p_offset;		/* Segment file offset */
            Elf64_Addr	p_vaddr;		/* Segment virtual address */
            Elf64_Addr	p_paddr;		/* Segment physical address */
            Elf64_Xword	p_filesz;		/* Segment size in file */
            Elf64_Xword	p_memsz;		/* Segment size in memory */
            Elf64_Xword	p_align;		/* Segment alignment */
          } Elf64_Phdr;
          
          
        * 字段解析：
        
          + p_type    4字节
          
          说明程序头所描述段的类型，除非有特别要求，否则所有程序头的段类型域 p_type 都是可选项，不是必须存在的。在所有程序头都不指定段类型的情况下，程序头表中所有的表项都不代表
          任何特别的类型，而只是作为一种索引，表明其相应的段的大小和位置。
          
              - PT_NULL      0             此类型表明本程序头是未使用的，本程序头内的其它成员值均无意义。
              - 
              - PT_LOAD      1             此类型表明本程序头指向一个可装载的段。段的内容会被从文件中拷贝到内存中。如前所述，段在文件中的大小是 p_filesz，在内存中的大小是                                                    p_memsz。如果 p_memsz 大于 p_filesz，在内存中多出的存储空间应填 0 补充，也就是说，段在内存中可以比在文件中占用空间更大；而相反，                                                p_filesz永远不应该比 p_memsz 大，因为这样的话，内存中就将无法完整地映射段的内容。在程序头表中，所有 PT_LOAD 类型的程序头按照 
                                           p_vaddr 的值做升序排列。
                                           
              - PT_DYNAMIC   2             此类型表明本段指明了动态连接的信息。
              - 
              - PT_INTERP    3             本段指向了一个以”null”结尾的字符串，这个字符串是一个 ELF 解析器的路径。这种段类型只对可执行程序有意义，当它出现在共享目标文件中时，是
                                           一个无意义的多余项。在一个 ELF 文件中它最多只能出现一次，而且必须出现在其它可装载段的表项之前。
                                           
              - PT_NOTE      4             本段指向了一个以”null”结尾的字符串，这个字符串包含一些附加的信息。
              - 
              - PT_SHLIB     5             该段类型是保留的，而且未定义语法。UNIX System V 系统上的应用程序不会包含这种表项。
              - 
              - PT_PHDR      6             此类型的程序头如果存在的话，它表明的是其自身所在的程序头表在文件或内存中的位置和大小。这样的段在文件中可以不存在，只有当所在程序
                                           头表所覆盖的段只是整个程序的一部分时，才会出现一次这种表项，而且这种表项一定出现在其它可装载段的表项之前。
                                           
              - PT_LOPROC    0x70000000    类型值在这个区间的程序头是为特定处理器保留的。
              - 
              - PT_HIPROC    0x7fffffff    类型值在这个区间的程序头是为特定处理器保留的。

          
          + p_flags    4字节
          
          此数据成员给出了本段内容的属性。

          + p_offset   8字节
          
          此数据成员给出本段内容在文件中的位置，即段内容的开始位置相对于文件开头的偏移量。
          
          + p_vaddr    8字节

          此数据成员给出本段内容的开始位置在进程空间中的虚拟地址。
          
          + p_paddr    8字节
          
          没点卵用
          
          + p_filesz   8字节

          给出本段内容在文件中的大小，单位是字节，可以是 0。
          
          + p_memsz
          
          本段内容在内容镜像中的大小，单位是字节，可以是 0。
          
          + p_align
          
          对于可装载的段来说，其 p_vaddr 和 p_offset 的值至少要向内存页面大小对齐。此数据成员指明本段内容如何在内存和文件中对齐。如果该值为 0 或 1，表明没有对齐要求；否则，
          p_align 应该是一个正整数，并且是 2 的幂次数。p_vaddr 和p_offset 在对 p_align 取模后应该相等。
          
          

          
          
          
          
          
          
          
        
                    
          
          
          
          
          
      
          
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
