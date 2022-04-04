#### 1. Task

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


#### 2. Analysis of raw data

I made a check of the raw data for the available architectures.</br>

Parameters of measurements: 10/100 (10 warm-up samples, 100 recorded samples)</br>

Container `seL4/seL4-CAmkES-L4v-dockerfiles`: e9079c69284bd79d817dd6e823d56821459083b9</br>
`sel4bench-manifest`: f05bf61609a2418075e66fd8f967b75da9d624a0</br>

All the raw data are published on github in this repository in forders starting from `data-...`. 
I'm posting links to the proper folder inside the following sections.</br>

It's convenient to watch raw `.log` files at you local workstation, not github. (Ex. "less" command on Linux)</br>

</br>

##### 2.1 Armv7-A

The only problem found here was the mentioned "every 8-th" pattern. Other samples were dispersed from each other in a limited way. 
No other appearant patterns were found.


Example of a measurement for MCS kernel. Run #1 on Sabre (file sabre-mcs-old-1.log)

> {"Benchmark": "Signal to high prio thread", "Results": [{"Min": 924, "Max": 1492, "Mean": 1095.0399999999997, "Stddev": 158.32188156353556, "Variance": 24815.159999999996, "Mode": 1012.0, "Median": 1050.0, "1st quantile": 1001.5, "3rd quantile": 1131.0, "Samples": 100, "Raw results": [1214, 1142, 1146, 1126, 1124, 1052, 956, 1492, 1022, 1146, 1128, 1042, 992, 970, 1064, 1474, 1152, 1130, 1028, 1026, 1136, 1124, 1078, 1492, 944, 1052, 1012, 1146, 1128, 1046, 1002, 1472, 1044, 1036, 1130, 1138, 1050, 1012, 942, 1478, 942, 1032, 1004, 1136, 1138, 1070, 1008, 1474, 1034, 1014, 1140, 1142, 1038, 1022, 950, 1480, 938, 1072, 1024, 956, 1048, 1012, 926, 1488, 960, 1054, 1000, 956, 1074, 1000, 944, 1480, 934, 1082, 994, 942, 1012, 960, 1056, 1488, 924, 978, 944, 1068, 988, 1116, 1134, 1478, 936, 1050, 1034, 948, 1050, 1016, 1116, 1488, 1004, 1128, 1130, 1062]}]}

