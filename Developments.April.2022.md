
#### 3. Developments

The solution (by Shane Kadish) to add emply loop delay into measuring loop rectified "every 8-th" pattern for Armv7
and improved latencies for Armv8.</br>

The code of the empty loop: `for( volatile int j = 0; j < 150; j++){}`

Further experiments showed that it's the combination of stores/loads produced by this code engaded cache maintenance
operations. It has nothing to do with clock cycles elapsed between recording current sample and starting the new measurement cycle.

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

</br>

I took measurements with the "delay" switched on for other platforms. The raw results published in
`data-riscv-...` and `data-x86_64-...` folders of this reposotory. Files contain `old-delay` in their names.







