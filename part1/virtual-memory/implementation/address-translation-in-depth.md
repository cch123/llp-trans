4.7.2 Address Translation in Depth

图 4- 2 反映了地址的翻译过程。



Figure 4-2.Virtual address translation schematic  


First, we take the first table starting address from cr3. The table is calledPage Map Level 4 \(PML4\). Fetching elements from PML4 is performed as follows:

* Bits 51:12 are provided bycr3.

* Bits 11:3 are bits 47:39 of the virtual address.

* The last three bits are zeroes.



The entries of PML4 are referred as PML4E. The next step of fetching an entry from the Page Directory Pointer table mimics the previous one:



* Bits 51:12 are provided by selected PML4E.

* Bits 11:3 are bits 38:30 of the virtual address.

* The last three bits are zeroes.

The process iterates through two more tables until at last we fetch the page frame address \(to be precise, its 51:12 bits\). The physical address will use them and 12 bits will be taken directly from the virtual address.



Are we going to perform so many memory reads instead of one now?Yes, it does look bulky. However, thanks to the page address cache,TLB, we usually access memory on already translated and memorized pages. We should only add the correct offset inside page, which is blazingly fast.



As TLB is an associative cache; it is quickly providing us with translated page addresses given a starting virtual address of the page.



Note that translation pages can be cached for a faster access. Figure4-3specifies the Page Table Entry format.



图 4-2.Page table entry



IfPis not set, an attempt to access the page results in an interrupt with code \#PF \(Page fault\). The operating system can handle it and load the respective page. It can also be used to implement lazy file memory mapping. The file parts will be loaded in memory as needed.

The operating system usesWbit to protect the page from being modified. It is needed when we want to share code or data between processes, avoiding unnecessary doubling. Shared pages marked withWcan be used for data exchange between processes.

Operating system pages haveUbit cleared. If we try to access them from ring3, an interrupt will occur.In absence of segment protection the virtual memory is the ultimate memory guarding mechanism.



---

■On segmentation faults in general, segmentation faults occurs when there is an attempt to access memory with insufficient permissions \(e.g., writing into read-only memory\). in case of forbidden addresses we can consider them to have no valid permissions, so accessing them is just a particular case of memory access with insufficient permissions.

---

EXB\(also called NX\) bit forbids code execution. The DEP\(Data Execution Prevention\) technology is based on it. When a program is being executed, parts of its input can be stored in a stack or its data section. A malicious user can exploit its vulnerabilities to mix encoded instructions into the input and then execute them. However, if data and stack section pages are marked withEXB, no instructions can be executed from them. The.textsection, however, will remain executable, but it is usually protected from any modifications byWbit anyway.