> [Link to Sabre MCS raw data](https://github.com/malus-brandywine/sel4bench-task-04.04.2022/tree/main/data-armv7-mcs)

</br>

Example of a measurement for non-MCS kernel. Run #1 on Sabre (file sabre-nomcs-old-1.log)

> {"Benchmark": "Signal to high prio thread", "Results": [{"Min": 709, "Max": 1770, "Mean": 932.65999999999997, "Stddev": 194.93656051716573, "Variance": 37620.259999999995, "Mode": 853.0, "Median": 885.5, "1st quantile": 846.75, "3rd quantile": 947.5, "Samples": 100, "Raw results": [949, 752, 933, 717, 964, 925, 936, 1364, 983, 749, 934, 931, 732, 958, 921, 1356, 749, 953, 964, 744, 1770, 955, 881, 1360, 868, 863, 853, 853, 846, 886, 851, 1357, 863, 847, 789, 738, 979, 716, 889, 1346, 875, 873, 894, 865, 869, 826, 830, 1359, 816, 759, 963, 755, 931, 964, 709, 1363, 903, 906, 983, 763, 937, 738, 905, 1359, 925, 947, 718, 920, 942, 926, 745, 1362, 881, 875, 873, 858, 888, 877, 856, 1359, 853, 885, 841, 857, 897, 889, 916, 1346, 873, 884, 847, 794, 840, 751, 957, 1358, 710, 889, 888, 870]}]}

> [Link to Sabre non-MCS raw data](https://github.com/malus-brandywine/sel4bench-task-04.04.2022/tree/main/data-armv7-non-mcs)

</br>

</br>

##### 2.2 Armv8-A

The mentioned "every 8-th" pattern was not found here. On the contrary, every 8-th sample was few cycles less then neighbouring ones.</br>

"Tail pattern": 2 - 3 tailing samples were considerably higher than others for both MCS and non-MCS kernels. Further experiments showed that the increased tailing samples didn't depend on number of measured samples, the "tail" always took place.</br>

"Head pattern". We see up to 4 first increased measurements for MCS kernel, but it happened in 2 casesout of 10: 1st and 9th. As for non-MCS kernels, there are always 21-23 first increased measurements take place before the process stabilizes.

Example of a measurement for MCS kernel. Run #1 on tx1a (file tx1a-mcs-old-1.log)

> {"Benchmark": "Signal to high prio thread", "Results": [{"Min": 688, "Max": 772, "Mean": 699.12, "Stddev": 13.572104486851707, "Variance": 182.36000000000001, "Mode": 697.0, "Median": 697.0, "1st quantile": 694.0, "3rd quantile": 697.0, "Samples": 100, "Raw results": [772, 717, 722, 719, 694, 694, 694, 688, 694, 694, 694, 694, 694, 694, 694, 696, 694, 697, 694, 697, 694, 697, 694, 691, 694, 697, 694, 697, 694, 697, 694, 740, 697, 697, 697, 697, 697, 697, 697, 691, 697, 697, 697, 697, 697, 697, 697, 691, 697, 697, 697, 697, 697, 697, 697, 691, 697, 697, 697, 697, 697, 697, 697, 691, 697, 697, 697, 697, 697, 697, 697, 691, 697, 697, 697, 697, 697, 697, 697, 691, 697, 697, 697, 697, 697, 697, 697, 691, 697, 697, 697, 697, 697, 697, 697, 691, 697, 709, 772, 755]}]}

> [Link to tx1a MCS raw data](https://github.com/malus-brandywine/sel4bench-task-04.04.2022/tree/main/data-armv8-mcs)

</br>

Example of a measurement for non-MCS kernel.  Run #1 on tx1a (file tx1a-nomcs-old-1.log)

> {"Benchmark": "Signal to high prio thread", "Results": [{"Min": 589, "Max": 702, "Mean": 605.94000000000005, "Stddev": 19.982315413708427, "Variance": 395.30000000000001, "Mode": 598.0, "Median": 598.0, "1st quantile": 595.0, "3rd quantile": 599.0, "Samples": 100, "Raw results": [630, 624, 624, 625, 595, 595, 595, 702, 641, 641, 641, 629, 625, 621, 624, 632, 642, 638, 638, 630, 621, 624, 625, 592, 598, 599, 595, 595, 595, 598, 599, 589, 598, 598, 598, 598, 599, 598, 598, 593, 595, 595, 595, 598, 599, 595, 598, 592, 598, 598, 599, 598, 598, 599, 595, 589, 595, 598, 599, 595, 598, 598, 598, 592, 599, 598, 598, 599, 595, 595, 595, 592, 599, 595, 598, 598, 598, 598, 599, 592, 598, 599, 595, 595, 595, 598, 599, 589, 598, 598, 598, 598, 599, 598, 598, 593, 595, 636, 676, 675]}]}

> [Link to tx1a non-MCS raw data](https://github.com/malus-brandywine/sel4bench-task-04.04.2022/tree/main/data-armv8-non-mcs)

</br>


</br>


##### 2.3 Risc-V

"Every 8-th" pattern was not found here. Rather sporadic increased measurements or in some cases series of 2-3 increased measurements.

Example of a measurement for MCS kernel.  Run #1 on hifive1 (file hifive-mcs-old-1.log)

> {"Benchmark": "Signal to high prio thread", "Results": [{"Min": 2433, "Max": 2606, "Mean": 2448.3200000000002, "Stddev": 16.453178133937246, "Variance": 268.0, "Mode": 2447.0, "Median": 2447.0, "1st quantile": 2447.0, "3rd quantile": 2447.0, "Samples": 100, "Raw results": [2462, 2446, 2446, 2446, 2446, 2446, 2446, 2449, 2606, 2444, 2436, 2433, 2446, 2447, 2447, 2443, 2447, 2447, 2447, 2447, 2447, 2447, 2447, 2443, 2447, 2447, 2447, 2447, 2447, 2447, 2447, 2443, 2447, 2447, 2447, 2447, 2447, 2447, 2447, 2443, 2447, 2447, 2447, 2447, 2447, 2447, 2447, 2443, 2447, 2447, 2447, 2447, 2447, 2447, 2447, 2478, 2447, 2447, 2447, 2447, 2447, 2447, 2447, 2443, 2447, 2447, 2447, 2447, 2447, 2447, 2447, 2443, 2447, 2447, 2447, 2447, 2447, 2447, 2447, 2443, 2447, 2447, 2447, 2447, 2447, 2447, 2447, 2443, 2447, 2447, 2447, 2447, 2447, 2447, 2447, 2443, 2447, 2447, 2447, 2447]}]}

> [Link to hifive1 MCS raw data](https://github.com/malus-brandywine/sel4bench-task-04.04.2022/tree/main/data-riscv-mcs)

</br>

Example of a measurement for non-MCS kernel.  Run #1 on hifive1 (file hifive-nomcs-old-1.log)

> {"Benchmark": "Signal to high prio thread", "Results": [{"Min": 909, "Max": 953, "Mean": 930.75999999999999, "Stddev": 14.412396459894229, "Variance": 205.63999999999999, "Mode": 944.0, "Median": 941.0, "1st quantile": 915.0, "3rd quantile": 944.0, "Samples": 100, "Raw results": [942, 919, 953, 953, 951, 909, 940, 946, 936, 913, 935, 915, 944, 915, 944, 919, 944, 915, 944, 915, 944, 915, 944, 919, 944, 915, 944, 915, 944, 915, 944, 919, 944, 915, 944, 915, 944, 915, 944, 919, 944, 915, 944, 915, 944, 915, 944, 919, 944, 915, 944, 915, 944, 915, 944, 919, 944, 915, 944, 915, 944, 915, 944, 919, 944, 915, 944, 915, 944, 915, 944, 919, 944, 915, 944, 915, 944, 915, 944, 919, 944, 915, 944, 915, 944, 915, 944, 919, 944, 915, 944, 915, 944, 915, 944, 943, 944, 915, 944, 915]}]}

> [Link to hifive1 non-MCS raw data](https://github.com/malus-brandywine/sel4bench-task-04.04.2022/tree/main/data-riscv-non-mcs)

</br>

</br>
    
##### 2.4 x86_64

For traditional kernel, there are number of sporadic increased measurements. No "every 8-th" or other patterns were not found.

For MCS kernel, there are first 22-25 recorded measurements are steadily high, more than 200 cycles. No "every 8-th" or other patterns were not found.

Example of a measurement for MCS kernel.  Run #1 on haswell4 (file haswell4-mcs-old-1.log)

> {"Benchmark": "Signal to high prio thread", "Results": [{"Min": 943, "Max": 1303, "Mean": 1010.59, "Stddev": 98.545738752432243, "Variance": 9614.1499999999996, "Mode": 952.0, "Median": 968.0, "1st quantile": 952.0, "3rd quantile": 977.0, "Samples": 100, "Raw results": [1188, 1188, 1185, 1182, 1182, 1185, 1191, 1213, 1182, 1191, 1182, 1185, 1182, 1182, 1191, 1188, 1185, 1182, 1182, 1185, 1191, 1303, 1013, 1022, 952, 946, 952, 977, 949, 946, 943, 949, 974, 955, 971, 949, 943, 974, 949, 968, 943, 943, 968, 943, 943, 968, 943, 943, 968, 943, 943, 968, 943, 943, 977, 949, 977, 949, 952, 974, 952, 974, 952, 974, 952, 952, 977, 952, 977, 952, 952, 949, 977, 949, 952, 974, 952, 974, 952, 977, 952, 952, 977, 952, 977, 952, 952, 949, 952, 974, 952, 974, 952, 949, 977, 974, 952, 952, 977, 952]}]}

> [Link to haswell4 MCS raw data](https://github.com/malus-brandywine/sel4bench-task-04.04.2022/tree/main/data-x86_64-mcs)

</br>

Example of a measurement for non-MCS kernel.  Run #1 on haswell4 (file haswell4-nomcs-old-1.log)

> {"Benchmark": "Signal to high prio thread", "Results": [{"Min": 698, "Max": 732, "Mean": 718.41999999999996, "Stddev": 11.86966254317657, "Variance": 139.47999999999999, "Mode": 726.0, "Median": 726.0, "1st quantile": 704.0, "3rd quantile": 726.0, "Samples": 100, "Raw results": [732, 707, 704, 704, 704, 704, 704, 704, 704, 701, 701, 726, 729, 729, 726, 729, 723, 726, 729, 726, 726, 726, 726, 726, 726, 726, 726, 726, 726, 726, 726, 729, 726, 729, 726, 729, 726, 729, 729, 726, 726, 723, 729, 698, 701, 698, 704, 701, 704, 701, 698, 729, 698, 726, 723, 729, 729, 726, 726, 723, 729, 723, 726, 723, 726, 729, 701, 698, 698, 701, 698, 704, 698, 704, 704, 701, 701, 698, 704, 723, 726, 723, 729, 726, 729, 726, 723, 726, 723, 729, 723, 729, 729, 726, 726, 723, 726, 723, 726, 723]}]}

> [Link to haswell4 non-MCS raw data](https://github.com/malus-brandywine/sel4bench-task-04.04.2022/tree/main/data-x86_64-non-mcs)

</br>

</br>

Next > Developments 
