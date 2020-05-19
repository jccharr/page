---
layout: default
title: Distributed Cloud Simulator
author: Jean-Claude Charr
---

# Cloud Model
## Cloud user
<p style="text-align: justify;text-justify: inter-word;"> 
A cloud user is a person or organization that has access to the cloud and is allowed on demand usage of some of the cloud's resources for a limited time and for a predefined fee (based on the time of usage and the number of used resources). The cloud user reserves resources on the cloud to execute an application.
</p>


## Cloud resources
<p style="text-align: justify;text-justify: inter-word;"> 
A cloud consists mainly of many homogeneous servers interconnected by an Ethernet network. Each server might have many processors and each processor consists of many cores. The servers might also have accelerators such as GPUs and TPUs. The cloud user expresses its resource requirements to execute a Task in terms of VCPUs and memory. A VCPU can be the equivalent of one core or one core-thread on a server. Some cloud service providers force users to select resources for their tasks form a predefined list of instances. Each instance has a predefined number of VCPUs and memory. 
Even though the user cannot choose over which server the resources are reserved to execute a given task, they are all located on the same server. The resource manager does not reserve resources on different servers for the same task. If a task requires more resources than those available on a server, it should be partitioned into two or more tasks which can be deployed in parallel on different servers and communicate over the network by exchanging messages or remote call procedures.
</p>

## Sleeping states of the computing resources
<p style="text-align: justify;text-justify: inter-word;"> 
Modern processors and cores have many PC and C sleeping states respectively. They are used to reduce their energy consumption when they are idle. When  a core is put to a C-state higher than C0, it stops executing instructions and turn off some of its components. When a core moves to a deeper C-State it consumes less energy but requires more time to wake up from this deep sleep state in order to execute some instructions. When all the cores of a given processor are in a CX sleeping state, the processor can be also put to the PCX sleeping state. The PC state of a processor cannot be higher than the minimum C-state of its cores. If all the processors of a server are in their last PC-State, the whole server can be turned off to save even more energy.
</p>

## Task
<p style="text-align: justify;text-justify: inter-word;"> 
A task is deployed on the reserved resources on the cloud and can contain a predefined set of stages to execute over these resources. Those stages can be one of the following: Execute a number of instructions, Read/Write an amount of data from memory or local disks, send and receive data over the network, etc. Before the execution of the task on the reserved resources, an adapted environment that satisfies the requirements of the task must be available on these resources. The most common two scenarios for this situation are: 
<ul>
  <li style="text-align: justify;text-justify: inter-word;">A general environment that contains the most common packages is already available on the resources and the task's dependencies are all satisfied by this environment which is rarely the case for complex tasks. </li>
  <li style="text-align: justify;text-justify: inter-word;">The user has to build an environment that contains all the application's requirements, starting from the selection of the operating system and its configuration to the installation of all the required packages and services such as execution environments, databases, libraries, etc. An image of this environment can be saved into a file to be reused and deployed over the reserved resources before the execution of the application. Most of the time on the cloud, a system image is deployed in a virtual machine because the resources of the cloud's servers might be shared between many applications requiring different environments.</li>
  </ul>
</p>

