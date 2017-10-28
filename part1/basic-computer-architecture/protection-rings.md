Protection rings are one of the mechanisms designed to limit the applications’ capabilities for security and robustness reasons. They were invented for Multics OS, a direct predecessor of Unix. Each ring corresponds to a certain privilege level. Each instruction type is linked with one or more privilege levels and is not executable on others. The current privilege level is stored somehow \(e.g., inside a special register\).

Intel 64 has four privilege levels, of whichonly twoare used in practice: ring-0 \(the most privileged\) and ring-3 \(the least privileged\). The middle rings were planned to be used for drivers and OS services, but popular OSs did not adopt this approach.

In long mode, the current protection ring number is stored in the lowest two bits of registercs\(and duplicated in those ofss\). It can only be changed when handling an interrupt or a system call. So an application cannot execute an arbitrary code with elevated privilege levels: it can only call an interrupt handler or perform a system call. See Chapter3“Legacy” for more information.

