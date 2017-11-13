14.8.1 Security Cookie

Thesecurity cookie\(stack guard, canary\) is supposed to protect us from arbitrary code execution by forcing abnormal program termination once the return address is changed.

The security cookie is a random value residing in the stack frame near the savedrbpand return address.



14-e

Overrunning the buffer will rewrite the security cookie. Before theretinstruction, the compiler emits a special check that verifies the integrity of the security cookie, and if it is changed, it crashes the program. Theretinstruction does not get to be executed.

Both MSVC and GCC have this mechanism turned on by default.

