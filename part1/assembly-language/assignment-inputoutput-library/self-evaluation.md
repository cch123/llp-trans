2.7.1 自我测验

为了避免在测试的时候碰到意料之外的结果，先快速检查一下下面的 checklist：



1.Labels denoting functions should be global; others should be local.

2.Youdonotassumethatregistersholdzero“bydefault.”  
3.Yousaveandrestorecallee-savedregistersifyouareusingthem.

1. Yousavecaller-savedregistersyouneedbeforecallandrestorethemafter.

2. Youdonotusebuffersin.data.Instead,youallocatethemonthestack,which allows you to adapt multithreading if needed.

3. Yourfunctionsacceptargumentsinrdi,rsi,rdx,rcx,r8,andr9.

4. Youdonotprintnumbersdigitafterdigit.Insteadyoutransformthemintostringsof

   characters and useprint\_string.

5. parse\_intandparse\_uintaresettingrdxcorrectly.Itwillbereallyimportantinthe

   next assignment.

6. All parsing functions andread\_wordwork when the input is terminated via Ctrl-D.

Done right, the code will not take more than 250 lines.

■Question 20try to rewriteprint\_newlinewithout callingprint\_charor copying its code. hint: read about tail call optimization.

■Question 21try to rewriteprint\_intwithout callingprint\_uintor copying its code. hint: read about tail call optimization.

■Question 22try to rewriteprint\_intwithout callingprint\_uint, copying its code, or usingjmp. you will only need one instruction and a careful code placement.

read about co-routines.

