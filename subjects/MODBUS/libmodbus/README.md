Please carefully read the [main README.md](../../../README.md), which is stored in the benchmark's root folder, before following this subject-specific guideline.

# Fuzzing libmodbus server with AFLNet and AFLnwe
Please follow the steps below to run and collect experimental results for libmodbus, which is a free software library to send/receive data with a device which respects the Modbus protocol.

## Step-1. Build a docker image
The following commands create a docker image tagged libmodbus. The image should have everything available for fuzzing and code coverage calculation.

```bash
cd $PFBENCH
cd subjects/Modbus/libmodbus
docker build . -t libmodbus
```

## Step-2. Run fuzzing
The following commands run 4 instances of AFLNet and 4 instances of AFLnwe to simultaenously fuzz libmodbus in 60 minutes.

```bash
cd $PFBENCH
mkdir results-libmodbus

profuzzbench_exec_common.sh libmodbus 4 results-libmobus aflnet out-libmodbus-aflnet "-P FTP -D 10000 -q 3 -s 3 -E -K" 3600 5 &
profuzzbench_exec_common.sh libmodbus 4 results-libmodbus aflnwe out-libmodbus-aflnwe "-D 10000 -K" 3600 5
```

## Step-3. Collect the results
The following commands collect the  code coverage results produced by AFLNet and AFLnwe and save them to results.csv.

```bash
cd $PFBENCH/results-lightftp

profuzzbench_generate_csv.sh lightftp 4 aflnet results.csv 0
profuzzbench_generate_csv.sh lightftp 4 aflnwe results.csv 1
```

## Step-4. Analyze the results
The results collected in step 3 (i.e., results.csv) can be used for plotting. Use the following command to plot the coverage over time and save it to a file.

```
cd $PFBENCH/results-lightftp

profuzzbench_plot.py -i results.csv -p lightftp -r 4 -c 60 -s 1 -o cov_over_time.png
```
