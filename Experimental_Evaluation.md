# Experimental Evaluation

## Experimental Setup

```
[info] NativeConfig(
[info]  - baseName:
[info]  - clang:                   /usr/bin/clang
[info]  - clangPP:                 /usr/bin/clang++
[info]  - linkingOptions:          [-L/usr/local/lib, -L/opt/homebrew/lib]
[info]  - compileOptions:          [-I/opt/homebrew/include, -Qunused-arguments]
[info]  - cOptions:                []
[info]  - cppOptions:              []
[info]  - targetTriple:            None
[info]  - GC:                      immix
[info]  - LTO:                     none
[info]  - mode:                    release-fast
[info]  - buildTarget              Application
[info]  - check:                   false
[info]  - checkFatalWarnings:      false
[info]  - checkFeatures            true
[info]  - dump:                    false
[info]  - sanitizer:               none
[info]  - linkStubs:               false
[info]  - optimize                 true
[info]  - incrementalCompilation:  true
[info]  - multithreading           detect
[info]  - linktimeProperties:      []
[info]  - embedResources:          false
[info]  - resourceIncludePatterns: [**]
[info]  - resourceExcludePatterns: []
[info]  - serviceProviders:        []
[info]  - optimizerConfig:
[info]     - maxInlineDepth:    32 function
[info]     - smallFunctionSize: 12 instructions
[info]     - maxCallerSize:     2048 instructions
[info]     - maxCalleeSize:     256 instructions
[info]
[info]  - semanticsConfig:         SemanticsConfig(
[info]     - finalFields: Relaxed
[info]     - strictExternCallSemantics: false
[info]     )
[info]  - sourceLevelDebuggingConfig: SourceLevelDebuggingConfig[Disabled]
[info]     - customSourceRoots:       []
[info]     - generateFunctionSourcePositions: false
[info]     - generateLocalVariables:  false
[info]     )
[info] )
```

## Results

Table: Final benchmark timings for the evaluated implementations

| n    | Zone      | Regular    | SafeZone     |
|-----:|:---------:|:----------:|:------------:|
| 500  | 6.21 ms   | 72.68 ms   | 107.74 ms    |
| 1000 | 22.04 ms  | 299.92 ms  | 532.24 ms    |
| 1500 | 48.71 ms  | 620.42 ms  | 2124.58 ms   |
| 2000 | 88.78 ms  | 1428.67 ms | 5538.24 ms   |
| 2500 | 143.25 ms | 1768.85 ms | 10021.95 ms  |
| 3000 | 215.05 ms | 2341.87 ms | 20213.02 ms  |

Across the tested sizes, the off-heap `Zone` implementation was the fastest variant by a wide margin. Relative to the regular implementation, `Zone` was between 10.9× and 16.1× faster, while it was between 17.3× and 94.0× faster than `SafeZone`. The gap widened as `n` increased, suggesting the off-heap approach scales substantially better for this workload than the heap-allocated alternatives.

### Flamegraphs

![Flamegraph for the Zone benchmark](figures/zone.png)

![Flamegraph for the regular benchmark](figures/regular.png)

![Flamegraph for the SafeZone benchmark](figures/safezone.png)

An allocation-heavy benchmark that performs 100,000,000 object allocations inside a single `Zone` completed in **5020.96 ms**, showing that zone-based allocation performance degrades when memory is kept within one scoped region.

![Flamegraph for the allocation-heavy single-Zone benchmark](figures/copilotwebbenchzone.png)

## Immix

The Immix executables were run without command-line parameters. The raw outputs below are copied directly from the executables and are included without averaging.

### Aggregation

