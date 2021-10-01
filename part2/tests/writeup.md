<h1>Bugs</h1>

<h2>Crash case 1</h2>
Crash case one was created by setting the value of the num_bytes variable to a negative integer and making it exceed the maximum integer value. In this way, the line fread(ptr, ret_val->num_bytes, 1, input_fd); is not able to read from a value of num_bytes as a negative space to read is nonexistent and the error will read: Thread 2 received signal SIGSEGV, Segmentation fault - upon testing. Even if the user were to take the absolute value of the parameter, it will result in an integer overflow.  

#Fix for this case
Take the absolute value of the num_bytes variable, cast it to a double to allow for more space and continue to read in the file as normal or abort upon abnormal condition.

<h2>Crash Case 2</h2>
This case came to my attention as a result of the hint in the header file // BDG note: this program structure looks complex. And I'm worried
 //           about that comment that says programs must be "VALID"??  I also noticed that program was one of the largest undeclared variables within the writer.c file and I wanted to take advantage of this with fwrite(hang.program, 352,1,fd1); If you'll notice- I allocated more space to this than the program can account for, and this reflects in the file size (504b). As a result- the reader file will run  while (!feof(input_fd)) and while it appears to nearly infinitely loop while debugging (thought I had my hang case here), the while loop is constructed to only go as far as eof file indicator. Since the file is by far too large- this causes num_bytes to overflow to essentially garbage data (-520093696). When attempting to read on this particular line- we will find that fread(ptr, ret_val->num_bytes, 1, input_fd); is attemtping to read from a value of num_bytes that cannot exist (negative value) thus causing a segmentation fault.


#Fix
 The fix for this is to allocate the number of bytes appropriate for a file to exist with all of the required bytes; but in the reader file, I did a check before fread to determine whether the number of bytes allocated was appropriate and to ensure that it is not negative, therefore, during an overflow case, the file will proceed reading as normal

<h2>Hang Case</h2>
The hang case is an interesting one that took me a while to exploit. Taking a look at the switch in the while loop of the animate function, there are various cases accessible by setting the program variable correctly. 0x09 and 0x010 are both cases that will cause the pc variable to add the arg value. 0x010 uses a condition for zf variable, so I immediately went with 0x09. The unsigned char *pc counter  is supposed to read positive bytes. In this case, setting arg1 to negative will decrement the value of pc if it is set to a negative value, so the position will technically move backwards and the pc > program + 256 will never be met, thus causing an infinite loop, because the while loop condition is not going to be met where pc moves backwards by a negative number. In my case I used -6 to ensure that it would move backwards by 6 bytes. The main issue here is casting an unsigned char to a char.

#fix
Changed char to unsigned char or simply do not cast it. I inserted a cast to unsigned char for arg1 so it is slightly redundant but fixes the hang.
 

<h2>Crash Case 3</h2>
This crash case is important because it demonstrates out of bounds arrays can cause undefined behavior in a C program. For this, I passed into program[1] instruction hang.program[0] = 0x01; where the very first instruction in the switch in the animate function would be executed. From this, it attempts to assign the value of mptr to the regs array at index arg1: regs[arg1] = *mptr; If we attempt to assign arg1 as a value that is outside of the bounds of the regs array (limit 16 elements), then it can cause undefined behavior. I passed in the value 72 to cause a segmentation fault. A fix for this would be to check against the size of the arg1 variable prior to animate being called as I have done in my fix. Another way to do this is to statically ensure that at least 100 elements can fit in the array.

<h2>Crash Case 4</h2>
The last crash case that I could come up with was setting num_bytes variable to a negative number. This is so that fread(ret_val->numbytes) reads from an invalid location in memory (a page cannot be read from a negative value) and will cause segmentation fault. The immediate fix for this is checking the value of ret_val->numbytes prior to reading in the number of bytes.

#also included in part3 (further detail)
Cov1.gft
I intentionally made the type of record equal to one, allocated the default 116 bytes for this and set the number of records equal to five to increase coverage.

Cov2.gft
I intentionally gave this file more bytes, left the type of record equal to 3; set program argument 1 equal to (program[1] = 0x03 to execute instruction 3 and program[2] to 7 and successfully compiled


