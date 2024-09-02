# Introduction to SLURM

SLURM is like a manager for a group of powerful computers. It helps you schedule and run tasks on these computers efficiently. Think of it as a smart assistant that helps you book time on a shared resource, like a community kitchen, so everyone gets a turn without conflicts.


Imagine you want to cook a meal in a community kitchen. You need to book a time slot and specify what resources (like stove, oven, and counter space) you need. `salloc` as we will se in what follows is the command you use to book these resources for an interactive session. 

## A toy example

Suppose also that you want to utilize GPU resources to run the following code (file names `gpu_test.py`):

```bash
# gpu_test.py

import torch

def main():
    # Check if GPU is available
    if torch.cuda.is_available():
        print("GPU is available.")
        # Perform a simple tensor operation on the GPU
        x = torch.tensor([1.0, 2.0, 3.0], device='cuda')
        y = torch.tensor([4.0, 5.0, 6.0], device='cuda')
        z = x + y
        print(f"Result of tensor addition on GPU: {z}")
    else:
        print("GPU is not available.")

if __name__ == "__main__":
    main()
```

To load the conda module:

```bash 
module load anaconda3/2023.09
```

Create a virtual environment 
```bash
conda create --name myenv
```

Activate the environment 
```bash
conda activate myenv
```

Install the packages you need. For example, you may need to install torch. You can do this with `pip install` or `conda install` the name of the package.   




## Interactive execution


Suppose that you logged to [astral](https://www.wm.edu/offices/it/services/researchcomputing/hw/nodes/astral/).

<!-- ### Booking resources with `salloc` -->
You want to allocate GPU resources for your code. You can book resources with `salloc`:

```bash 

salloc -N 1 -n 1 -t 0:20:00 --gpus=1

```
Here:

- `-N`: is the number of nodes
- `-n`: is the number of tasks
- `-t`: is the time allocation 
- `--gpus`: specifies the GPUs utilized

You can also use:
- `--ntasks=1`: This means you are booking 1 CPUs.
- `--mem=4G`: This means you need 4GB of memory.
- `--time=01:00:00`: This means you want allocation for an hour.

This should produce a output similar to 
```bash
salloc: Granted job allocation 45381
salloc: Nodes as01 are ready for job
```

<!--
```bash
salloc --ntasks=2 --mem=4G --time=01:00:00
```
-->


<!-->
```bash
salloc: Granted job allocation 12345
salloc: Waiting for resource configuration
salloc: Nodes compute-node-01 are ready for job
```
-->


You then run your code as `python gpu_test.py`


<!--
```{admonition} Try this
:class: tip

Navigate to the git repo, and add a new text file `third_file.txt` to it, and push it to a new branch named `slurm-branch`
```
-->



## Submitting jobs through `sbatch`

Once you have written your script or executable file, you can submit it as a job to be executed on the SLURM cluster using the `sbatch` command. This allows you to run your code in a batch mode, without the need for an interactive session.

To submit a job using `sbatch`, you need to create a submission script that specifies the necessary resources and commands for your job. Here's an example of a submission script, you can name it `run_job.csh`, ofcourse we need to modify some info here:

<!--
```bash
#!/bin/tcsh
#SBATCH --job-name=myjob           # Job name
#SBATCH --output=<path/to/output/output.log>        # Output file
#SBATCH --error=<path/to/output/error.log>       # Error file
#SBATCH --ntasks=1                 # Number of tasks (CPUs) to allocate
#SBATCH --mem=1G                   # Memory per node
#SBATCH --time=01:00:00            # Walltime limit

# Load any necessary modules
module load python/3.8
module load anaconda3

# Run your executable or script or ANYTHING YOU WANT
echo "I am going to sleep for 10s"
sleep 10s
python -c "print (1+3+4+5)"
sleep 15s
echo "I am going run something that does not exist"
AID2E_IS_AWESOME
echo "I am going to gracefully end this job"
```
-->


```bash 
#!/bin/tcsh
#SBATCH -J gpu_test_job        # Job name
#SBATCH --output=/sciclone/scr10/cfanelli/test/output.log        # Output file
#SBATCH --error=/sciclone/scr10/cfanelli/test/error.log       # Error file
#SBATCH -N 1                   # Number of nodes
#SBATCH -n 1                   # Number of tasks/cores per node
#SBATCH --gres=gpu:1           # Request 1 GPU
#SBATCH -t 0:10:00             # Runtime in hh:mm:ss


# Load the necessary modules (if required)
module load anaconda3/2023.09

# Activate your Python virtual environment
conda activate mytestenv

# Run your Python script
python gpu_test.py > gpu_test_output.txt

# Deactivate the environment (optional)
# conda deactivate
```

Let's break down the different parts of the submission script:

- `#!/bin/bash`: This line specifies the interpreter to be used for the script (in this case, Bash).

- `#SBATCH --job-name=myjob` (or -J): This line sets the name of your job. You can choose any name you like.

- `#SBATCH --output=output.log`: This line specifies the file where the standard output of your job will be written.

- `#SBATCH --error=error.log`: This line specifies the file where the standard error of your job will be written.

- `#SBATCH --ntasks=1`: This line specifies the number of tasks (CPUs) to allocate for your job. You can adjust this number based on the requirements of your code.

- `#SBATCH --mem=1G`: This line if present specifies the amount of memory to allocate per node for your job. Again, you can adjust this based on your code's memory requirements.

- `#SBATCH --time=01:00:00`: This line sets the walltime limit for your job. It specifies the maximum amount of time your job can run.

To submit your job, save the submission script to a file (e.g., `myscript.csh`) and use the following command:

```bash
sbatch run_job.csh
```

This will submit your job to the SLURM cluster, and you will receive a job ID as output. You can use this job ID to monitor the status of your job using the `squeue` command.

That's it! You have successfully submitted a job using SLURM. Now you can sit back and let SLURM manage the execution of your code on the cluster.


To check your submitted jobs, do:

```bash
squeue -u your_username
```