# Cluster Benchmark

A common generic benchmark for clusters is Linpack, or HPL (High Performance Linpack), which is famous for its use in rankings in the TOP500 supercomputer list over the past few decades.

I wanted to see where my various Pi clusters would rank, historically, so I built this playbook which installs all the necessary tooling for HPL to run, connects all the nodes together via SSH, then runs the benchmark and outputs the result.

## Why not PTS?

Phoronix Test Suite includes [HPL Linpack](https://openbenchmarking.org/test/pts/hpl) and [HPCC](https://openbenchmarking.org/test/pts/hpcc) test suites. I may see how they compare in the future.

When I initially started down this journey, the PTS versions didn't play nicely with the Pi, especially when clustered.

## Usage

Make sure you have Ansible installed, and make sure you've at _least_ run the `networking.yml` playbook in the main directory. Then run the benchmarking playbook inside this directory:

```
ansible-playbook main.yml
```

You should be able to log directly into any of the nodes (I did my tests on node 1), and run the following commands to kick off a benchmarking run:

```
cd ~/tmp/hpl-2.3/bin/rpi
mpirun -f cluster-hosts ./xhpl
```

> The configuration here was tested on smaller 1, 4, and 6-node clusters with 6-64 GB of RAM. Some settings in the `config.yml` file that affect the generated `HPL.dat` file may need diffent tuning for different cluster layouts!

### Single Node Usage

To run locally on a single node, make the following changes:

First, create a `hosts.ini` file in this directory with the following contents:

```
[cluster]
127.0.0.1 ansible_connection=local
```

Second, _don't_ run the entire playbook. Run the playbook with the following tags so SSH inter-node configuration is not set:

```
ansible-playbook main.yml --tags "setup,benchmark"
```

> When testing, I run the playbook inside an Ubuntu docker container:
> 
> ```
> docker run -it --rm -v $PWD:/code geerlingguy/docker-ubuntu2204-ansible:latest bash
> ```

## Results

In my testing on Raspberry Pi OS Bullseye, in November 2021, I got the following results:

| Benchmark | Configuration | Result | Wattage | Gflops/W |
| --- | --- | --- | --- | --- |
| HPL (1.5 GHz default clock) | Turing Pi 2 (4x CM4) | 44.942 Gflops | 24.5W | 1.83 Gflops/W |
| HPL (2.0 GHz overclock) | Turing Pi 2 (4x CM4) | 51.327 Gflops | 33W | 1.54 Gflops/W |
| HPL (1.5 GHz default clock) | DeskPi Super6c (6x CM4) | TODO Gflops | TODOW | TODO Gflops/W |
