# 虛體地址（Virtual Addresses）



When a program accesses memory, it does not know or care where the physical memory backing the address is stored. It knows it is up to the operating system and hardware to work together to map locate the right physical address and thus provide access to the data it wants. Thus we term the address a program is using to access memory a _virtual address_. A virtual address consists of two parts; the page and an offset into that page.

#### 6.1 Page

Since the entire possible address space is divided up into regular sized pages, every possible address resides within a page. The page component of the virtual address acts as an index into the page table. Since the page is the smallest unit of memory allocation within the system there is a trade-off between making pages very small, and thus having very many pages for the operating-system to manage, and making pages larger but potentially wasting memory

#### 6.2 Offset

The last bits of the virtual address are called the _offset_ which is the location difference between the byte address you want and the start of the page. You require enough bits in the offset to be able to get to any byte in the page. For a 4K page you require (4K == (4 \* 1024) == 4096 == 212 ==) 12 bits of offset. Remember that the smallest amount of memory that the operating system or hardware deals with is a page, so each of these 4096 bytes reside within a single page and are dealt with as "one".

#### 6.3 Virtual Address Translation

Virtual address translation refers to the process of finding out which physical page maps to which virtual page.

When translating a virtual-address to a physical-address we only deal with the _page number_ . The essence of the procedure is to take the page number of the given address and look it up in the _page-table_ to find a pointer to a physical address, to which the offset from the virtual address is added, giving the actual location in system memory.

Since the page-tables are under the control of the operating system, if the virtual-address doesn't exist in the page-table then the operating-system knows the process is trying to access memory that has not been allocated to it and the access will not be allowed.

![](../.gitbook/assets/virtaddress.svg)

We can follow this through for our previous example of a simple _linear_ page-table. We calculated that a 32-bit address-space would require a table of 1048576 entries when using 4KiB pages. Thus to map a theoretical address of 0x80001234, the first step would be to remove the offset bits. In this case, with 4KiB pages, we know we have 12-bits (212 == 4096) of offset. So we would right-shift out 12-bits of the virtual address, leaving us with 0x80001. Thus (in decimal) the value in row 524289 of the linear page table would be the physical frame corresponding to this page.

You might see a problem with a linear page-table: since every page must be accounted for, whether in use or not, a physically linear page-table is completely impractical with a 64-bit address space. Consider a 64-bit address space divided into 64 KiB pages creates 264/216 = 252 pages to be managed; assuming each page requires an 8-byte pointer to a physical location a total of 252\*23 = 255 or 32 PiB of contiguous memory would be required just for the page table! There are ways to split addressing up that avoid this which we will discuss later.

