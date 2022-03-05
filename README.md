Scalable and Practical Locking with *Shuffling*
===============================================

## Paper
* [Scalable And Practical Locking with Shuffling (SOSP 2019)]()

### Overview
ShflLocks are a family of locking protocols that
implement use shuffling mechanism to implement several policies,
such as NUMA-awareness, or blocking. We implement three locking
protocols: NUMA-aware spinlock, NUMA-aware blocking mutex,
and NUMA-aware blocking rwlock.

ShflLock for the kernel(```klocks```) is provided under the terms of 
GNU General Public License v2, and the user library(```ulocks```) 
is released under the terms of MIT license. Please see the LICENSE 
files in each directory.

## Tested Environment
- We use Ubuntu 16.04 but with our versions of kernels for testing purposes.

### Required programs
- We use the following programs for evaluation
  - Fxmark: file system micro-benchmarking in the kernelspace
  - lock1: This benchmark stresses the kernel spinlock and is part of the will-it-scale benchmark suite
  - Exim: mail server to stress kernelspace locks
  - Metis: map-reduce library to stress readers-writer lock in the kernel
  - AFL: fuzzer to stress kernel locks
  - Dedup: Parsec benchmark that stresses lock allocation in the usersapce
  - Streamcluster: Parsec benchmark that has about 40% of `trylock` calls in the userspace
  - Leveldb: Database benchmark that stresses a single lock in the userspace
  - Extended version of RCU-table benchmark, used for nano-benchmarking locks.


### Kernelspace Benchmark
- To test locks in the kernelspace, patch the kernel in the `patches` folder
- Clone the following benchmark suite:
```bash
  $ git clone https://github.com/sslab-gatech/vbench
  $ git clone https://github.com/sslab-gatech/fxmark
```

#### Fxmark

- We use the following benchmarks for evaluating all types of locks
  - MWCM
  - MWRL
  - MWRM
  - MRDM

- We use following settings for our evaluation, which can be modified in `bin/run-fxmark.py`:
```python
    self.DIRECTIOS = ["nodirectio"]
    self.MEDIA_TYPES = ["mem"]
    self.FS_TYPES = ["tmpfs"]
```

- One issue with Fxmark is that we need to specify which benchmarks to run on line 520 in `run-fxmark.py`.
  Please, modify the following line `("mem", "*", "MWCM", "80", "directio")),` to `("mem", "tmpfs", "MWCM", "*", "nodirectio")),`
  If we use `*` for each column in the tuple, then Fxmark will comprehensively run all benchmarks on specified mediums for varying core count.
  Columns in the tuple represent the following:
  - `mem`: storage medium
  - `tmpfs`: file system
  - `DWOL`: benchmark name
  - `80`: thread count
  - `nodirectio`: This is specific for nodirectio
- Note that `NUM_TEST_CONF=0` may be encountered on not appropriately setting the thread count.
  For testing purpose, please use the online core count (excluding HT).

#### Vbench

- We use the following benchmarks from Vbench:
  - Exim
  - Metis

- Please follow the README.md in the Vbench repo on how to run the benchmark and generate graphs.

- AFL: Please follow the README.md in the `benchmarks/fuzzing` directory

- Nano-benchmark: Please refer to the `benchmarks/kernel-syncstress`.

### Userspace Benchmark

- We extend the `Litl` framework (`ulocks/src/litl/`) and use the `LD_PRELOAD` for usersapce benchmarks.

- Compile the `Litl` framework as follows:
```bash
make -C ulocks/src/litl
```

- For Leveldb, follow `Benchmark.md` in the `benchmarks/leveldb-1.20` folder.

- For Dedup and Streamcluster, follow the README.md in the `benchmarks/parsec` folder.


### Contact
- Sanidhya Kashyap (sanidhya@gatech.edu)
