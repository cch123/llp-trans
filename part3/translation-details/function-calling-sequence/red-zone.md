14.1.4 Red Zone

The red zone is an area of 128 bytes that spans fromrspto lower addresses. It relaxes the rule “no data belowrsp”; it is safe to allocate data there and it will not be overwritten by system calls or interrupts. We are speaking about direct memory writes relative torspwithout changingrsp. The function calls will, however, still overwrite the red zone.

The red zone was created to allow a specific optimization. If a function never calls other functions, it can omit stack frame creation \(rbpchanges\). Local variables and arguments will then be addressed relative torsp, notrbp.

* The total size of local variables is less than 128 bytes.

* A function is a leaf function \(does not call other functions\).

* Function does not changersp; otherwise it is impossible to address memory relative to it.



By movingrspahead you can still get more free space to allocate your data in, than 128 bytes in the stack. See also section 16.1.3.



