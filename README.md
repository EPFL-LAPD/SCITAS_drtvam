# SCITAS_drtvam

1. Open CMD in Windows
2. Do `ssh <username>@kuma.hpc.epfl.ch` bring you to the cluster
3. Copy a `config.json` from your local computer to cluster
```
scp -r <local-path-with/ply-and-config>/ <username>@kuma.hpc.epfl.ch:/home/<username>/
```
4. You can also edit an existing file on the cluster with `nano <path-to-config>/config.json`
5. To launch an optimization job, create a file called `optimize.sh`:
```bash
#!/bin/bash
#SBATCH --job-name=drtvam
#SBATCH --output=job_output.log
#SBATCH --error=job_error.log
#SBATCH --partition=l40s
#SBATCH --ntasks=1
#SBATCH --gpus-per-task=1
#SBATCH --mem=30G
#SBATCH --time=00:30:00

LOGFILE="`pwd`/job_output_$(date '+%Y-%m-%d_%H-%M-%S').log"
ERRORFILE="`pwd`/job_error_$(date '+%Y-%m-%d_%H-%M-%S').log"

# Run the command
apptainer run --bind /scratch/wechsler --nv /home/wechsler/mitsuba-vam/container_pip.sif drtvam $1 > "$LOGFILE" 2> "$ERRORFILE"
```
5. To run an optimization, call: `sbatch optimize.sh /home/wechsler/TVAM_patterns/FVB02_sparse_2/config.json`
6. For example, inspect if your job is running. In this case my status is `ST R`, so it runs since 2mins.
```bash
[wechsler@kuma1 ~]$ squeue -u wechsler
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
            367920      l40s   drtvam wechsler  R       2:28      1 kl001
            367919      l40s   drtvam wechsler  R       3:07      1 kl001
```
7. Run `cat job_output_2025-03-17_13-26-47.log` or `cat job_error_2025-03-17_13-26-47.log` to see your output. The filename has a timestamp included, so use the right one.
8. Synchronize patterns back to the computer. For example with
```bash
scp <username>@kuma.hpc.epfl.ch:"/scratch/username/FVB02_sparse_2/patterns/*" <local-path>/<where-you-want>
```
