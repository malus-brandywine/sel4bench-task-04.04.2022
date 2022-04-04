#### Task

In benchmark suite (sel4bench), benchmark "Signal to High Prio Thread" produces repeating pattern where every 8-th measurement
is considerably higher than others and makes standard deviation 150-160 ccycles for MCS kernel and 170-190 ccycles for traditional one when run on Armv7-A platform (Sabre).
Initial guess was that the issue related to eviction of a cache line. (Thesis Paper by Shade Kadish)

The task was to implement the new methodology which avoids writing a long array of raw data sequentially into memory.
Instead, while sampling processes goes on the four parameters are to be calculated:
   - sum of samples,
   - sum of squared samples,
   - min,
   - max

It would allow to calculate parameters variance, standard deviation and mean post-process and to drop raw data.


#### Analysis of raw data

I made a check of the raw data for the available architectures. </br>Parameters of measurements: 10/100 (10 warm-up samples, 100 recorded samples)


   - Armv7-A

The only problem found here was the mentioned "every 8-th" pattern. Other samples dispersed from each other in a limited way, 
but no other appearant patterns were found.

   - Armv8-A

The mentioned "every 8-th" pattern was not found here. On the contrary, every 8-th sample was few cycles less then neighbouring ones.
In addition, 2 - 3 tailing samples were considerably higher than others. Further experiments showed that the increased tailing samples
don't depend on number of measured samples.

   - Risc-V

"Every 8-th" pattern was not found here. Rather sporadic increased measurements or in some cases series of 2-3 increased measurements.
    
   - x86_64

For traditional kernel, there are number of sporadic increased measurements. No "every 8-th" or other patterns were not found.

For MCS kernel, there are first 22-25 recorded measurements are steadily high, more than 200 cycles. No "every 8-th" or other patterns were not found.


