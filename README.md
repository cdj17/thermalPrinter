# thermalPrinter
updated code for Adafruit thermal printer library

As described here: https://forums.adafruit.com/viewtopic.php?f=22&t=111655
there is a timing bug in the library that is used with the thermal printer sold by Adafruit.
The timeoutWait method in the Adafruit_Thermal.cpp file contains this line of code:

		    while((long)(micros() - resumeTime) < 0L); // (syntax is rollover-proof)

This code is necessary, but it is insufficient.  The timeoutWait routine is intended to pace the output to the printer in order to avoid overrunning printer's internal buffer.  The length of time that it takes to print a character is variable, and the time depends on how many dots are required to form the character.  After a character is sent to the printer, the required delay time is calculated and then used to calculate a "resume" time in microseconds.  The resume time is the counter value returned by the micros() routine beyond which it is ok to send the next character.  If the next character is ready to be sent to the printer before the resume time is reached, then the above line of code is intended to introduce a delay until the resume time is reached.

The problem occurs when the next character to be printed arrives after a long delay.  For instance, if a line of output is produced every few hours, then the time between the last char of the previous line and the first character of the next line is quite long, and also unpredictable.  If the time between characters to print is longer than 1/2 of the maximum value of a 32 bit microsecond counter, which translates to about 36 minutes, then there can be problems.  In the case of such a delay there is actually no need for executing the while loop at all before sending the newly arrived character to the printer.  

The problem can be fixed by adding a few lines of code to the timeoutSet and timeoutWait functions.  I do not know what the absolute character delay time limit is for the printer, so I selected an arbitrary value of 10 seconds as the maximum time that the while loop should ever run.  With the code below, the while loop is invoked only when the current time is 10 seconds or less behind the resume time (including rollover).  A calculation that yields a value of more than 10 seconds indicates a rollover condition whereby the current time is well past the resume time.    

I have tested the proposed code changes (see the previously referenced post in the Adafruit forum for more details).

The code posted below also includes new methods for setFontA and setFontB.  These methods are unrelated to the timing bug described above, but are desirable additions to the library.  I have also tested these proposed code changes.



