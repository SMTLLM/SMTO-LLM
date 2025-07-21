# Artifact description
This is the artifact of submission #5665 (**SMTO-LLM: Solving SMTO Formulas via Large Language Models**) in ICSE 2026. 

The artifact is provided as a docker image, which contains the prototype of the proposed method and benchmarks used for evaluation. The aim is to assist the reviewers in reproducing the experimental results in our evaluation.

# Prerequisites
+ Hardware: ~2.50 GHz CPU (all experiments were performed on a server with Intel(R) Xeon(R) Platinum 8269CY CPU @ 2.50GHz, 80 Cores, 192GB), 50+GB free disk space.
+ Unix / Linux OS: We have validated the artifact on Ubuntu system
+ [Docker](https://www.docker.com/pricing/)

# Running the artifact
We assume that the following commands are run in sudo mode. 

Firstly, pull the already prebuilt docker image from [docker hub](https://hub.docker.com/r/smtllm/smto-llm). Please make sure its name is `smtllm/smto-llm`.
```sh
$ docker pull smtllm/smto-llm:v2
```

If everything is ok, a `smtllm/smto-llm` image should be found in the images listed by `docker images` command. Then, you can create a container of such image and start a Bash session using the following command. An interactive `bash` shell on the container is also executed at the moment.
```sh
$ docker run -it smtllm/smto-llm:v2 bash
```

If all goes well, the container should be running successfully. Otherwise, you can seek help from [Docker Doc](https://docs.docker.com/) if needed. 

Now, navigate into the directory containing our experimental environment and list the contents. 
```sh
$ cd /home/aaa/LLM-SMT/ && ls
LLM-KQuery  expriment  klee-uclibc  z3-4.14.1
```
Below are the steps for reproducing all experiments in sequence

# Reproducing experimental results

## SMTLIB2 benchmarks

All SMTLIB2 format benchmarks related to SMTO issues mentioned in the paper are located in the `/home/aaa/LLM-SMT/LLM-KQuery/benchmarks` directory

```sh
$ cd /home/aaa/LLM-SMT/LLM-KQuery/benchmarks && ls
Bouvier              delphi-benchmarks        np2-mod-1-unsat          strcmp-models-64-unsat
bvisalpha-16         delphi-benchmarks-unsat  realtime-xi              strcmp-models-unsat
bvisalpha-16-unsat   iabs                     realtime-xi-unsat        sygus-examples
....                 ....                     ....                     ....
```

The running script for this experiment is located in the `/home/aaa/LLM-SMT/LLM-KQuery/scripts` directory.

```sh
$ cd /home/aaa/LLM-SMT/LLM-KQuery/scripts/ && ls
benchmark_name_mapping.csv    fullbench.list    run_llm_bench.py    run_llm.py
....                          ....              ....                ....
```

We have listed the 4 most important files in `scripts`:

| Filename | Description |
| --- | --- |
| `benchmark_name_mapping.csv` | Describe the absolute paths of 210 SMTLIB2 files. |
| `fullbench.list` | Based on the sequence numbers contained in this file, determine which SMTLIB2 formulas to run in the experimental script. |
| `run_llm_bench.py` | Script for batch running experiments. Serial running multiple experimental configurations(`LLM`,`SMTO-LLM(no opt)`,`SMTO-LLM(o4-mini)`,`SMTO-LLM(o3-mini)`, `SMTO-LLM(gpt-4.1-mini)`) |
| `run_llm.py` | The most essential script for calling LLM and parsing the results, which is used to call LLM in all experiments |

For the convenience of demonstrating the effectiveness of the tool, we provide a free LLM API that limits the number of tokens. If the limit is exhausted, please modify the `run_llm.py` file and fill in your LLM key to call LLM.

```python
# please fill in your LLM API
LLM_KEY = ["xxxxxxxxxxx", "https://xxxxxxxxxx"]
llm_dict = {
"gpt-4.1-mini":     LLM_KEY,
"o3-mini"     :     LLM_KEY,
"o4-mini"     :     LLM_KEY,
}
```

**Reproduce experimental results:** When everything is ready, we can directly call the following command to run the SMTLIB2 experiment:

```sh
$ cd /home/aaa/LLM-SMT/LLM-KQuery/scripts/ 
$ LLMSMT_BIN=/home/aaa/LLM-SMT/LLM-KQuery/build/bin/SMTLIB2KQuery ./run_llm_bench.py --bench-list fullbench.list -t 60
benchmark_name_mapping.csv    fullbench.list    run_llm_bench.py    run_llm.py
......
Reading benchmark list: fullbench.list
Creating static lib for: lisalpha
Evaluating file: /home/aaa/LLM-SMT/LLM-KQuery/benchmarks-run/bvisalpha-16/test000013.smt2
Iteration #1:, status: sat, total time: 7.7550
Median Runtime: 7.7550
......
......
......
```

After all 5 configurations are completed (**about 200 minutes**), it will generate 5 corresponding CSV files that record the solution of all formulas under the corresponding configurations.

```sh
$ cd /home/aaa/LLM-SMT/LLM-KQuery/scripts/ && ls
....                 ....                    ....
llm_only.csv         llm_safe.csv            llm_opt_o4mini.csv 
llm_opt_o3mini.csv   llm_opt_gpt41mini.csv   ....
```

We use the script `compute_bench.py` to calculate these 5 CSV files, export the complete result table Table 1 (**total.csv**) from the paper, and draw Figure 7(**performance.pdf**).

```sh
$ python3 compute_bench.py && ls
total.csv            performance.pdf         ....
....                 ....                    ....
```



## Symbolic Execution on GSL

The APIs involved in the GSL experiment are located in the `/home/aaa/LLM-SMT/expriment/llmbench-gsl` directory, and each c file represents a tested API for GSL.

```sh
$ cd /home/aaa/LLM-SMT/expriment/llmbench-gsl && ls
gsl_cdf_beta_Pinv.c               gsl_sf_bessel_Y1_e.c              gsl_sf_erf_e.c                     gsl_sf_hyperg_2F1_conj_e.c
gsl_cdf_beta_Q.c                  gsl_sf_bessel_Yn_e.c              gsl_sf_eta_int_e.c                 gsl_sf_hyperg_2F1_renorm_e.c
....
```

In view of the long running time of our evaluation, the scripts try to perform all experiments in parallel. Specifically, the scripts take a user-defined value *n* and exploit at most *n* processes of working machine at run time. The value should be carefully considered to avoid resource exhaustion. In our evaluation, *n* is 32, since all experiments were performed on a 56-core server with 256GB memory.

**Reproduce experimental results:**

```bash
# python3 run_expr.py <parallel num> <execute time>
$ nohup python3 run_gsl.py 32 1800 &  
# wait......
```

The analysis will take approximately **130 core-hours**. If you run one CPU core for one hour, that is one core-hour. 

After the analysis is completed, the script will output the coverage results under different configurations to `gsl_fin.csv`, as shown below.
```bash
$ cat gsl_fin.csv
Task name, ,bfs_z3_1800_0,, ,bfs_llm_1800_0,
-, line, branch, time, line, branch, time
gsl_sf_zeta_e,128,1800,63,130,33,1803
......
......
```
