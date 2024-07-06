
Array
Pointers
Function Pointer
Structure
Union


<h2>Pointers</h2>
Pointers store the address to which they are pointing.
taking memory blocks from heap is malloc() calloc() and alloc().
	1. malloc() only initializes the memory for you
	2. calloc() it initializes the memory with zeros for you
	3. realloc() can add more memory to the allocated memory. Howevrer, if the allocated memory has already been given to other piece of code then the starting address wont be the same
- lets write a small program to see how memory placement works in linux
```
  1 #include <stdio.h>
  2 #include <stdlib.h>
  3 int main()
  4 {
  5   char *cptr = NULL;
  6   //*cptr = 'Z';
  7   cptr = (char *)malloc(10);
  8   if (!cptr) return -1;
  9   printf("\n cptr = %p ",cptr);
 10   *cptr = 'A';
 11   printf("\n cptr = %c \n",*cptr);
 12   getchar();
 13   return 0;
 14 }
  // compile it "gcc test.c -o cptr -Wall"
```
- run this program which will give you an address space. In my case i got this address
```
 cptr = 0x55d9ef21a2a0 
 cptr = A 
```

-  As the program is still running search for it and find the process id by "ps aux | grep </name>" and then go to "/proc/<process id/>maps"
```
55d9eee88000-55d9eee89000 r--p 00000000 00:4c 6946983                    /home/sonic/Code/linux_training/cptr
55d9eee89000-55d9eee8a000 r-xp 00001000 00:4c 6946983                    /home/sonic/Code/linux_training/cptr
55d9eee8a000-55d9eee8b000 r--p 00002000 00:4c 6946983                    /home/sonic/Code/linux_training/cptr
55d9eee8b000-55d9eee8c000 r--p 00002000 00:4c 6946983                    /home/sonic/Code/linux_training/cptr
55d9eee8c000-55d9eee8d000 rw-p 00003000 00:4c 6946983                    /home/sonic/Code/linux_training/cptr
55d9ef21a000-55d9ef23b000 rw-p 00000000 00:00 0                          [heap]
7fce6a9e1000-7fce6a9e4000 rw-p 00000000 00:00 0 
7fce6a9e4000-7fce6aa0c000 r--p 00000000 fd:01 17433503                   /usr/lib/x86_64-linux-gnu/libc.so.6
7fce6aa0c000-7fce6aba1000 r-xp 00028000 fd:01 17433503                   /usr/lib/x86_64-linux-gnu/libc.so.6
7fce6aba1000-7fce6abf9000 r--p 001bd000 fd:01 17433503                   /usr/lib/x86_64-linux-gnu/libc.so.6
7fce6abf9000-7fce6abfa000 ---p 00215000 fd:01 17433503                   /usr/lib/x86_64-linux-gnu/libc.so.6
7fce6abfa000-7fce6abfe000 r--p 00215000 fd:01 17433503                   /usr/lib/x86_64-linux-gnu/libc.so.6
7fce6abfe000-7fce6ac00000 rw-p 00219000 fd:01 17433503                   /usr/lib/x86_64-linux-gnu/libc.so.6
7fce6ac00000-7fce6ac0d000 rw-p 00000000 00:00 0 
7fce6ac2a000-7fce6ac2c000 rw-p 00000000 00:00 0 
7fce6ac2c000-7fce6ac2e000 r--p 00000000 fd:01 17433320                   /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7fce6ac2e000-7fce6ac58000 r-xp 00002000 fd:01 17433320                   /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7fce6ac58000-7fce6ac63000 r--p 0002c000 fd:01 17433320                   /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7fce6ac64000-7fce6ac66000 r--p 00037000 fd:01 17433320                   /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7fce6ac66000-7fce6ac68000 rw-p 00039000 fd:01 17433320                   /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7ffd8585b000-7ffd8587c000 rw-p 00000000 00:00 0                          [stack]
7ffd859b8000-7ffd859bc000 r--p 00000000 00:00 0                          [vvar]
7ffd859bc000-7ffd859be000 r-xp 00000000 00:00 0                          [vdso]
ffffffffff600000-ffffffffff601000 --xp 00000000 00:00 0                  [vsyscall]

```
- and you can see "55d9ef21a000-55d9ef23b000 rw-p 00000000 00:00 0                          [heap]" is the place where it is located
- Heap keeps growing until its boundary which is set by brk system call which in our case is set until 55d9ef23b000. if it crosses the mentioned address then the signal will be send to interrupt the program and will exit

