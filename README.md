# `sparkhpc`: deploy Spark standalone on High-Performance Computing resources

## Usage

### Command line

```
$ sparkcluster
Usage: sparkcluster [OPTIONS] COMMAND [ARGS]...

Options:
  --scheduler [lsf]  Which scheduler to use
  --help             Show this message and exit.

Commands:
  info    Get info about currently running clusters
  kill    Kill a currently running cluster
  submit  Submit the spark cluster as an LSF job

$ sparkcluster submit 10

$ sparkcluster info
----- Cluster 0 -----
Job 31454252 yet started

$ sparkcluster info
----- Cluster 0 -----
Number of cores: 4
master URL: spark://10.11.12.13:7077
Spark UI: http://10.11.12.13:8080

$ sparkcluster kill 0
Job <31463649> is being terminated
```

### Python code

```python
import sparkhpc
import pyspark

sj = sparkhpc.LSFSparkJob(ncores=10)

sj.wait_to_start()

master = sj.master_url(sj.jobid)

sc = pyspark.SparkContext(master=master)

sc.parallelize(...)
```

## Installation

```
$ python setup.py install
```

This will install the python package to your default package directory as well as the `sparkcluster` command-line script. 


## Spark jupyter notebook

Running Spark applications, especially with python, is really nice from the comforts of a [Jupyter notebook](http://jupyter.org/).
The `start_notebook.py` script will setup and launch a secure, password-protected notebook for you. The first time you run the notebook
script, you should run it with the `--setup` flag. It will first ask for a password for the notebook and generate a self-signed ssh
certificate - this is done to prevent other users of your cluster to stumble into your notebook by chance. To get some usage information
just type

```
$ ./start_notebook.py
usage: start_notebook.py [-h] [--setup] [--launch] [--port PORT] [--spark]
                         [--spark_options SPARK_OPTIONS]
                         [--spark_conf SPARK_CONF]

Setup and launch a python notebook set up to serve a Spark session

optional arguments:
  -h, --help            show this help message and exit
  --setup               setup the notebook (does not launch)
  --launch              launch the notebook
  --port PORT           Port number for the notebook server
  --spark               launch the notebook with a spark backend
  --spark_options SPARK_OPTIONS
                        options to pass to spark
  --spark_conf SPARK_CONF
                        spark_configuration_directory
```

You have two options for launching the notebook:  

1. if you want to launch a spark context by hand and do all the configuration inside
your application, just launch the notebook using the `--launch` flag (this does nothing spark-specific, just starts the server). Because
this script is meant to be used while running an interactive session on an HPC cluster, it will also tell you the IP of the machine
where it is running to simplify any tunneling that needs to be done. 

2. if you'd like the notebook to also initialize a spark context and connect to e.g. YARN or some other master, you should specify 
the `--spark` option on the command line instead. This then also gives you the option of specifying other spark options via
`--spark_options` or a configuration directory via `--spark_conf`. For example, to launch a notebook connected to a standalone
spark cluster created with the `start_spark_lsf.py` script, you would request an interactive job and do something like:

```
compute node $ ./start_spark_lsf.py 24 50G
compute node $ ./start_notebook.py --spark --spark_conf "--master spark://<master-host>:7077
```

where you would replace `<master-host>` with the actual master hostname, which will be printed on the screen by the `start_spark_lsf.py` script. 

The provided `notebook_job.lsf` is a template job submission script for LSF. It can run the notebook server as a batch job, you 
just need to note the host so you can connect to it. This is easily done with `bpeek` to inspect the output of the job:

```
head node $ bsub < notebook_job.lsf
Generic job.
Job <1351868> is submitted to queue <pub.1h>.

head node $ bpeek
The Spark driver will be available on host IP: 1.2.3.4
Picked up _JAVA_OPTIONS: -XX:ParallelGCThreads=1
[I 15:11:13.250 NotebookApp] Serving notebooks from local directory: /cluster/home03/sdid/roskarr
[I 15:11:13.250 NotebookApp] 0 active kernels
[I 15:11:13.250 NotebookApp] The IPython Notebook is running at: https://[all ip addresses on your system]:8889/
[I 15:11:13.250 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
```

So in this case, you could set up a port forward to host `1.2.3.4` and instruct your browser to connect to `https://1.2.3.4:8889`.