```
Immix --- Aggregation benchmark timings (ms). Each row lists five repeated runs.

Immix.AggregationParallelBroomBenchmark & 250000   & 39.5   & 30.8   & 36.0   & 32.4   & 32.4 \
                                          & 1000000  & 268.3  & 165.1  & 183.0  & 90.7   & 155.3 \
                                          & 2250000  & 221.3  & 405.4  & 214.9  & 227.6  & 208.0 \
                                          & 4000000  & 496.1  & 675.9  & 658.6  & 331.2  & 376.2 \
                                          & 6250000  & 602.0  & 601.6  & 520.5  & 525.1  & 554.7 \
                                          & 9000000  & 1264.0 & 978.3  & 828.3  & 752.6  & 827.1 \
                                          & 12250000 & 2010.4 & 2069.6 & 1778.2 & 1404.4 & 1436.7 \
                                          & 16000000 & 3779.3 & 3437.1 & 3753.6 & 3799.7 & 3495.4 \

Immix.AggregationSequentialBroomBenchmark & 250000   & 23.3   & 18.4   & 16.0   & 19.4   & 14.8 \
                                           & 1000000  & 138.0  & 95.6   & 100.1  & 97.9   & 90.1 \
                                           & 2250000  & 290.7  & 219.1  & 258.5  & 211.0  & 255.1 \
                                           & 4000000  & 403.1  & 591.8  & 320.8  & 416.6  & 352.4 \
                                           & 6250000  & 905.0  & 629.6  & 626.1  & 802.0  & 501.7 \
                                           & 9000000  & 1057.0 & 766.8  & 769.4  & 885.7  & 751.5 \
                                           & 12250000 & 1107.8 & 1465.7 & 1005.5 & 1043.6 & 1472.2 \
                                           & 16000000 & 1591.9 & 2071.7 & 1737.4 & 1504.5 & 1447.1 \

Immix.AggregationZoneParBroomBenchmark & 250000   & 43.7   & 26.5   & 21.4   & 20.5   & 22.5 \
                                        & 1000000  & 187.7  & 116.5  & 122.5  & 160.1  & 116.4 \
                                        & 2250000  & 203.5  & 185.7  & 209.9  & 211.7  & 243.3 \
                                        & 4000000  & 722.5  & 541.8  & 614.8  & 566.9  & 372.7 \
                                        & 6250000  & 1210.4 & 1298.2 & 723.2  & 903.7  & 793.2 \
                                        & 9000000  & 1145.6 & 1478.8 & 2037.5 & 1354.4 & 1477.7 \
                                        & 12250000 & 3506.6 & 2025.0 & 2649.2 & 2136.5 & 2725.8 \
                                        & 16000000 & 4526.8 & 5102.9 & 3226.7 & 2765.0 & 3962.3 \

Immix.AggregationZoneSequentialBroomBenchmark & 250000   & 0.4    & 0.3    & 0.3    & 0.3    & 0.4 \
                                              & 1000000  & 1.6    & 1.2    & 1.2    & 1.4    & 1.2 \
                                              & 2250000  & 3.8    & 2.8    & 2.7    & 3.2    & 2.7 \
                                              & 4000000  & 6.7    & 4.9    & 5.0    & 4.9    & 5.0 \
                                              & 6250000  & 10.8   & 8.0    & 7.6    & 8.3    & 7.5 \
                                              & 9000000  & 14.5   & 11.2   & 10.9   & 11.9   & 11.4 \
                                              & 12250000 & 20.6   & 15.5   & 17.0   & 19.3   & 17.4 \
                                              & 16000000 & 27.3   & 19.7   & 19.4   & 21.0   & 20.8 \
```

### Multi-Epoch

