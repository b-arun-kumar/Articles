# Valgrind - Memcheck - Memory leaks and Memory errors detector

This article introduces the basic usage of Valgrind's Memcheck tool.

Memcheck allows you to detect many memory-related errors which commonly lead to crashes and unpredictable behavior.

# Valgrind

- It is an instrumentation framework for building dynamic analysis tools.

- It is open source & free software.

- It runs on several popular platforms like x86/Linux, ARM/Linux, X86/Android, ARM/Android, etc.

- It works directly with binaries, therefore it works with any programming language.

- The Valgrind distribution includes seven production-quality tools and one experimental tool.

- The most popular of these tools is Memcheck.

# Memcheck

Memcheck is aimed primarily at C and C++ programs.

It detects memory-management problems:

- Invalid memory access (areas not yet allocated, areas that have been freed, areas past the end of heap blocks).

- Memory leaks.

- Bad frees of heap blocks (double frees, mismatched frees).

- Passing overlapping source and destination memory blocks to memcpy() and similar functions.

- Use of uninitialized values in decision making.

# Valgrind installation

**Method 1:**

    sudo apt-get install valgrind

**Method 2:**

1. Download Valgrind from the Valgrind download page: <https://www.valgrind.org/downloads/current.html>

2. Decompress and untar (XYZ is the version number):

       bzip2 -d valgrind-XYZ.tar.bz2
       
       tar -xf valgrind-XYZ.tar

3. This will create a directory called valgrind-XYZ. Change into that directory and run:

       ./configure
       
       make
       
       make install

# Run Memcheck

Valgrind uses dynamic binary instrumentation, so there's no need to modify, recompile or relink applications. Just prefix your command line with "valgrind" and everything works. Memcheck is the default tool when we run Valgrind.

1. Compile a C++ program (Creates a binary named sample):

       g++ -o sample -g sample.cpp

2. Analyze the binary using Valgrind (Shows the analysis report on the terminal):

       valgrind ./sample

3. To see the details of leaked memory and to redirect the report to a file:

       valgrind --leak-check=full --log-file=./report.txt ./sample

**sample.cpp:**

    #include <iostream>
    #include <stdlib.h>
    
    using namespace std;
    
    int main()
    {
    	// Allocate memory on heap for 10 ints (Valid)
    	int *a = (int*) malloc( 10 * sizeof(int) );
    	
    	// Uninitialized variable used in decision making (Invalid)
    	if ( a[0] != 1)
    	{
    		// Write to allocated memory (Valid)
    		a[0] = 5;
    	}	
    	
    	// Write to out of bound memory (Invalid)
    	a[10] = 5;
    	
    	// Free memory allocated by malloc using free (Valid)
    	free(a);
    	
    	// Writing to freed memory (Invalid)
    	a[0] = 5;
    	
    	// Freeing already freed memory (Invalid)
    	free(a);
    	
    	// Allocate memory on heap for 10 ints (Valid)
    	int *b = (int*) malloc( 10 * sizeof(int) );
    	
    	// Free memory allocated by malloc using delete (Invalid)
    	delete b;
    	
    	// Allocate memory on heap for 10 ints (Valid)
    	int *c = new int[10];
    	
    	// Free memory allocated by new[] using delete (Invalid)
    	delete c;
    	
    	// Allocate memory on heap for 10 ints (Valid)
    	int *d = (int*) malloc( 10 * sizeof(int) );
    	
    	// This allocated memory is not being freed (Invalid)
    	//free(d);
    	
    	// Allocate memory on stack for 10 ints (Valid)
    	int e[10];
    	
    	// Write to allocated memory (Valid)
    	e[0] = 5;	
    	
    	// Write to out of bound memory (Invalid)
    	e[10] = 5;
    	
    	return 0;
    }

