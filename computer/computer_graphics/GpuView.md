parentTopic:: [[Computer graphics]]

# How to capture
Steps to capture a GPUView trace:
1. Open CMD as admin
2. Go to GPUView location, usually `C:\Program Files (x86)\Windows Kits\10\Windows Performance Toolkit\gpuview`
3. Run `log.cmd`
4. Reproduce
5. Run `log.cmd`
6. Result is in `Merged.etl` file

Some notes:
- Windows Performance Recorder also produces `.etl` files, but they may not work (depending on WPR version) in GPUView due to new packet types.
- ETL file compress quite nicely, I've seen 9x compression ratio to `.zip`.
- Handy powershell script:
    ```powershell
    $gpu_view_path = "C:\Program Files (x86)\Windows Kits\10\Windows Performance Toolkit\gpuview"
    Start-Process "cmd" -ArgumentList "/K","cd $gpu_view_path && echo TIP: run log.cmd to capture GpuView" -Verb RunAs
    ```

# Main window
## Adapter section (blue)
Shows real hardware queues, that the GPU has. Either copy/compute/3D. Queues may be either software scheduled or hardware scheduled.

|                                            | SWS                                                                                         | HWS                                                                                                          |
| ------------------------------------------ | ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Name in adapter section                    | *Hardware queue*                                                                            | *Hardware scheduler*                                                                                         |
| Name in application section                | *Device context*                                                                            | *HW queue*                                                                                                   |
| Packet correlation between adapter and app | Possible, the same packets from per-app *Device context* show up in global *Hardware queue* | Impossible, global *Hardware scheduler* only has *Focus* packets, which cannot be tied to *HW queue* packets |
| See queue of packets                       | Yes, packets pile up in the *Hardware queue*                                                | No, only *Focus* packets in *Hardware scheduler*                                                             |
| Paging queues                              | Have data                                                                                   | Are empty                                                                                                    |
| Related events                             | `DxgKrnl DmaPacket - ClientRender` or `DxgKrnl DmaPacket - ClientPaging`                    | `DxgKrnl SchedulingLogContextStateChange`                                                                    |

![[GPUView0.png]]

Hardware scheduling can be disabled with Settings: `System - >Display -> Graphics -> Advanced Graphics Settings -> Hardware-accelerated GPU scheduling`. Reboot is required after change.

## Flip queue section (yellow)
I used to see this section but now I don't. It probably depends on presentation mode or some Windows API used. Present packets are instead usually seen in a separate per-app queue. Maybe it's composition vs exclusive fullscreen thing?

## Application sections (green)
Each application has its own software GPU queues and threads associated with it. It's best to first hide all processes and show only the one that we want to debug.

Some queues may be called "HW queue". That's a HWS queue tied to a "Hardware scheduler" queue in adapter section
Some queues may be called "Device context". That's an SWS queue tied to "Hardware queue" in adapter section.

GPUView docs talk about "Context CPU queue", but I've never encountered it. I think it just got renamed to "Device context", because the name was confusing. "CPU" might suggest it's a CPU work, but it's a GPU work that's waiting on CPU to be submitted.

## GPU queues
Semantics of GPU queues are often similar in both real hardware queue in adapter section and software queues in application section. New packets are put on the top of the queue. Packets gradually make their way to the bottom. When they're bottom-most packet, it means they're executing.

Standard queue packets are regular command buffers with work for the GPU. They have a solid color dependent on their assigned queue color key. Color key is visible on the left, below the queue name. They're created with `DxgKrnl QueuePacket Render` events.

Paging packets are red.

Present packets are diagonally dashed from left to right, top to bottom.

Monitored fence packets represent inter-queue dependencies. Signals are purple, waits are pink. When a packet is selected, it becomes crosshatched and so does every other packet (either signal or wait) that references the same monitored fence object. The packet that references the same fence value gets a black border. Packets are created with `DxgKrnl QueuePacket Signal-MonitoredFence` and `DxgKrnl QueuePacket Wait-MonitoredFence` events.

See example image below. The bottom queue waits for a fence, executes a command buffer and then signals two fences. The second signal is waited on by the top queue, which then executes a present packet. Notice only some fence packets are cross-hatched. Other ones reference other fence objects.

![[GPUView1.png]]


## Threads
Background color denotes a processor core. If all rectangles for a given thread are of the same color, then it's nice, the threads always runs on the same core. If colors are different, it's being switched between core. It's not necessarily a problem, it's a common thing.

Blue crosshatched regions are DPCs (deferred procedure calls)

Red crosshatched regions are hardware interrupts

Rectangles can have nested sub-rectangles, which represent Nested Execution Profiling. There are special APIs to use that to show time spent in some functions. Some colors are reserved:
- blue: video kernel code. 
- red: video driver kernel code

Little dots under the timeline are stack walks.