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
        
            - 在目标文件中可以包含很多“节”(section)，所有这些“节”都登记在一张称为“节头表”(section header table)的数组里。节头表的每一个表项是一个 Elf32_Shdr结构，通过每一个表
              项可以定位到对应的节。
              
            - 某些表项的索引值被保留，有特殊的含义。ELF 文件的节头表中不会出现索引值为以下各值的表项
            
                + SHN_UNDEF         0          该值被定义为 0，它表示一个未定义的、不存在的节的索引。尽管索引值 0 是一个未定义的保留值，但在节头表中的索引还是会从 0 开始。                        
                + SHN_LORESERVE     0xff00     被保留索引号区间的下限
                
                + SHN_LOPROC        0xff00     为处理器定制节所保留的索引号区间的下限
                
                + SHN_HIPROC        0xff1f     为处理器定制节所保留的索引号区间的上限
                
                + SHN_ABS           0xfff1     此节中所定义的符号有绝对的值，这个值不会因重定位而改变
                
                + SHN_COMMON        0xfff2     此节中所定义的符号是公共的，比如 FORTRAN COMMON 符号或者未分配的 C 外部变量
                
                + SHN_HIRESERVE     0xffff     被保留索引号区间的上限

            - 通常，目标文件中含有众多的“节”，“节”区是文件中最大的部分，它们需要满足下列这些条件:
            
                + 每一个节所占用的空间是连续的
                
                + 各个节之间不能互相重叠
                
                + 在目标文件中，节与节之间可能会存在一些多余的字节，这些字节不属于任何节
            
            
        
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
            
            + sh_name          4字节
            
            本节的名字，整个名字的字符串并不存储在这里，它仅是一个索引号，指向“字符串表”节中的某个位置，那里存储了一个以’\0’结尾的字符串。
            
            + sh_type          4字节
            
            本节的类型:
                - SHT_NULL         0                   此值表明本节头是一个无效的（非活动的）节头，它也没有对应的节。本节头中的其它成员的值也都是没有意义的
                
                - SHT_PROGBITS     1                   此值表明本节所含有的信息是由程序定义的，本节内容的格式和含义都由程序来决定。
                
                - SHT_SYMTAB       2                   这两类节都含有符号表。目前，目标文件中最多只能各包含一个这两种节，但这种限制以后可能会取消。一般来说，SHT_SYMTAB 提供的
                                                       符号用于在创建目标文件的时候编辑连接，在运行期间也有可能会用于动态连接。SHT_SYMTAB 包含完整的符号表，它往往会包含很多在
                                                       运行期间(动态连接)用不到的符号。所以，一个目标文件可以再有一个 SHT_DYNSYM 节，它含有一个较小的符号表，专门用于动态连接
                                                       
                - SHT_STRTAB       3                   此值表明本节是字符串表。目标文件中可以包含多个字符串表节
                
                - SHT_RELA         4                   此值表明本节是一个重定位节，含有带明确加数(addend)的重定位项，对于64位类型的目标文件来说，这个加数就是 Elf64_Rela。一
                                                       个目标文件可能含有多个重定位节
                
                - SHT_HASH         5                   此值表明本节包含一张哈希表。所有参与动态连接的目标文件都必须要包含一个符号哈希表。目前，一个目标文件中最多只能有一个哈
                                                       希表，但这一限制以后可能会取消。
                                                       
                - SHT_DYNAMIC      6                   此值表明本节包含的是动态连接信息。目前，一个目标文件中最多只能有一个 SHT_DYNAMIC 节，但这一限制以后可能会取消
                
                - SHT_NOTE         7                   此值表明本节包含的信息用于以某种方式来标记本文件
                
                - SHT_NOBITS       8                   此值表明这一节的内容是空的，节并不占用实际的空间。在定义 sh_offset 时提到过，这时 sh_offset 只代表一个逻辑上的位置概
                                                       念，并不代表实际的内容。
                                                       
                - SHT_REL          9                   此值表明本节是一个重定位节，含有带明确加数(addend)的重定位项，对于64位类型的目标文件来说，这个加数就是 Elf64_Rela。一
                                                       个目标文件可能含有多个重定位节
                                                       
                - SHT_SHLIB        10                  保留值，暂未指定语义
                
                - SHT_DYNSYM       11                  见上
                
                - SHT_LOPROC       0x70000000          为特殊处理器保留的节类型索引值的下边界
                
                - SHT_HIPROC       0x7fffffff          为特殊处理器保留的节类型索引值的上边界
                
                - SHT_LOUSER       0x80000000          为应用程序保留节类型索引值的下边界
                
                - SHT_HIUSER       0xffffffff          为应用程序保留节类型索引值的上边界
                
            - sh_flags         8字节
            
            本节的一些属性：
               + SHF_WRITE       0x1           本节所包含的内容在进程运行过程中是可写的
               + SHF_ALLOC       0x2           本节内容在进程运行过程中要占用内存单元。并不是所有节都会占用实际的内存，有一些起控制作用的节，在目标文件映射到进程空间时，并不需要
                                               占用内存
               + SHF_EXECINSTR   0x4           表示此节内容是指令代码
               + SHF_MASHPROC    0xf0000000    所有被此值所覆盖的位都是保留做特殊处理器扩展用的

            - sh_addr          8字节

            此成员指定映射的起始地址；如果不需要映射，此值为 0
            
            - sh_offset        8字节
            
            指明了本节所在的位置，该值是节的第一个字节在文件中的位置，即相对于文件开头的偏移量。单位是字节。如果该节的类型为 SHT_NOBITS 的话，表明这一节的内容是空的，节并不占用实
            际的空间，这时 sh_offset 只代表一个逻辑上的位置概念，并不代表实际的内容
            
            - sh_size          8字节
            
            指明节的大小，单位是字节。如果该节的类型为 SHT_NOBITS，此值仍然可能为非零，但没有实际的意义
            
            - sh_link          4字节

            此成员是一个索引值，指向节头表中本节所对应的位置。根据节的类型不同，本成员的意义也有所不同，具体见下表
            
            - sh_info          4字节
            
            此成员含有此节的附加信息，根据节的类型不同，本成员的意义也有所不同
            
                sh_type           sh_link                                           sh_info 
                SHT_DYNAMIC       用于本节中项目的字符串表在节头表中相应的索引值        0 
                SHT_HASH          用于本节中哈希表的符号表在节头表中相应的索引值        0 
                SHT_REL/SHT_RELA  相应符号表在节头表中的索引值，                      本重定位节所应用到目标节在节头表中的索引值
                SHT_SYMTAB/SHT_DYNSYM  相关字符串表的节头索引                             符号表中最后一个本地符号的索引值加 1 
                其它               SHN_UNDEF                                        0
                
            - sh_addralign     8字节
            
            此成员指明本节内容如何对齐字节，即该节的地址应该向多少个字节对齐。比如，在这个节中如果含有一个双字(doubleword)，系统必须保证整个节的内容向双字对齐。也就是说，本节内容
            在进程空间中的映射地址 sh_addr 必须是一个向sh_addralign 对齐，即能被 sh_addralign 整除的值。目前，sh_addralign 只能取 0、1或者 2 的正整数倍。如果该值为 0 或 1，表
            明本节没有字节对齐约束
            
            - sh_entsize       8字节
            
            有一些节的内容是一张表，其中每一个表项的大小是固定的，比如符号表。对于这种表来说，本成员指定其每一个表项的大小。如果此值为 0 则表明本节内容不是这种表格
                










         
         
                     
          
    - 程序头表

        * 介绍：

          - 一个可执行文件或共享目标文件的程序头表(program header table)是一个数组，数组中的每一个元素称为“程序头(program header)”，每一个程序头描述了一个“段(segment)”或者一块
          用于准备执行程序的信息。一个目标文件中的“段(segment)”包含一个或者多个“节(section)”。程序头只对可执行文件或共享目标文件有意义，对于其它类型的目标文件，该信息可以忽略。在
          目标文件的文件头(elf header)中，e_phentsize 和 e_phnum 成员指定了程序头的大小。
    
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
          
          

          
          
          
          
          
          
          
        
                    
          
          
          
          
          
      
          
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