## Stage
<p style="text-align: justify;text-justify: inter-word;"> 
When executing a task over a processor, the processor will execute sequentially the different instructions of the task. The instructions of the task are defined by the developer of the task using high level programming language such as Java or C, etc. However, to execute these instructions on a processor they must be first transformed into machine instructions such as load and store data or arithmetic operations over data in the registers. A single instruction in a high-level language might be transformed into many machine instructions. In theory, a modern processor can execute on average more than one instruction per cycle because it uses pipelining and has many execution units. In practice, this could be the case if all the data required to execute the arithmetic instructions is already in the registers of the processor and no memory reads are required. At the beginning of the task all the data is on the memory (RAM) and even the task's instructions are also on the memory. Therefore, the processor has to load the data to the registers of the processor before applying the task's arithmetic operations over the data. Moreover, since the processor has a limited number of registers, it cannot load all the task's data to its registers. It can on the other hand prefetch some data and store them in its cache memory which is faster to access than RAM but has also a very limited size.  Memory access is very expensive, about 100 processor cycles to fetch a word from memory because the clock rate of the memory bus is much slower than the clock rate of the processor. If the processor is executing the instructions in order, it has to wait for the reception of the data before executing the next instruction and it would be idle during 100 cycles. In modern processors, out of order execution is possible for independent instructions. A processor can execute future independent instructions while waiting for IO. However, if the future instructions depend on the fetched data the execution of the task is blocked until receiving the data. Therefore, the execution of a task consists of  very small alternating CPU burst periods and CPU idle periods as shown in the following figure. If two tasks are being executed, the scheduler stops the task waiting for data and allocate the CPU cycles to the second task if its ready for execution.  
</p>
<p align="center">
<img alt="CPU burst and idle periods while executing a task" src="images/cpu_burst.svg"/><br/>
CPU burst and idle periods while executing a task.
</p>
<p style="text-align: justify;text-justify: inter-word;"> 
To make it easy to model a task execution, the CPU burst periods are regrouped together to form a large stage of computation and CPU idle periods are also regrouped together to form a large stage of Memory access. Therefore, a task can be assumed to consists of different large stages of similar instructions instead of very small alternating stages. In other words, instead of defining a computation stage for each arithmetic operation and an IO stage for each data transfer instruction, one large computation stage including all the arithmetic operations of a task and one large IO stage including all the IO memory operations are defined. Of course, many synchronous communication stages can be added to a task and between the communication stages, large stages of computation and IO can exist as shown in the following figure. 
</p>
<p align="center">
<img alt="Difference between a real task and a modeled task" src="images/stages.jpg"/><br/>
</p>

## Example of a simple task
<p style="text-align: justify;text-justify: inter-word;"> 
A Task can be directly defined in the Java code or it can be generated by another script as an XML file. The latter method is prefered because it allows the user of the simulator to simulate the execution of many sets of tasks without any modification to the simulator's code and without requiring any new compilation of code nor re-deployment of the byte code. In the following code, an XML example of the definition of a task with multiple stages. The task's description file starts with a description of the virtual machine that should be booted before executing the task. The size of the image is given in MB and it is used to compute the transfer time of the image to the server where it should be booted. The number of instructions to boot and to turn off the virtual machine are also given and used to estimate the virtual machine boot and turn off times. The parallel parameter is set to true when all the reserved VCPUs can be used to turn on/off the virtual machine. Otherwise, just one VCPU can be used to execute these instructions. After the virtual machine's description, the requirements of the task are specified in the instance node. The most important values to specify are the number of VCPUs ans the amount of memory. For the current version of the simulator a VCPU is the equivalent of a physical core. in this example, the task requires two cores and 4000 MB of memory to run. Finally, the different stages of the task are described. For each stage, the type of the stage should be given (EXEC for computation, IO_MEMORY for memory access, etc.) and some values related to the type of the stage are also required, for example the number of instructions (size tag) for the EXEC stage, the size of data to read/write (size tag) for the IO_MEMORY stage, etc. The order of the stages in the description XML file will be respected by the simulator.
</p>

```xml
<task>
  <virtualMachine>
      <os>Debian</os>
      <size>750</size>
      <instructionToBoot>50000</instructionToBoot>
      <instructionsToTurnOff>50000</instructionsToTurnOff>
      <parallel>true</parallel>
  </virtualMachine>
  <instance>
    <name>a.small</name>
    <nbVCPUs>2</nbVCPUs>
    <memory>4000</memory>
  </instance>
  <stage>
    <type>IO_MEMORY</type>
    <size>7536478</size>
  </stage>
  <stage>
    <type>EXEC</type>
    <size>51145</size>
  </stage>
  <stage>
    <type>IO_MEMORY</type>
    <size>11381876</size>
  </stage>
</task>
```


## Multi-threaded task
<p style="text-align: justify;text-justify: inter-word;"> 
A task that requires many VCPUs cannot fully use these VCPUs unless it is multi-threaded. Therefore, the task has a main process that starts when the virtual machine is booted. This main process has a predefined set of stages to execute as defined in the previous paragraph. However, in order to use more than one VCPU at a time, the main process can create new processes with their own sets of stages to execute. These processes can be executed in parallel and exchange data using shared memory. To execute a number of instructions, a process has to lock a VCPU during the execution and free it at the end. If all the VCPUs are taken, a process can force another process to stop its execution and release its VCPU, if it has a higher priority. Otherwise, the process has to wait for a VCPU to be freed in order to execute its instructions. The following XML code describes the main process that creates a new process after executing two compute and I/O stages. In this example, the new process has to execute just one compute stage containing 101975 instructions on another VCPU. The main process keeps on executing its following stages in parallel with the new created process.
  
