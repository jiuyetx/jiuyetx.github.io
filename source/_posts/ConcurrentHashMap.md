---
title: ConcurrentHashMap
date: 2023-07-22 17:34:58
categories:
- Java
tags:
- ConcurrentHashMap
---
### Java集合 `ConcurrentHashMap`

`ConcurrentHashMap` 是 Java 并发包中的一个线程安全的哈希表实现，它扩展了 `HashMap` 并支持高并发环境下的并发操作。`ConcurrentHashMap` 允许多个线程同时读取和写入数据，而不需要显式地加锁。

#### 特点

1. **分段锁**：
   在 JDK 1.7 版本及之前，`ConcurrentHashMap` 使用了分段锁（Segment）的机制，将哈希表分成多个小的段（Segment），每个段都拥有一个独立的锁。不同的线程可以同时对不同的段进行并发操作，从而提高并发性能。但是在 JDK 1.8 版本之后，已经废弃了分段锁，采用了更加高效的 CAS（Compare and Swap）和 synchronized 实现。

2. **并发度**：
   `ConcurrentHashMap` 的并发度是指内部分段的个数。在 JDK 1.7 及之前，默认的并发度是 16，可以在构造函数中指定并发度。在 JDK 1.8 及之后，`ConcurrentHashMap` 不再提供构造函数指定并发度，而是采用了自动根据 CPU 核心数来设定并发度，通常是 CPU 核心数的 2 倍。

3. **无需加锁的读操作**：
   `ConcurrentHashMap` 的读操作不需要加锁，多个线程可以同时进行读取，提高了并发读的效率。

4. **适用场景**：
   `ConcurrentHashMap` 适用于多线程同时读写的场景，它可以在保持高并发性能的同时，提供线程安全的数据访问。

总的来说，`ConcurrentHashMap` 是一个线程安全的哈希表实现，它在高并发环境下表现出色，并且在 JDK 1.8 及之后的版本中进行了优化，不再使用分段锁，而是采用了更加高效的并发控制机制，使其在各种场景下都能保持较好的性能表现。
