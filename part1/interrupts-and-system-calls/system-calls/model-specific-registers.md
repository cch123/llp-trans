6.3.1 Model-Specific Registers

Sometimes when a new CPU appears it has additional registers, which other, more ancient ones, do not have. Quite often these are so-called **Model-Specific Registers**. When these registers are rarely modified, their manipulation is performed via two commands:rdmsrto read them andwrmsrto change them. These two commands operate on the registeridentifying number.

rdmsr accepts the MSR number inecx, returns the register value inedx:eax.  
wrmsr accepts the MSR number inecxand stores the value taken fromedx:eaxin it.