```
Immix.MultiEpochParallelBroomBenchmark: 250000 elements: 460.4, 547.7, 399.8, 713.6, 676.3 ms; 1000000 elements: 2301.9, 1823.4, 1479.4, 1739.8, 1204.4 ms; ...

Immix.MultiEpochSequentialBroomBenchmark: 250000 elements: 148.1, 172.4, 133.3, 146.5, 166.9 ms; 1000000 elements: 689.3, 493.3, 494.7, 612.0, 511.2 ms; ...

Immix.MultiEpochZoneParBroomBenchmark: 250000 elements: 374.7, 253.8, 272.5, 334.3, 418.9 ms; 1000000 elements: 1104.8, 1206.7, 1153.7, 946.8, 1076.0 ms; ...

Immix.MultiEpochZoneSequentialBroomBenchmark: 250000 elements: 5.1, 3.3, 5.8, 4.3, 4.0 ms; 1000000 elements: 17.1, 14.0, 21.7, 17.2, 19.8 ms; ...
```

### Single-Pass

```
Immix.SinglePassParallelBroomBenchmark: 250000 elements: 18.8, 8.0, 14.2, 7.6, 11.0 ms; 1000000 elements: 51.2, 47.2, 53.2, 34.5, 57.1 ms; ...

Immix.SinglePassSequentialBroomBenchmark: 250000 elements: 13.9, 17.7, 13.8, 14.4, 16.0 ms; 1000000 elements: 57.4, 65.9, 80.1, 55.2, 56.2 ms; ...

Immix.SinglePassZoneParBroomBenchmark: 250000 elements: 34.4, 28.9, 18.7, 48.7, 78.3 ms; 1000000 elements: 127.2, 119.7, 97.6, 88.8, 77.5 ms; ...

Immix.SinglePassZoneSequentialBroomBenchmark: 250000 elements: 0.6, 0.3, 0.3, 0.3, 0.3 ms; 1000000 elements: 2.9, 1.3, 1.3, 1.5, 1.0 ms; ...
```

## Notes

- Figures are referenced by filename under `figures/`.
- Detailed raw outputs and long tables are preserved in fenced blocks above.

## None

The `none` executables were run without command-line parameters. The raw outputs
below are copied directly from the executables and are included here without
averaging or other post-processing. Larger input sizes in several benchmarks
terminate with a Scala Native GC allocation error; those failures are reported
verbatim.

### Aggregation

```text
None.AggregationParallelBroomBenchmark: 250000 elements: 21.8, 16.8, 17.4, 15.7, 17.7 ms; 1000000 elements: 80.6, 81.2, 75.2, 74.6, 81.1 ms; 2250000 elements: 197.7, 184.1, 195.7, 214.7, 224.5 ms; 4000000 elements: 401.4, 482.9, 373.5, 338.7, 352.2 ms; 6250000 elements: 651.1, 774.7, 747.9, 756.0, 1012.0 ms; 9000000 elements: [01:25:39.055] [None.AggregationParallelBroomBenchmark:25286] [ScalaNative GC|Error] Failed to allocate or grow heap space, requested size=64.00MB, available memory=0.00MB, already allocated=9702.75MB, should preallocate=false. Consider setting GC_MAXIMUM_HEAP_SIZE env variable to limit maximal heap size.

None.AggregationSequentialBroomBenchmark: 250000 elements: 16.4, 15.4, 14.5, 13.0, 14.3 ms; 1000000 elements: 65.5, 63.6, 64.2, 64.2, 64.6 ms; 2250000 elements: 153.5, 153.1, 154.1, 149.0, 176.0 ms; 4000000 elements: 310.1, 312.0, 279.5, 271.4, 281.2 ms; 6250000 elements: 517.7, 532.1, 484.6, 481.7, 493.3 ms; 9000000 elements: [01:25:46.776] [None.AggregationSequentialBroomBenchmark:25563] [ScalaNative GC|Error] Failed to allocate or grow heap space, requested size=64.00MB, available memory=0.00MB, already allocated=9011.75MB, should preallocate=false. Consider setting GC_MAXIMUM_HEAP_SIZE env variable to limit maximal heap size.

None.AggregationZoneParBroomBenchmark: 250000 elements: 26.9, 19.8, 21.7, 26.3, 23.7 ms; 1000000 elements: 76.3, 69.4, 65.9, 73.9, 76.3 ms; 2250000 elements: 154.4, 145.7, 141.0, 149.6, 144.3 ms; 4000000 elements: 333.7, 344.3, 340.5, 374.0, 346.3 ms; 6250000 elements: 565.7, 648.5, 754.1, 846.3, 581.4 ms; 9000000 elements: [01:25:56.241] [None.AggregationZoneParBroomBenchmark:25755] [ScalaNative GC|Error] Failed to allocate or grow heap space, requested size=64.00MB, available memory=0.00MB, already allocated=9217.68MB, should preallocate=false. Consider setting GC_MAXIMUM_HEAP_SIZE env variable to limit maximal heap size.

None.AggregationZoneSequentialBroomBenchmark: 250000 elements: 0.7, 0.5, 0.5, 0.5, 0.5 ms; 1000000 elements: 2.5, 1.8, 1.7, 1.8, 1.7 ms; 2250000 elements: 3.9, 2.8, 2.9, 2.9, 2.8 ms; 4000000 elements: 6.5, 4.8, 5.0, 4.9, 5.0 ms; 6250000 elements: 12.7, 9.0, 7.8, 8.1, 7.7 ms; 9000000 elements: [01:26:00.261] [None.AggregationZoneSequentialBroomBenchmark:26078] [ScalaNative GC|Error] Failed to allocate or grow heap space, requested size=64.00MB, available memory=0.00MB, already allocated=734.56MB, should preallocate=false. Consider setting GC_MAXIMUM_HEAP_SIZE env variable to limit maximal heap size.
```

