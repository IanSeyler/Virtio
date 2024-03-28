# Virtio Block


```
┌────────────┐                                                                                     
│Virtio Block│                                                                                     
├────────────┴──────────────────────────────────────────────┐                                      
│This assumes the following:                                │                                      
│                                                           │                                      
│- It is the first read on a freshly reset device           │                                      
│- Table/Ring memory has already been configured            │                                      
│- VirtQueue 0 is being used                                │                                      
└───────────────────────────────────────────────────────────┘                                      
												   
┌──────────────────────────────────────┐                                                           
│Add 3 entries to the Descriptor Table │                                                           
├──────────────────────────────────────┴───────────────┐  ┌──────┐                                 
│Entry 0                                               │  │Header│                                 
│                                                      │  ├──────┴────────────────────────────────┐
│64-bit Address -> Address of header                   │  │32-bit Type -> VIRTIO_BLK_T_IN         │
│32-bit Length -> 16                                   │  │32-bit Reserved -> 0                   │
│16-bit Flags -> VIRTQ_DESC_F_NEXT                     │  │64-bit Sector -> Starting sector number│
│16-bit Next -> 1                                      │  └───────────────────────────────────────┘
├──────────────────────────────────────────────────────┤                                           
│Entry 1                                               │                                           
│                                                      │                                           
│64-bit Address -> Address of where to store data      │                                           
│32-bit Length -> Number of bytes to read              │                                           
│16-bit Flags -> VIRTQ_DESC_F_NEXT | VIRTQ_DESC_F_WRITE│                                           
│16-bit Next -> 2                                      │                                           
├──────────────────────────────────────────────────────┤  ┌──────┐                                 
│Entry 2                                               │  │Footer│                                 
│                                                      │  ├──────┴──────────┐                      
│64-bit Address -> Address of footer                   │  │8-bit Status -> 0│                      
│32-bit Length -> 1                                    │  └─────────────────┘                      
│16-bit Flags -> VIRTQ_DESC_F_WRITE                    │                                           
│16-bit Next -> 3                                      │                                           
└──────────────────────────────────────────────────────┘                                           
												   
┌───────────────────────────┐                                                                      
│Add entry to Available Ring│                                                                      
├───────────────────────────┴──────────────────────────┐                                           
│16-bit Flags -> 1 (No interrupts)                     │                                           
├──────────────────────────────────────────────────────┤                                           
│16-bit Index -> 1                                     │                                           
├──────────────────────────────────────────────────────┤                                           
│Entry 0                                               │                                           
│                                                      │                                           
│16-bit Ring -> 0                                      │                                           
└──────────────────────────────────────────────────────┘                                           
												   
┌────────────────┐                                                                                 
│Notify the queue│                                                                                 
├────────────────┴────────────────────────┐                                                        
│xor eax, eax                    ; Queue 0│                                                        
│mov edx, [os_virtioblk_base]             │                                                        
│add dx, VIRTIO_QUEUESELECT               │                                                        
│out dx, ax                               │                                                        
│mov edx, [os_virtioblk_base]             │                                                        
│add dx, VIRTIO_QUEUENOTIFY               │                                                        
│out dx, ax                               │                                                        
└─────────────────────────────────────────┘                                                        
												   
┌─────────────────────┐                                                                            
│Inspect the Used Ring│                                                                            
├─────────────────────┴────────────────────────────────┐                                           
│16-bit Flags                                          │                                           
├──────────────────────────────────────────────────────┤                                           
│16-bit Index -> This should have incremented to 1     │                                           
├──────────────────────────────────────────────────────┤                                           
│Entry 0                                               │                                           
│                                                      │                                           
│32-bit Address                                        │                                           
│32-bit Length                                         │                                           
└──────────────────────────────────────────────────────┘                                           
												   
┌───────────────────────────────────────────────────────────────────────┐                          
│To read again:                                                         │                          
│                                                                       │                          
│- Create three new entries in the Description Table starting at Entry 3│                          
│- Set Entry 0 in the Available Ring to 3                               │                          
│- Kick the queue                                                       │                          
│- Inspect the Index of the Used Ring                                   │                          
└───────────────────────────────────────────────────────────────────────┘                          
```