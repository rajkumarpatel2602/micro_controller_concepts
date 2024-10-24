What is DMA?
- it's a controller to offload data copy operations between peripheral and memory or memory to memory.

How it helps?
- MCU needs to use load/store instruction to fetch data from peripheral and move to memory or vice versa.
- These instructions needs bus transactions, decoding units to be engaged and hence controller CPU to be enaged and endup in wasting CPU cycles, slower copying operation and more power consumption.
- For making memory transactions between memory and peripheral, peripheral lacks bus controlling capabilities, which are just with MCU MAster
- so designers came up with an idea to introduce one more master on the bus, which is DMA controller to make the data movement possible with no CPU cycle / cycle stealing.
- In standard flow, peripheral, e.g. uarat, interrupt controller on data reception. MCU stops execution and server interrupt, then it has to initiate load transaction on bus so that data gets copied to regiseter,
- then this data will get copied to memory from register, with store instruction.
- with DMA we get rid of such overhead, and uart can directly create interrupt to dma controller and it can initiate transaction data movement operation.
