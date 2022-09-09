---
title: JVM runtime system and garbage collection
date:  2019-07-18 12:00:00
tags:
- java
---
> A summary of the Java memory model and how garbage collection works. (JDK8)

Internal Architecture of JVM
![internal-details-of-jvm](/img/2019/7/internal-details-of-jvm-f55be87d027b4b1f92a10329a8ceaafd.png)

Java was developed with the concept of WORA (Write Once Run Anywhere), which runs on a VM. The compiler compiles the Java file into a Java .class file, then that .class file is input into the JVM, which Loads and executes the class file.

## Class Loader Subsystem

Class loader subsystem loads, links and initializes the class when it refers to a class for the first time at runtime, not at compile-time.

It performs three major functionality such as **Loading**, **Linking**, and **Initialization**.

##### Loading
![classLoaderDelegationAlgo](/img/2019/7/classLoaderDelegationAlgo-fc59f73a39da43499af86e7de63a6773.png)
The above Class Loaders will follow Delegation Hierarchy Algorithm while loading the class files-


##### Linking
Verify – Bytecode verifier will verify whether the generated bytecode is proper or not if verification fails we will get verification error

Prepare – For all static variables memory will be allocated and assigned with default values.

Resolve – All symbolic memory references are replaced with the original references from Method Area.

##### Initialization
Initialization This is the final phase of Class Loading, here all static variable will be assigned with the original values and static block will be executed.


## Java runtime data areas
- Privately created by thread：
   - PC Register
     PC register contains the address of the Java Virtual Machine instruction currently being executed. (Null when native method)
   - VM stack
     JVM stack stores frames which are pushed and popped out of stack, a JVM stack is never manipulated directly. A new frame is created when a method is invoked, this frame is then pushed into the JVM stack for the thread. Each frame has its own array of **local variables**, its own **operand stack** and a **reference to the run-time constant pool** of the class of the current method.
   - Native method stack
     
- Shared by threads
   - Heap
     Heap is the JVM run-time data area from which memory is allocated to objects, instance variables and arrays.
   - Method area (MetaSpace)
     Method area stores meta data about the loaded classes and interfaces. It stores per-class structures such as the run-time constant pool, field and method data, and the code for methods and constructors.
   - Run-time constant pool
     Constant_pool contains constants (string literals, numeric literals) which are known at compile-time, it also stores method and field references that must be resolved at run time.


## Java memory structure
![jvm-memory-allocations1](/img/2019/7/jvm-memory-allocations1-f1dfae31ed5f4dd09757b42cce4d3776.png)
- Heap Memory: JVM uses this memory to store objects. This memory is in turn split into two different areas called the “Young Generation Space” and “Tenured Space“. The Young Generation or the New Space is divided into two portions called “Eden Space” and “Survivor Space".
- Meta Space: This memory is out of heap memory and part of the native memory. This space is used to store the class definitions loaded by class loaders. It is also used to store package, method, field, bytecode, constant pool.
- Code Cache: JVM has an interpreter to interpret the byte code and convert it into hardware dependent machine code. As part of JVM optimization, the Just In Time (JIT) compiler has been introduced. The frequently accessed code blocks will be compiled to native code by the JIT and stored it in code cache. The JIT compiled code will not be interpreted.

## Execution Engine and Garbage Collector

Execution engine execute the .class (bytecode). It reads the byte-code line by line, use data and information present in various memory area and execute instructions. It can be classified in three parts :

- Interpreter : It interprets the bytecode line by line and then executes. The disadvantage here is that when one method is called multiple times, every time interpretation is required.
- Just-In-Time Compiler(JIT) : It is used to increase efficiency of interpreter.It compiles the entire bytecode and changes it to native code so whenever interpreter see repeated method calls,JIT provide direct native code for that part so re-interpretation is not required,thus efficiency is improved.
- Garbage Collector : It destroy un-referenced objects.

#### GC algorithmes
##### Marking Reachable Objects
First, GC defines some specific objects as Garbage Collection Roots. Examples of such GC roots are:

- Local variable and input parameters of the currently executing methods
- Active threads
- Static field of the loaded classes
- JNI references

Next, GC traverses the whole object graph in your memory, starting from those Garbage Collection Roots and following references from the roots to other objects, e.g. instance fields. Every object the GC visits is marked as alive.

##### Removing Unused Objects
Removal of unused objects is somewhat different for different GC algorithms but all such GC algorithms can be divided into three groups: sweeping, compacting and copying. 

- Sweep
Mark and Sweep algorithms use conceptually the simplest approach to garbage by just ignoring such objects. What this means is that after the marking phase has completed all space occupied by unvisited objects is considered free and can thus be reused to allocate new objects.

- Compact
Mark-Sweep-Compact algorithms solve the shortcomings of Mark and Sweep by moving all marked – and thus alive – objects to the beginning of the memory region. The downside of this approach is an increased GC pause duration as we need to copy all objects to a new place and to update all references to such objects. The benefits to Mark and Sweep are also visible – after such a compacting operation new object allocation is again extremely cheap via pointer bumping. Using such approach the location of the free space is always known and no fragmentation issues are triggered either.

- Copy
Mark and Copy algorithms are very similar to the Mark and Compact as they too relocate all live objects. The important difference is that the target of relocation is a different memory region as a new home for survivors. Mark and Copy approach has some advantages as copying can occur simultaneously with marking during the same phase. The disadvantage is the need for one more memory region, which should be large enough to accommodate survived objects.

#### GC usage of algo
![jvm-gc-impl-all-algo](/img/2019/7/jvm-gc-impl-all-algo-bbbc9c26592a4899bae13370af4ee569.png)

#### GC Conditions
- Minor GC: when Eden area is full
- Full GC: 
  1. System.gc()
  2. Old Generation is not sufficient (92%)
  3. MetaSpace is not sufficient
  4. Promoted Objects after MinorGC is greater than the rest of Old Generation area
  5. When copy from Eden, From space to To space, objects are bigger than To space, then transferred to old generation, but still bigger than the rest of Old Generation area

#### GC Tuning
- Latency
- Throughput

1. With ParallelGC, Throughput will be the priority. If we could allow MaxGCPause bigger than 1s, prefer ParallelGC.
2. With CMS(Concurrent Mark Sweep) or G1, Latency wiil be the priority. If we cannot allow MaxGCPause bigger than 1s, prefer CMS.

  -XX:MaxGCPauseMillis
  -XX:GCTimeRatio
  -Xmx 