</p>

```xml
<stage>
  <type>IO_MEMORY</type>
  <size>7536478</size>
</stage>
<stage>
  <type>EXEC</type>
  <size>51145</size>
</stage>
<stage>
  <type>CREATE_PROCESS</type>
  <stage>
    <type>EXEC</type>
    <size>101975</size>
  </stage>
</stage>
<stage>
  <type>EXEC</type>
  <size>102073</size>
</stage>
```

## Service task
<p style="text-align: justify;text-justify: inter-word;"> 
A service task does not have a predefined set of stages to execute. However, it is a service that is turned on when the virtual machine is booted. The service listens over a predefined port and waits for the reception of independent transactions. When the service receives a transaction, it creates a new process to handle this transaction. A transaction consists of a set of stages to execute. When all the stages are executed, the process is terminated. The task service can only be turned off by the user that deployed it or by the cloud manager if the resource reservation time has expired. The following XML code describes a transaction. The size and responseSize tags contain respectively the size of the transaction's data and the size of the transaction's response. Both values will be used to compute the transmission time over the network. The submission time is when the transaction will be submitted to the cloud. This value could be computed according to a chosen probability distribution that best suits the objectives of the experiment. The stages of a transaction are similarly described as those of a regular task.
</p>
```xml
<transaction>
  <size>10</size>
  <responseSize>10</responseSize>
  <submissionTime>212.1035258127757</submissionTime>
  <stage>
    <type>IO_MEMORY</type>
    <size>1344355</size>
  </stage>
  <stage>
    <type>EXEC</type>
    <size>2083293310</size>
  </stage>
</transaction>
```

## Application
<p style="text-align: justify;text-justify: inter-word;"> 
An application consists of many tasks which can be multi-threaded or service tasks. While the normal tasks of an application with predefined set of stages can communicate via the main process on each task, the service tasks of an application do not exchange any data because the transactions are independent. However, if the same service is deployed by many tasks of an application, in other words the service tasks of an application handle the same type of transactions, a scheduler can be created at the upper level and it will be the point of entry of the transactions. Once the scheduler receives a transaction it will submit it to one service task according to a scheduling policy. 
</p>

# Simulated cloud infrastructure
<p style="text-align: justify;text-justify: inter-word;"> 
Warehouse-scale computers (WSCs) are the building blocks of a cloud infrastructure. A WSC consists of tens of thousands of
servers and has a hierarchical organization as shown in the following figure. A rack consists of S servers, each server has P processors, each processor has C cores, and each core has T threads. A number R of racks form a cell and the WSC consists of N cells. Each node in the hierarchy is controlled by many software modules called managers which handle the requests of the users and manage the physical components of the simulated cloud.   These managers compose the logical infrastructure of the cloud  and their various roles are explained in the following sections.
</p>
<p align="center">
<img alt="The organization of a Warehouse-Scale Computer" src="images/WSC.jpg" width="80%"/>
</p>

## Warehouse manager
<p style="text-align: justify;text-justify: inter-word;">
It is the entry point of a WSC. It receives the applications submitted by clients. For each application, it checks if there are enough resources to execute all the tasks of the submitted application. The requirements of each task in the application are expressed in number of VCPUs and they must be reserved on the same server because a VM can only be hosted on a single server. If the WSC has enough resources the required resources are reserved and for each task a VM is deployed with the necessary resources to execute the task. Otherwise, the application is added to the waiting list until enough resources are freed up to satisfy the application's requirements.
</p>

## Cell manager
<p style="text-align: justify;text-justify: inter-word;">
  It manages a number of racks. It is the link between the warehouse manager and the racks managers. All the warehouse requests for resource reservation and tasks submissions are transferred via the Cell manager to the racks.
</p>

## Rack manager
<p style="text-align: justify;text-justify: inter-word;">
It handles the servers in a rack. All the warehouse requests for resource reservation and tasks submissions are transferred from the Cell manager to the servers via the Rack manager. If more than one identical service tasks were deployed on the servers of one rack, the Rack manager can host a dispatcher to dispatch the transactions to the service tasks according to a predefined policy. Therefore, the Rack manager will become the entry point of the transactions submitted to the service tasks.
</p>

## Server manager
<p style="text-align: justify;text-justify: inter-word;">
  
</p>

# Install

# Getting started