<h3>Using of free </h3>
```
  1 #include <stdio.h>
  2 #include <stdlib.h>
  3 int main()
  4 {
  5   int *iptr = NULL;
  6   printf("\n iptr = %p \n", iptr);
  7   iptr = (int *)malloc(sizeof(int)*10);
  8   free(iptr);
  9 
 10   int *cptr = NULL;
 11   printf("\n cptr = %p \n", cptr);
 12   // *cptr = 'Z';
 13   printf("\n size of *= %ld\n",sizeof(cptr));
 14   cptr = (char *)malloc(sizeof(int)*10);
 15   if (!cptr) return -1;
 16   printf("\n cptr = %p ",cptr);
 17   *cptr = 'A';
 18   for(int i=0;i<10;i++){
 19     printf("\n (cptr + %d) = %p",i,(cptr+i));
 20     *(cptr+i) = 'A' + i;
 21     printf(" *(cptr + %d) = %c\n",i,*(cptr+i));
 22   }
 23   free(cptr);
 24   for(int i=0;i<10;i++){
 25     printf("\n (cptr + %d) = %p",i,(cptr+i));
 26     *(cptr+i) = 'A' + i;
 27     printf(" *(cptr + %d) = %c\n",i,*(cptr+i));
 28   }
 29   printf("\n cptr = %c \n",*cptr);
 30   free(cptr);
 31   getchar();
       
```
  
> [!attention]+
> ```
>  - The free can only be used once at a time after allocation of the memory
>  -  If malloc is set to -1 the size will go beyond the system brk limit and it wont allocate
>  -  if malloc set to zero then it will still memory wihich is set by 
>  -  as pointer type is int but then it is  being allocated as a char the memory address increament will be 4 bytes which is as int
>  

 <h2> Malloc Errors </h2>
 ```
  1 #include <stdio.h>
  2 #include <stdlib.h>
  3 int main()
  4 {
  5   int *iptr = NULL;
  6   printf("\n iptr = %p \n", iptr);
  7   iptr = (int *)malloc(sizeof(int)*10);
  8   free(iptr);
  9 
 10   int *cptr = NULL;
 11   printf("\n cptr = %p \n", cptr);
 12   // *cptr = 'Z';
 13   printf("\n size of *= %ld\n",sizeof(cptr));
 14   cptr = (char *)malloc(sizeof(int)*10);
 15   if (!cptr) return -1;
 16   printf("\n cptr = %p ",cptr);
 17   *cptr = 'A';
 18   cptr = cptr + 10;
 19   for(int i=0;i<10;i++){
 20     printf("\n (cptr + %d) = %p",i,(cptr-i));
 21     *(cptr-i) = 'A' + i;
 22     printf(" *(cptr + %d) = %c\n",i,*(cptr-i));
 23   }
 24   //free(cptr);
 25 
 26   printf("\n cptr = %p \n",cptr);
 27   /*for(int i=0;i<10;i++){
 28     printf("\n (cptr + %d) = %p",i,(cptr+i));
 29     *(cptr+i) = 'A' + i;
 30     printf(" *(cptr + %d) = %c\n",i,*(cptr+i));
 31   }
 32   printf("\n cptr = %c \n",*cptr);
 33   free(cptr);*/
 34   getchar();
 35   return 0;
 36 }
~        

```
The code above is having a malloc error which i will explain now.
- first we are allocating total of +(10\*8(as computer is 64 bit== 8 bytes)) which totals to 80 bytes for the heap memory at line number 14
- we increase the heap size by adding 10 more which is 40 bytes