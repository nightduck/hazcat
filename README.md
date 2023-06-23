# Introduction

Hazcat is a zero-copy communication framework with heterogeneity awareness. Meaning, it can support zero-copy of data not only in host memory, but in device memory as well.

Zero copy is a shared memory approach for pub/sub systems, where messages can be sent without a single copy. Existing zero-copy middleware has used centralized daemons to manage this pool of shared memory, but Hazcat does so in a completely decentralized manner.

Additionally, Hazcat's awareness of device memory meaning components can be hardware accelerated without needing to concern themselves without copying data in and out before and after kernel runs. Hazcat is able to perform these copies contextually if needed. If a publisher is hardware accelerated, but a subscriber is not, a copy must occur during publication, as the subscriber is expecting the message to be in host memory, and dereferencing device memory would generate a segfault. In this case, Hazcat creates a duplicate copy of the message just for host memory. This copy can be re-used by additional subscribers later. If the subscriber is also hardware accelerated, no copies occur, and we avoid three unnessary copy operations: out of device memory, from publisher to subscriber, and back into device memory.

# Usage

The two key concepts in Hazcat are message queues and allocators.

## Message Queues

A message queue is analogous to a topic, and stores a cyclical queue of messages recently published to that topic. There are 3 ways to interact with a message queue:

* **register/unregister** - A message queue must be made aware of publishers and subscribers attached to the relevant topic. Calls to hazcat_register_publisher, hazcat_register_subscriber, hazcat_unregister_publisher, and hazcat_unregister_subscriber can perform this registration and event create and destroy message queues. This is also where publishers and subscriber indicate a preference for their memory domain
* **publish** - This is called by publishers to enqueue a message in the message queue
* **take** - This is called by subscribers to fetch a reference to an available message. If the message is in the wrong domain, this is where neccesary copy operations occur

## Allocators

Each publisher and subscriber needs an affiliated allocator. Each allocator its own algorithm and associated memory domain. There are 8 methods to interact with an allocator

* **constructor** - Actual creation of the allocator
* **remap** - When a different process needs to access a message created by this allocator, that allocator must be recreated in the new process's address space
* **unmap** - Removes this allocator from process address space. If this is last process to have this allocator, the allocator is destructed.
* **allocate** - Creates a new memory allocation of a specified size is created in the specified memory domain
* **deallocate** - Decrements a reference count and destroys a memory allocation if the count is zero
* **share** - Increments the reference count
* **copy_from** - Copies memory from allocator into another place in host memory
* **copy_to** - Copies memory from host memory into allocator's memory pool
* **copy** - Copies memory from this allocator into another allocator. This call is often just a wrapper for calling copy_from and copy_to in sequence.

## ROS2

An [RMW](https://github.com/nightduck/rmw_hazcat) has been written for Hazcat to implement it as a DDS layer in ROS2
