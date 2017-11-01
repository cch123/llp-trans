3.4.3 Explanation

Now back to the example shown in Listing3-3. Let’s think about instruction decoding. The part of a CPU called instruction decoder is constantly translating commands from an older CISC system to a more convenient RISC one. Pipelines allow for a simultaneous execution of up to six smaller instructions.  


To achieve that, however, the notion of registers should be virtualized. During microcode execution, the decoder chooses an available register from a large bank of physical registers. As soon as the bigger instruction ends, the effects become visible to programmer: the value of some physical registers may be copied to those, currently assigned to be, let’s say,rax.

The data interdependencies between instructions stall the pipeline, decreasing performance. The worst cases occur when the same register is read and modified by several consecutive instructions \(think aboutrflags!\).

If modifying eax means keeping upper bits of rax intact, it introduces an additional dependency between current instruction and whatever instruction modifiedraxor its parts before. By discarding upper 32 bits on each write toeaxwe eliminate this dependency, because we do not care anymore about previousraxvalue or its parts.

This kind of a new behavior was introduced with the latest general purpose registers’ growth to 64 bits and does not affect operations with their smaller parts for the sake of compatibility. Otherwise, most older binaries would have stopped working because assigning to, for example,bl, would have modified the entireebx, which was not true back when 64-bit registers had not yet been introduced.

