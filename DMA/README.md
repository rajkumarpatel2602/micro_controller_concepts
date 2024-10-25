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
- in this case, power consumption is reduced, and hence low power application make use of DMA controller. (ARM block will be completely turned off.) 

Why DMA is elegant?
- In MCU may have multiple master. e.g. ethernet, usb, dma, and actual main controller.
- master has capacity to move data on the bus from one place to another.
- Peripheral can't push data asynchronously from its data register to memory, it can only notify controller.

Bus master configuration
- 7 masters provided in discover board. I-BUS, D-BUS, S-BUS, DMA1, DMA2, USB bus.
- masters have controler over AHB bus.
- Peripheral and Memory buses are controlled by AHB(advnaced high speed) bus master port sitting inside controller.
- Slaves are connected over APB buses. Controllers have different level of busses, AHB for master, APB for peripherals.
- Anyways, these peripherals are connected to AHB via AHB-APB bridge.
- USB phy and Camera may need high speed and hence it may come under AHB2 buses as well, even if these are peripherals/slaves.
- Memories are also slaves, Flash is also slave (I code and D code buses of flash engines connected on bux matrix)

AHB bus matrix
- ![image](https://github.com/user-attachments/assets/ac8ec269-04aa-4223-94db-b70ff152c44e)
- DMA P1 is not touching bus matrix, and hence we're not considering DMA1_P1 as master.
- ![image](https://github.com/user-attachments/assets/b5254f1e-48a1-4be3-a3d5-96b92d0a6b13)
- ![image](https://github.com/user-attachments/assets/b5d59ff8-c453-48b1-ac81-ea81525dffe2)
- ![image](https://github.com/user-attachments/assets/914b0305-84c4-4eb6-a26f-4ce2eea64887) // bus matrix avoided here.
- ![image](https://github.com/user-attachments/assets/54bdab13-d2b9-423b-880b-5707883f8e43)
- Here if you see, DMA2 support memory to memory data transfer, as peripheral is also memory slave. but for dm1 it is only between peripheral slave and memory slave.
- ![image](https://github.com/user-attachments/assets/c648ddda-bdaa-4571-8d23-1f864c6cd3a6)
- ![image](https://github.com/user-attachments/assets/e64af958-88bc-43b9-8cb1-22fa4bfbf78c)
- Suppose SRAM1 is not present, and both peripheral wants to access SRAM2 at the same time, then there will be bus arbitration due to con-current access via 2 masters.
- Due to this, whichever master is having higher priority will acquire bus and go on performing operation. If DMA path is deprioritized, still it's better as it comes with internal FIFO to hold the data,
- and when bus access comes, this data can be safely placed in SRAM2

Excercise:
- Scenarion
- - without DMA, controller is running one loop and setting and resetting gpio in it and this on-off is surrounded by writing some fixed number of bytes to SRAM. SO controller is accessing SRAM, whereas uart gives interrupt on each byte reception to controller // UART_Recv_with_IT()
  -  whenever there is interrupt from uart, gpio ON/OFF time varied heavily. it reaches 20us from 15us in standard operation
  -  how it works?
     - in while(1), ARM will keep on writing to srams in a while loop and this writes are surrounded with gpio high/low
     - before this while(1) uart_recv_with_it() is called, and it writes to SRAM. which will call a call-back after 250 bytes reception, where another 250 bytes reception call is made.
     - there will be one more uartirq handler which gets called on each byte reception on the uart in its data register or fifo.
- - with DMA, 
  - Uart here won't give interrupt to controller, instead uart transfer for fixed bytees will happen with DMA, and when these many bytes will come, DMA interrupt will be called. // UART_Recv_with_DMA()
  - and inside that DMA handler will get called, where we will again recieve bytes from uart and transfer via DMA.
  - how it works?
    - DMA will do uart transfer and will let processor know. We have also uart handler callback call at 250bytes data transfer, where we are again getting data via DAM from uarat.
    - https://www.udemy.com/course/microcontroller-dma-programming-fundamentals-to-advanced/learn/lecture/9165056#overview
