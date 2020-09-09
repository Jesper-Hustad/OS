## 1. The process abstraction

### 1.

We know that to start a new process/program the program is put in memory,  
the program counter is set to the first instruction,  
stack pointer is set to the base of the user stack.  
Finally there will of course be a switch from kernel to user mode


The benefits of going to user mode is that of security (this goes back to the core purpose of an OS). Meaning the code (that could be mallicious or buggy) can't take direct controll over the hardware.


### 2.

At `include/linux/sched.h` we can find task_struct defined on line 629 and the variable for the process ID and accumulated virtual memory respectively:

a)
```c
        /* PID/PID hash table linkage. */
840:	struct pid			*thread_pid
```

b)
```c
        /* Accumulated virtual memory usage: */
1047:	u64				acct_vm_mem1;
```

--------

By using `man top` we can find out what the fields stand for.  


The field PR stand for Priority of the task. We can find "prio" (that im guessing stand for priority) being assigned like this:

```c
675:	int				prio;
```

The TIME+ is the total  CPU  time  the  task  has  used  since it started. To calculate this time it's refrencing some start time, we can see such a variable being assigned here:
```c
    	/* Monotonic time in nsecs: */
873:	u64				start_time;
```

## 2. Process memory and segments


### 1.  

| Address space |
|------|
| Stack segment (growing down) |
| Heap  segment (growing up)|
| Data  |
| Text  |



### 2.
- **Text**
Location of the code that is run

- **Data**
Static variables explicitly initialized  
(BSS) Uninitialized static variables

- **Heap**
Dynamically allocated memory, using `malloc` `new` `free` `delete`

- **Stack**
Where the variables are located and generated automatically

Why 0x0 unavailable?  

If we try to read a pointer that has not been initialized/is null than we don't want it to suddenly be looking at some variable that happened to have that address. So the 0x0 address is reserved and gives a access vialoation. So it stops errors and bugs.

### 3.
Global variables can be accessed globally because they are initialized outside the main function while local variables are defined inside a function and will dissapear once that function is completed.

```c
#include<stdio.h>

int a = 100;

int main()
{
    {
        int a = 10;  
        printf("Local a = %d\n", a);
    }

    printf("Global a = %d\n", a);

    return 0;
}
```

``` c
Local a = 10
Global a = 100
```

For static variables i think this quote from geeksforgeeks sums it up:
>A static int variable remains in memory while the program is running. A normal or auto variable is destroyed when a function call where the variable was declared is over.

----
### Which segment does each of the variables (var1, var2, var3) belong to?
``` c
#include <stdio.h>
#include <stdlib.h>
int var1 = 0;
void main()
{

int var2 = 1;

int *var3 = (int *)malloc(sizeof(int));
// where is the pointer stored?
*var3 = 2;

printf("Address: %x; Value: %d\n", &var1, var1);
printf("Address: %x; Value: %d\n", &var2, var2);
printf("Address: %x; Address: %x; Value: %d\n", &var3, var3, *var3);
```
After running the code we see that the address of both var2 and var3 are very close to eachother, leading me to believe they are both stored in the stack. So even tough the address of var3 is a location in the heap, the variable that points to that address still resides in the stack.  

Var1 has a much lower address, from what we have just learned about the organisation of a process address space we can be pretty certain that it resides in data section.

## 3. Program code


1. Using the size tool we get:


| text | data | bss |
|------|------|-----|
| 1895 | 616  | 8   |

2. The start address for the program is: `0x00000000000005f0`

3. After we disassemle the compiled program we find a function called `_start`. After some web searching we find that it is the beginning part of the text section and that the _start tells the kernel where the program execution begins.

4. According to Wikipedia one of the reasons is for security (ASLR)

>Address space layout randomization (ASLR) is a computer security method which involves randomly arranging the positions of key data areas, usually including the base of the executable and position of libraries, heap, and stack, in a process's address space.

>Benefits are Address space randomization hinders some types of security attacks by making it more difficult for an attacker to predict target addresses. For example, attackers trying to execute return-to-libc attacks must locate the code to be executed, while other attackers trying to execute shellcode injected on the stack have to find the stack first. In both cases, the related memory addresses are obscured from the attackers. These values have to be guessed, and a mistaken guess is not usually recoverable due to the application crashing.

## 4. The stack
### 1 
Compiled succesfully
### 2 
Using `ulimit -s` i got a stack size of 8192 kb (about 8MB)

### 3
 After running the program the error was  
```
terminated by signal SIGSEGV (Address boundary error)
```
As the program ran the frame address decreased as seen by the address going trough like this this:

```c
func() frame address @ 0xc929eff0
...
func() frame address @ 0xc929eef0
...
                       0xc929edf0
...
                       0xc929ecd0
...
                       0xc929ebf0
...
                       0xc929ea90
...
                       0xc929e990
                                                    
```
The address is decreasing which is what we excpect as the stack grows downwards in address space. From the error it would seem there is no more place in the stack

### 4
`./stackoverflow | grep func | wc -l`  

After running the command we see that a total of 523685 lines are printed out before we get an error. The assignment says to check the relation between this number and the stack size. After doing `lines/stack size` i get 63.9264... ~ 64 

Tough i am not 100% sure what this means i know that 64 is a common computer number (2^6), maybe it shows that the computer is 64 bit?

### 5
By taking two consecutive functions frame address we can see how far away they are stored from eachother

```c
func() frame address @ 0xc26c47b0
//func() with localvar @ 0xc26c4784
func() frame address @ 0xc26c4790
```

**Hex value**:     `c26c47b0 – c26c4790 = 20`  
**Deciman value**: `261876144 – 3261876112 = 32`

So my best guess is that the stack memory is 32 bits -> 4 bytes. That is the same as in int, maybe that means something?


<!-- ## Conclusions
This is all very new to me, i may have misunderstood some obvious stuff. -->
