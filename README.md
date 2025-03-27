# SCITAS_drtvam

## Useful commands on Linux -> SCITAS
* `ls`: lists all files in a directory
* `cat`: prints a file content
* `nano:`: simple editor to edit a file
* `<TAB>`: autocomplete on the cluster


## Optimize a new patterns
* Copy a folder, on the cluster do:`cp -r FVR02 FVR03`
* `nano FVR03/config.json` and adapt values, then do `CTRL+X` save
* `sbatch optimize.sh FVR03/config.json`


# One Example
1. Open CMD in Windows
2. Create a folder on your local machine containing the `target.stl` and the `config.json`. Copy the whole folder from your local computer to cluster
```
scp -r <local-path-with/ply-and-config>/ <username>@kuma.hpc.epfl.ch:/home/<username>/
```
3. Do `ssh <username>@kuma.hpc.epfl.ch` to access the cluster terminal
4. To launch an optimization job, create a file (or use the existing one) called `optimize.sh`:
```bash
#!/bin/bash
#SBATCH --job-name=drtvam
#SBATCH --output=job_output.log
#SBATCH --error=job_error.log
#SBATCH --partition=l40s
#SBATCH --ntasks=1
#SBATCH --gpus-per-task=1
#SBATCH --mem=30G
#SBATCH --time=01:05:00

LOGFILE="`pwd`/logs/$(date '+%Y-%m-%d_%H-%M-%S').log"


mkdir -p logs
# Run the command
apptainer run --bind /scratch/<username> --nv /home/<username>/container.sif drtvam $1 >>  "$LOGFILE" 2>&1
```

5. To run an optimization, call: `sbatch optimize.sh /home/wechsler/TVAM_patterns/FVB02_sparse_2/config.json`
6. For example, inspect if your job is running. In this case my status is `ST R`, so it runs since 2mins.
```bash
[wechsler@kuma1 ~]$ squeue -u wechsler
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
            367920      l40s   drtvam wechsler  R       2:28      1 kl001
            367919      l40s   drtvam wechsler  R       3:07      1 kl001
```
7. Run `cat logs/2025-03-17_13-26-47.log` to see your output. The filename has a timestamp included, so use the right one.
8. Once finished, synchronize patterns back to the computer. On your local machine, execute 
```bash
scp -r <username>@kuma.hpc.epfl.ch:"/scratch/username/RR01" <local-path>/<where-you-want>
```
9. Perform analysis of the optimization (`histogram.png`, patterns).


## Container file for Dr.TVAM on Scitas
Build container with: `srun --pty -p l40s -n 1 --cpus-per-task=16 --gpus-per-task=1  --time=00:10:00 apptainer build --force container.sif container.def`
```
Bootstrap: docker
From: nvidia/cuda:12.6.0-cudnn-devel-ubuntu22.04

%post -c /bin/bash
        apt-get -y update
        apt-get -y install libpython3-dev python3-setuptools python3-pip python3-venv git
        # Set up Python virtual environment
        python3 -m venv /opt/venv
        source /opt/venv/bin/activate
        pip install --upgrade pip
        pip install numpy mitsuba scipy matplotlib
        pip install git+https://github.com/rgl-epfl/drtvam

%environment
        export VIRTUAL_ENV=/opt/venv
        export PATH="$VIRTUAL_ENV/bin:$PATH"

%startscript
        #!/bin/bash
        source /opt/venv/bin/activate
        echo "Container was created $NOW"

%runscript
        #!/bin/bash
        echo "Container was created $NOW"
        echo "Arguments received: $*"
        source /opt/venv/bin/activate
        exec $@


```
