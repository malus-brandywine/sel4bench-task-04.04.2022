
#### 3. Developments



##### 3.1 Loop delay to address "every 8-th" pattern

</br>

The solution (by Shane Kadish) to add emply loop delay into measuring loop rectified "every 8-th" pattern problem for Armv7
and improved latencies for Armv8.</br>

The code of the empty loop: `for( volatile int j = 0; j < 150; j++){}`

Further experiments showed that it's the combination of stores/loads issued by this code engaded cache 
operations. It has nothing to do with clock cycles elapsed between recording the current sample and starting the new measurement cycle.

The code with the "delay" (sel4bench/apps/signal/src/main.c):


```
void low_prio_signal_fn(int argc, char **argv)
{
    assert(argc == N_LO_SIGNAL_ARGS);
    seL4_CPtr ntfn = (seL4_CPtr) atol(argv[0]);
    volatile ccnt_t *end = (volatile ccnt_t *) atol(argv[1]);
    ccnt_t *results = (ccnt_t *) atol(argv[2]);
    seL4_CPtr done_ep = (seL4_CPtr) atol(argv[3]);

    for (int i = 0; i < N_RUNS; i++) {
        ccnt_t start;

        SEL4BENCH_READ_CCNT(start);
        DO_REAL_SIGNAL(ntfn);
        results[i] = (*end - start);
        
        /* the empty loop */
        for (volatile int j = 0; j < 150; j++){}

    }

    /* signal completion */
    seL4_Send(done_ep, seL4_MessageInfo_new(0, 0, 0, 0));
    /* block */
    seL4_Wait(ntfn, NULL);
}
```
</br>

The raw results published in `data-[arch]-...` folders of this reposotory. You need files that contain `old-delay` in their names.
(I took measurements with the empty loop switched on for other platforms as well.)

</br>


##### 3.2 Early Processing to address "every 8-th" pattern

</br>

Early processing methodology described in the "Task" section was implemented and showed results the same as "Loop delay" for Armv7-A
and somewhat worse than "Loop delay" for Armv8-A.

Important factor when choosing which methodology to improve: the more code and the more branches is contained inside the measuring loop
the more instruction cache events affect measured latencies.

The raw results published in `data-[arch]-...` folders of this reposotory. You need files that contain `new` in their names.
(I took measurements with Early Processing methodology for other platforms as well.)

</br>

Sources of the implementation is in 
[`signal.task` branch of malus-brandywine's fork](https://github.com/malus-brandywine/sel4bench/tree/signal.task)

</br>

##### 3.3 "Untailing" to address "Tail pattern" for Armv8-A

</br>

I added a parameter "number of tail samples to skip" and measured latencies. For MCS kernel we see decent values;
for non-MCS kernel the latencies didn't change, because "head pattern" was not resolved.


The raw results published in `data-armv8-...` folders, in files containing `untailed` in their names.


##### 3.4 Comparisons Tables

</br>

Parameters min, max, mean, and standard deviation for 10 runs are collected into the tables.


Comparison tables</br>
`"Late Processing" ("old" methodology)` VS `"Late Processing" with the delay` VS `"Early Processing" (new methodology)`</br>
are in the following pdf files:

   - [Tables for Arm](https://github.com/malus-brandywine/sel4bench-task-04.04.2022/blob/main/data-tables/MethodologiesComparison-Arm-03.11.2022-tables.pdf)
   - [Tables for Risc-V](https://github.com/malus-brandywine/sel4bench-task-04.04.2022/blob/main/data-tables/MethodologiesComparison-RISCV-03.15.2022-tables.pdf)
   - [Tables for x86_64](https://github.com/malus-brandywine/sel4bench-task-04.04.2022/blob/main/data-tables/MethodologiesComparison-x86_64-03.16.2022-tables.pdf)

Comparison tables</br>
`"Late Processing" ("old" methodology)` VS `"Late Processing" with the delay` VS `"Late Processing", untailed`</br>
is in this pdf file:

   - [Table for Armv8](https://github.com/malus-brandywine/sel4bench-task-04.04.2022/blob/main/data-tables/MethodologiesComparison-II-Armv8-04.04.2022-tables.pdf)




