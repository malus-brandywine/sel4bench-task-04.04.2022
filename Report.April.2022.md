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


</br>


#### Analysis of raw data

I made a check of the raw data for the available architectures. </br>Parameters of measurements: 10/100 (10 warm-up samples, 100 recorded samples)

</br>

##### Armv7-A

The only problem found here was the mentioned "every 8-th" pattern. Other samples were dispersed from each other in a limited way. 
No other appearant patterns were found.


Example of a measurement for MCS kernel. Run #1 on Sabre (file sabre-mcs-old-1.log)

> {"Benchmark": "Signal to high prio thread", "Results": [{"Min": 924, "Max": 1492, "Mean": 1095.0399999999997, "Stddev": 158.32188156353556, "Variance": 24815.159999999996, "Mode": 1012.0, "Median": 1050.0, "1st quantile": 1001.5, "3rd quantile": 1131.0, "Samples": 100, "Raw results": [1214, 1142, 1146, 1126, 1124, 1052, 956, 1492, 1022, 1146, 1128, 1042, 992, 970, 1064, 1474, 1152, 1130, 1028, 1026, 1136, 1124, 1078, 1492, 944, 1052, 1012, 1146, 1128, 1046, 1002, 1472, 1044, 1036, 1130, 1138, 1050, 1012, 942, 1478, 942, 1032, 1004, 1136, 1138, 1070, 1008, 1474, 1034, 1014, 1140, 1142, 1038, 1022, 950, 1480, 938, 1072, 1024, 956, 1048, 1012, 926, 1488, 960, 1054, 1000, 956, 1074, 1000, 944, 1480, 934, 1082, 994, 942, 1012, 960, 1056, 1488, 924, 978, 944, 1068, 988, 1116, 1134, 1478, 936, 1050, 1034, 948, 1050, 1016, 1116, 1488, 1004, 1128, 1130, 1062]}]}

Example of a measurement for non-MCS kernel. Run #1 on Sabre (file sabre-nomcs-old-1.log)

> {"Benchmark": "Signal to high prio thread", "Results": [{"Min": 709, "Max": 1770, "Mean": 932.65999999999997, "Stddev": 194.93656051716573, "Variance": 37620.259999999995, "Mode": 853.0, "Median": 885.5, "1st quantile": 846.75, "3rd quantile": 947.5, "Samples": 100, "Raw results": [949, 752, 933, 717, 964, 925, 936, 1364, 983, 749, 934, 931, 732, 958, 921, 1356, 749, 953, 964, 744, 1770, 955, 881, 1360, 868, 863, 853, 853, 846, 886, 851, 1357, 863, 847, 789, 738, 979, 716, 889, 1346, 875, 873, 894, 865, 869, 826, 830, 1359, 816, 759, 963, 755, 931, 964, 709, 1363, 903, 906, 983, 763, 937, 738, 905, 1359, 925, 947, 718, 920, 942, 926, 745, 1362, 881, 875, 873, 858, 888, 877, 856, 1359, 853, 885, 841, 857, 897, 889, 916, 1346, 873, 884, 847, 794, 840, 751, 957, 1358, 710, 889, 888, 870]}]}

</br>

   - Armv8-A

The mentioned "every 8-th" pattern was not found here. On the contrary, every 8-th sample was few cycles less then neighbouring ones.</br>
"Tail pattern". 2 - 3 tailing samples were considerably higher than others for both MCS and non-MCS kernels. Further experiments showed that the increased tailing samples don't depend on number of measured samples, the "tail" always takes place.
"Head pattern". We see up to 4 first increased measurements for MCS kernel, but it happened in 2 cases: 1st and 9th runs. As for non-MCS kernels there are always 21-23 first increased measurements take place before the process stabilizes.

Example of a measurement for MCS kernel. Run #1 on tx1a (file tx1a-mcs-old-1.log)

> {"Benchmark": "Signal to high prio thread", "Results": [{"Min": 688, "Max": 772, "Mean": 699.12, "Stddev": 13.572104486851707, "Variance": 182.36000000000001, "Mode": 697.0, "Median": 697.0, "1st quantile": 694.0, "3rd quantile": 697.0, "Samples": 100, "Raw results": [772, 717, 722, 719, 694, 694, 694, 688, 694, 694, 694, 694, 694, 694, 694, 696, 694, 697, 694, 697, 694, 697, 694, 691, 694, 697, 694, 697, 694, 697, 694, 740, 697, 697, 697, 697, 697, 697, 697, 691, 697, 697, 697, 697, 697, 697, 697, 691, 697, 697, 697, 697, 697, 697, 697, 691, 697, 697, 697, 697, 697, 697, 697, 691, 697, 697, 697, 697, 697, 697, 697, 691, 697, 697, 697, 697, 697, 697, 697, 691, 697, 697, 697, 697, 697, 697, 697, 691, 697, 697, 697, 697, 697, 697, 697, 691, 697, 709, 772, 755]}]}

Example of a measurement for non-MCS kernel.  Run #1 on tx1a (file tx1a-nomcs-old-1.log)

> {"Benchmark": "Signal to high prio thread", "Results": [{"Min": 589, "Max": 702, "Mean": 605.94000000000005, "Stddev": 19.982315413708427, "Variance": 395.30000000000001, "Mode": 598.0, "Median": 598.0, "1st quantile": 595.0, "3rd quantile": 599.0, "Samples": 100, "Raw results": [630, 624, 624, 625, 595, 595, 595, 702, 641, 641, 641, 629, 625, 621, 624, 632, 642, 638, 638, 630, 621, 624, 625, 592, 598, 599, 595, 595, 595, 598, 599, 589, 598, 598, 598, 598, 599, 598, 598, 593, 595, 595, 595, 598, 599, 595, 598, 592, 598, 598, 599, 598, 598, 599, 595, 589, 595, 598, 599, 595, 598, 598, 598, 592, 599, 598, 598, 599, 595, 595, 595, 592, 599, 595, 598, 598, 598, 598, 599, 592, 598, 599, 595, 595, 595, 598, 599, 589, 598, 598, 598, 598, 599, 598, 598, 593, 595, 636, 676, 675]}]}


</br>


   - Risc-V

"Every 8-th" pattern was not found here. Rather sporadic increased measurements or in some cases series of 2-3 increased measurements.
    
   - x86_64

For traditional kernel, there are number of sporadic increased measurements. No "every 8-th" or other patterns were not found.

For MCS kernel, there are first 22-25 recorded measurements are steadily high, more than 200 cycles. No "every 8-th" or other patterns were not found.


