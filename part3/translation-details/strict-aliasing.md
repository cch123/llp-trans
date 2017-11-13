14.6 Strict Aliasing

Beforerestrictwas introduced, programmers sometimes achieved the same effect by using different structure names. The compiler thinks that different data types imply that the respective pointers cannot point to the same data \(which is known as thestrict aliasing rule\).

The assumptions include the following:

* Pointers to different built-in types do not alias.

* Pointers to structures or unionswith different tagsdo not alias \(sostruct fooandstruct barare never used one for another\).

* Type aliases, created usingtypedef,canrefer to the same data.

* The typechar\*is exceptional \(signed or not\). The compiler always assumes thatchar\*can alias other types, butnotvice versa. It means that we can create acharbuffer, use it to get data, and then alias it as an instance of somestruct packet.

Breaking these rules can lead to subtle optimization bugs, because it triggers undefined behavior.

The example shown in Listing14-18, can be rewritten to achieve the same effect without therestrictkeyword. The idea is to use the strict aliasing rules to our benefit, packing both parameters into the structures with different tags.

Listing14-23shows the modified source.



Listing 14-23.restrict-hack.c

codecodecode



To our satisfaction, the compiler optimizes the reads away just as we wanted. Listing14-24shows the disassembly.

Listing 14-24.restrict-hack-dump

codecodecode



We discourage using aliasing rules for optimization purposes in code for C99 and newer standards becauserestrictmakes the intention more obvious and does not introduce unnecessary type names.