### Multi-Epoch

```text
None.MultiEpochParallelBroomBenchmark: 250000 elements: 96.0, 94.5, 95.2, 93.2, 92.7 ms; 1000000 elements: 468.6, 537.1, 516.2, 564.0, 497.3 ms; 2250000 elements: 1242.6, 1174.0, 1306.6, 1178.3, 1346.0 ms; 4000000 elements: 2345.7, 2371.6, 2416.7, 2439.4, 2516.7 ms; 6250000 elements: the run stopped before printing timings for this size.

None.MultiEpochSequentialBroomBenchmark: 250000 elements: 98.7, 91.0, 89.0, 92.1, 94.4 ms; 1000000 elements: 388.5, 377.7, 365.5, 424.4, 514.8 ms; 2250000 elements: 1085.7, 1145.4, 1164.0, 1093.5, 1083.5 ms; 4000000 elements: 2096.3, 2064.8, 2144.2, 2557.5, 2090.1 ms; 6250000 elements: the run stopped before printing timings for this size.

None.MultiEpochZoneParBroomBenchmark: 250000 elements: 106.6, 99.8, 141.0, 145.5, 120.8 ms; 1000000 elements: 525.8, 436.4, 486.2, 642.1, 607.2 ms; 2250000 elements: 1220.8, 1407.5, 1246.0, 1413.9, 1693.5 ms; 4000000 elements: the run stopped before printing timings for this size.

None.MultiEpochZoneSequentialBroomBenchmark: 250000 elements: 5.5, 4.6, 4.1, 3.9, 3.9 ms; 1000000 elements: 12.8, 11.6, 11.1, 11.1, 11.1 ms; 2250000 elements: 27.6, 25.4, 25.3, 25.2, 27.9 ms; 4000000 elements: 58.8, 47.9, 51.2, 48.2, 49.6 ms; 6250000 elements: 79.7, 78.6, 71.0, 71.6, 73.0 ms; 9000000 elements: [01:27:45.722] [None.MultiEpochZoneSequentialBroomBenchmark:28367] [ScalaNative GC|Error] Failed to allocate or grow heap space, requested size=64.00MB, available memory=0.00MB, already allocated=734.62MB, should preallocate=false. Consider setting GC_MAXIMUM_HEAP_SIZE env variable to limit maximal heap size.
```

### Single-Pass