**report.txt:**

    ==3512== Memcheck, a memory error detector
    ==3512== Copyright (C) 2002-2013, and GNU GPL'd, by Julian Seward et al.
    ==3512== Using Valgrind-3.10.1 and LibVEX; rerun with -h for copyright info
    ==3512== Command: ./sample
    ==3512== Parent PID: 2751
    ==3512== 
    ==3512== Conditional jump or move depends on uninitialised value(s)
    ==3512==    at 0x8048660: main (sample.cpp:12)
    ==3512== 
    ==3512== Invalid write of size 4
    ==3512==    at 0x8048673: main (sample.cpp:19)
    ==3512==  Address 0x434d050 is 0 bytes after a block of size 40 alloc'd
    ==3512==    at 0x402917C: malloc (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
    ==3512==    by 0x8048652: main (sample.cpp:9)
    ==3512== 
    ==3512== Invalid write of size 4
    ==3512==    at 0x8048689: main (sample.cpp:25)
    ==3512==  Address 0x434d028 is 0 bytes inside a block of size 40 free'd
    ==3512==    at 0x402A3D8: free (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
    ==3512==    by 0x8048684: main (sample.cpp:22)
    ==3512== 
    ==3512== Invalid free() / delete / delete[] / realloc()
    ==3512==    at 0x402A3D8: free (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
    ==3512==    by 0x804869A: main (sample.cpp:28)
    ==3512==  Address 0x434d028 is 0 bytes inside a block of size 40 free'd
    ==3512==    at 0x402A3D8: free (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
    ==3512==    by 0x8048684: main (sample.cpp:22)
    ==3512== 
    ==3512== Mismatched free() / delete / delete []
    ==3512==    at 0x402A838: operator delete(void*) (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
    ==3512==    by 0x80486B7: main (sample.cpp:34)
    ==3512==  Address 0x434d080 is 0 bytes inside a block of size 40 alloc'd
    ==3512==    at 0x402917C: malloc (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
    ==3512==    by 0x80486A7: main (sample.cpp:31)
    ==3512== 
    ==3512== Mismatched free() / delete / delete []
    ==3512==    at 0x402A838: operator delete(void*) (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
    ==3512==    by 0x80486D3: main (sample.cpp:40)
    ==3512==  Address 0x434d0d8 is 0 bytes inside a block of size 40 alloc'd
    ==3512==    at 0x4029DFC: operator new[](unsigned int) (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
    ==3512==    by 0x80486C3: main (sample.cpp:37)
    ==3512== 
    ==3512== 
    ==3512== HEAP SUMMARY:
    ==3512==     in use at exit: 40 bytes in 1 blocks
    ==3512==   total heap usage: 4 allocs, 4 frees, 160 bytes allocated
    ==3512== 
    ==3512== 40 bytes in 1 blocks are definitely lost in loss record 1 of 1
    ==3512==    at 0x402917C: malloc (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
    ==3512==    by 0x80486E0: main (sample.cpp:43)
    ==3512== 
    ==3512== LEAK SUMMARY:
    ==3512==    definitely lost: 40 bytes in 1 blocks
    ==3512==    indirectly lost: 0 bytes in 0 blocks
    ==3512==      possibly lost: 0 bytes in 0 blocks
    ==3512==    still reachable: 0 bytes in 0 blocks
    ==3512==         suppressed: 0 bytes in 0 blocks
    ==3512== 
    ==3512== For counts of detected and suppressed errors, rerun with: -v
    ==3512== Use --track-origins=yes to see where uninitialised values come from
    ==3512== ERROR SUMMARY: 7 errors from 7 contexts (suppressed: 0 from 0)

# Analyze Memcheck report

The report lists all the detected errors and summarizes the results.

**Error 1:**

    ==3512== Conditional jump or move depends on uninitialised value(s)
    ==3512==    at 0x8048660: main (sample.cpp:12)

This error indicates that an uninitialized value has been used in a condition.

Line 12:

    	// Uninitialized variable used in decision making (Invalid)
    	if ( a[0] != 1)

**Errors 2 & 3:**

    ==3512== Invalid write of size 4
    ==3512==    at 0x8048673: main (sample.cpp:19)
    ==3512==  Address 0x434d050 is 0 bytes after a block of size 40 alloc'd
    ==3512==    at 0x402917C: malloc (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
    ==3512==    by 0x8048652: main (sample.cpp:9)
    ==3512== 
    ==3512== Invalid write of size 4
    ==3512==    at 0x8048689: main (sample.cpp:25)
    ==3512==  Address 0x434d028 is 0 bytes inside a block of size 40 free'd
    ==3512==    at 0x402A3D8: free (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
    ==3512==    by 0x8048684: main (sample.cpp:22)

These errors indicate that we are writing to an illegal address.

Line 19:

    	// Write to out of bound memory (Invalid)
    	a[10] = 5;

Line 25:

    	// Free memory allocated by malloc using free (Valid)
    	free(a);
    	
    	// Writing to freed memory (Invalid)
    	a[0] = 5;

**Error 4:**

    ==3512== Invalid free() / delete / delete[] / realloc()
    ==3512==    at 0x402A3D8: free (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
    ==3512==    by 0x804869A: main (sample.cpp:28)
    ==3512==  Address 0x434d028 is 0 bytes inside a block of size 40 free'd
    ==3512==    at 0x402A3D8: free (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
    ==3512==    by 0x8048684: main (sample.cpp:22)

This error indicates that we are trying to free memory that has already been freed.

Line 28:

    	// Free memory allocated by malloc using free (Valid)
    	free(a);
    	
    	// Writing to freed memory (Invalid)
    	a[0] = 5;
    	
    	// Freeing already freed memory (Invalid)
    	free(a);

**Errors 5 & 6:**

    ==3512== Mismatched free() / delete / delete []
    ==3512==    at 0x402A838: operator delete(void*) (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
    ==3512==    by 0x80486B7: main (sample.cpp:34)
    ==3512==  Address 0x434d080 is 0 bytes inside a block of size 40 alloc'd
    ==3512==    at 0x402917C: malloc (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
    ==3512==    by 0x80486A7: main (sample.cpp:31)
    ==3512== 
    ==3512== Mismatched free() / delete / delete []
    ==3512==    at 0x402A838: operator delete(void*) (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
    ==3512==    by 0x80486D3: main (sample.cpp:40)
    ==3512==  Address 0x434d0d8 is 0 bytes inside a block of size 40 alloc'd
    ==3512==    at 0x4029DFC: operator new[](unsigned int) (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
    ==3512==    by 0x80486C3: main (sample.cpp:37)

These errors indicate that we are trying to deallocate memory in a way that isn't compatible with how it was allocated.

Line 34:

    	// Allocate memory on heap for 10 ints (Valid)
    	int *b = (int*) malloc( 10 * sizeof(int) );
    	
    	// Free memory allocated by malloc using delete (Invalid)
    	delete b;

Line 40:

    	// Allocate memory on heap for 10 ints (Valid)
    	int *c = new int[10];
    	
    	// Free memory allocated by new[] using delete (Invalid)
    	delete c;

**Error 7:**

    ==3512== 40 bytes in 1 blocks are definitely lost in loss record 1 of 1
    ==3512==    at 0x402917C: malloc (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
    ==3512==    by 0x80486E0: main (sample.cpp:43)

This error indicates that the memory allocated here has not been freed.

Line 43:

    	// Allocate memory on heap for 10 ints (Valid)
    	int *d = (int*) malloc( 10 * sizeof(int) );
    	
    	// This allocated memory is not being freed (Invalid)
    	//free(d);

**Undetected error:**

    	// Allocate memory on stack for 10 ints (Valid)
    	int e[10];
    	
    	// Write to allocated memory (Valid)
    	e[0] = 5;	
    	
    	// Write to out of bound memory (Invalid)
    	e[10] = 5;

Even though we are accessing out of bound memory, Valgrind didn't report this error.  
This is because Valgrind does not perform bounds checking on static arrays.

**Summary:**

    ==3512== HEAP SUMMARY:
    ==3512==     in use at exit: 40 bytes in 1 blocks
    ==3512==   total heap usage: 4 allocs, 4 frees, 160 bytes allocated
    ==3512== 
    ==3512== 40 bytes in 1 blocks are definitely lost in loss record 1 of 1
    ==3512==    at 0x402917C: malloc (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
    ==3512==    by 0x80486E0: main (sample.cpp:43)
    ==3512== 
    ==3512== LEAK SUMMARY:
    ==3512==    definitely lost: 40 bytes in 1 blocks
    ==3512==    indirectly lost: 0 bytes in 0 blocks
    ==3512==      possibly lost: 0 bytes in 0 blocks
    ==3512==    still reachable: 0 bytes in 0 blocks
    ==3512==         suppressed: 0 bytes in 0 blocks

The report summary indicates how much memory has been leaked.

**Conclusion:**

The Memcheck report has helped us find 7 memory errors in our program and has showed us that we are leaking 40 bytes of memory.

# Limitations of Valgrind

- For performing analysis, Valgrind adds instrumentation code to our program. Due to this, when the program is run with Valgrind, it consumes more memory (\~2x) and it slows down the program (\~10x to 50x).

- Valgrind does not perform bounds checking on static arrays (i.e., memory allocated on the stack).

# When to use Valgrind?

- After major changes, to ensure new bugs haven't been introduced.

- When a bug occurs, to get instant feedback about what the bug is, where it occurred, and why.

- Before a release, to ensure the release is as stable and as bug-free as possible.
