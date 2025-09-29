# PRE2Fuzz - Bridging Protocol Reverse Engineering and Protocol Fuzzing
PRE2Fuzz is for bridging protocol reverse engineering and protocol fuzzing. It includes a suite of representative open-source network servers for popular protocols (e.g., FTP), and tools to automate experimentation.
# Folder Structure
```
pre2fuzz
├── subjects: contains different protocol implementations
│   └── FTP
│       └── lightftp
│           └── Dockerfile: for building the Docker image specific to the target server
│           └── run.sh: main script to run the experiment inside a Docker container
│           └── other necessary files
└── scripts: contains all scripts for running experiments and analyzing results
    ├── execution
    │   └── prefuzzbench_pre.sh: main script to run reverse experiments
    │   └── transform.py: main script to run transform experiments
    │   └── prefuzzbench_fuzz.sh: main script to run fuzzing experiments
    │   ...
    └── analysis
        └── profuzzbench_plot.py: sample script for plotting the results
└── config.ini: configuration file
└── README.md: this file
```
# Quick Start - Reversing LightFTP traffic with NetPlier and Fuzzing LightFTP server with Peach
Follow the steps below to run and collect experimental results for LightFTP, which is a lightweight File Transfer Protocol (FTP) server. The similar steps should be followed to run experiments on other subjects.
## Step-0. Set up environmental variables
```
git clone https://github.com/csu-wingmate/pre2fuzz.git
cd PRE2Fuzz
sudo mkdir in
sudo mkdir pits
export PFBENCH=$(pwd)
export PATH=$PATH:$PFBENCH/scripts/execution:$PFBENCH/scripts/analysis
```

## Step-1. Build a reverse tool docker image, a fuzzer docker image and a protocol docker image
```bash
cd $PFBENCH
cd fuzzers/Peach
docker build . -t peach
```
```bash
cd $PFBENCH
cd subjects/FTP/LightFTP
docker build . -t lightftp
```
```bash
# netplier and netzob share a common docker image
docker pull netplier:out
```

## Step-2. Run reversing
Run prefuzzbench_pre.sh script to start an experiment. The script takes 3 arguments as listed below.
- ***1st argument (PROTOCOL)*** : name of the protocol implementation
- ***2th argument (PRE)***   : reverse tool name (e.g., netplier)
- ***3th argument (TIMEOUT)***  : time for fuzzing in seconds
The following commands run an instance of Peach to fuzz LightFTP for 5 minutes.

```bash
cd $PFBENCH/scripts
sudo chmod +x PRE2Fuzz_pre.sh
./PRE2Fuzz_pre.sh lightftp netplier:out 300
```
_________________
A successful script execution will produce output similar to this:
```
NETPLIER:OUT: Collecting results from container and save them to $PFBENCH/in
NETPLIER:OUT: I am done!
```

## Step-3. Run transforming
The following commands transform the reverse result to a Pit file.
Before running the script, you should edit the config.ini as your wish.
```bash
cd $PFBENCH/scripts
sudo python PRE2Fuzz_tran.py
```

## Step-4. Run fuzzing
- ***1st argument (PROTOCOL)*** : name of the protocol Implementation
- ***2rd argument (SAVETO)***   : path to a folder keeping the results
- ***3th argument (FUZZER)***   : fuzzer name (e.g., peach)
- ***4th argument (TIMEOUT)***  : time for fuzzing in seconds
The following commands run an instances of Peach to fuzz LightFTP for 5 minutes.

```bash
cd $PFBENCH
sudo mkdir results-lightftp
cd scripts
sudo chmod +x PRE2Fuzz_fuzz.sh
sudo ./PRE2Fuzz_fuzz.sh lightftp results-lightftp peach 300
```

## Step-5. Collect the results
All results are stored in tar files within the folder created in Step-2 (results-lightftp). This includes directories named similarly to peach-1-branch and peach-1-logs, where peach-1-branch contains the collected branch coverage data and peach-1-logs contains the log files from the Peach testing process, including the number of test runs and potential bug reports.

## Step-6. Analyze the results
The branch coverage data collected in Step 3 can be used for plotting. We provide a sample Python script profuzzbench_plot.py to visualize code coverage over time. Use the following command to plot the results and save them to a file.
```bash
cd $PFBENCH/results-lightftp

profuzzbench_plot.py -i <input_data> -o <output_plot_file>
```
Replace <input_data> with the path to your coverage data and <output_plot_file> with the desired filename for your plot.

## Full process excution
You can also run the prefuzzbench_common script for the full process excution.
```bash
cd $PFBENCH
sudo mkdir results-lightftp
cd scripts
sudo chmod +x prefuzzbench_common.sh
sudo ./prefuzzbench_common.sh lightftp results-lightftp netplier:out peach 300
```