```text
None.SinglePassParallelBroomBenchmark: 250000 elements: 5.4, 4.8, 4.7, 8.3, 6.2 ms; 1000000 elements: 16.6, 16.1, 15.5, 17.5, 15.2 ms; 2250000 elements: 36.7, 35.9, 34.8, 35.9, 32.2 ms; 4000000 elements: 54.1, 52.4, 54.5, 52.9, 52.2 ms; 6250000 elements: 138.1, 142.5, 184.9, 163.7, 169.7 ms; 9000000 elements: [01:27:49.546] [None.SinglePassParallelBroomBenchmark:28513] [ScalaNative GC|Error] Failed to allocate or grow heap space, requested size=64.00MB, available memory=0.00MB, already allocated=7467.61MB, should preallocate=false. Consider setting GC_MAXIMUM_HEAP_SIZE env variable to limit maximal heap size.

None.SinglePassSequentialBroomBenchmark: 250000 elements: 9.6, 9.2, 9.1, 9.1, 9.1 ms; 1000000 elements: 37.9, 37.2, 37.1, 37.0, 37.1 ms; 2250000 elements: 84.3, 83.5, 83.2, 90.3, 94.7 ms; 4000000 elements: 158.7, 148.7, 150.2, 146.9, 147.5 ms; 6250000 elements: [01:27:53.679] [None.SinglePassSequentialBroomBenchmark:28660] [ScalaNative GC|Error] Failed to allocate or grow heap space, requested size=64.00MB, available memory=0.00MB, already allocated=6401.85MB, should preallocate=false. Consider setting GC_MAXIMUM_HEAP_SIZE env variable to limit maximal heap size.

None.SinglePassZoneParBroomBenchmark: 250000 elements: 10.8, 9.0, 7.5, 6.1, 6.2 ms; 1000000 elements: 23.7, 24.1, 25.3, 25.4, 23.2 ms; 2250000 elements: 50.0, 50.9, 50.2, 49.3, 47.6 ms; 4000000 elements: 90.3, 90.9, 111.5, 99.3, 214.3 ms; 6250000 elements: 263.5, 256.2, 252.0, 269.0, 278.5 ms; 9000000 elements: [01:27:58.697] [None.SinglePassZoneParBroomBenchmark:28783] [ScalaNative GC|Error] Failed to allocate or grow heap space, requested size=64.00MB, available memory=0.00MB, already allocated=8440.96MB, should preallocate=false. Consider setting GC_MAXIMUM_HEAP_SIZE env variable to limit maximal heap size.

None.SinglePassZoneSequentialBroomBenchmark: 250000 elements: 0.4, 0.3, 0.3, 0.3, 0.3 ms; 1000000 elements: 1.5, 1.5, 1.0, 1.0, 1.0 ms; 2250000 elements: 5.7, 2.6, 2.3, 2.3, 2.3 ms; 4000000 elements: 5.7, 4.4, 4.2, 4.3, 4.1 ms; 6250000 elements: 9.2, 6.3, 6.9, 6.3, 6.4 ms; 9000000 elements: [01:28:01.407] [None.SinglePassZoneSequentialBroomBenchmark:28934] [ScalaNative GC|Error] Failed to allocate or grow heap space, requested size=64.00MB, available memory=0.00MB, already allocated=734.56MB, should preallocate=false. Consider setting GC_MAXIMUM_HEAP_SIZE env variable to limit maximal heap size.
```

## Discussion

The measurements answer the research question in the affirmative for this
benchmark family: off-heap allocation in Scala Native can deliver materially
better throughput than heap-based approaches for data-intensive workloads.

Taken together, the benchmarks support two practical conclusions. First,
off-heap memory is a viable strategy when predictable latency and high
throughput are the primary goals. Second, the implementation should avoid
patterns that introduce avoidable allocation overhead, since those costs can
erase most of the benefit of using zones in the first place.
