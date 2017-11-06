4.7.3 Page Sizes

The structure of tables of a different hierarchy level is very much alike. The page size may be tuned to be 4KB, 2MB, or 1GB. Depending on the structure, this hierarchy can shrink to a minimum of two levels. In this case PDP will function as a page table and will store part of a 1GB frame. See Figure4-4to see how the entry format changes depending on page size.



这里有两张图



Figure 4-4.Page Directory Pointer table and Page Directory table entry format  


This is controlled by the 7-th bit in the respective PDP or PD entry. If it is set, the respective table maps

pages; otherwise, it stores addresses of the next level tables.

