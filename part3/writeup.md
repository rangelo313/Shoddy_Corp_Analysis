#TESTING

I first began my part three by compiling with gcc:
gcc -g --coverage giftcardreaderfix.c -o giftcardreaderfix

After compiling, I moved my crash.gft files and hang.gft files to the part3 directory. 


I then ran the following line against my fixed giftcardreader file





./giftcardreaderfix 1 hang.gft && ./giftcardreaderfix 2 hang.gft && ./giftcardreaderfix 1 crash1.gft && ./giftcardreaderfix 2 crash1.gft && ./giftcardreaderfix 1 crash2.gft && ./giftcardreaderfix 2 crash2.gft

after this- I ran 
gcov giftcardreader #reads gcda file to produce 54% coverage.

File 'giftcardreaderfix.c'
Lines executed:53.39% of 171
Creating 'giftcardreaderfix.c.gcov'



 #######################################################
 Code Cov AFTER AFL
 #######################################################


I began this sectiob by compiling with 
 AFL_HARDEN=1 afl-gcc giftcardreaderfix.c -o giftcardreaderfix 

I then ran the fuzzer within afl against my inputs directory which contained gft files

Afterwards, I created an output dir

afl-fuzz -i inptest -o output -- ./giftcardreaderfix 1 @@

Cov1.gft
I intentionally made the type of record equal to one, allocated the default 116 bytes for this and set the number of records equal to five to increase coverage. By default my giftcardreaderfix will essentially cover most of the cases from crash assuming they compile successfully.

Cov2.gft
I intentionally gave this file more bytes, left the type of record equal to 3; set program argument 1 equal to (program[1] = 0x03 to execute instruction 3 and program[2] to 7 and successfully compiled

./giftcardreaderfix 1 cov1.gft #change record type or change variable to Increase coverage
./giftcardreaderfix 2 cov1.gft 
./giftcardreaderfix 1 cov2.gft #change type of record
./giftcardreaderfix 2 cov2.gft
./giftcardreaderfix 1 fuzz2.gft
./giftcardreaderfix 2 fuzz2.gft
./giftcardreaderfix 1 fuzz1.gf
./giftcardreaderfix 2 fuzz1.gft

File 'giftcardreaderfix.c'
Lines executed:54.97% of 171
Creating 'giftcardreaderfix.c.gcov'

Percentage increase went up to 54.97%


fuzz1.gft (Crash)
crashes through the indication that a gft could be passed into the program without error checking. after running in debug I found that this error occurred upon reading in  a new file until eof condition is met. As a result- the ret_val->num_bytes variable will point to garbage data as a 
giftcardwriter file was not taken to create a fictitious gft. Garbage data may point to a negative integer- very large number- or really anything, so to handle this, I checked for negative values within my fixed reader and produced an exit condition.



fuzz2.gft 
crashes through the indication that 0x09, 0x010, 0x03 instructions are executed because of an unsigned character being casted to a character; this can cause undefined behavior if the value passed in is negative- this can cause a hang, a crash, or overflow the variable. The immediate fix is to cast to an unsigned character in case nine to eliminate the chance of a loop or potential undefined behavior. 






