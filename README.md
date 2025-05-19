# SCITAS_drtvam

## Useful commands on Linux -> SCITAS
* `ls`: lists all files in a directory
* `cat`: prints a file content
* `nano:`: simple editor to edit a file
* `<TAB>`: autocomplete on the cluster


## Optimize a new patterns
* Copy a folder, on the cluster do:`cp -r FVR02 FVR03`
* `nano FVR03/config.json` and adapt values (change `output:`!), then do `CTRL+X` save
* `./optimize_slurm.sh FVR03/config.json`


# One Example
1. Open CMD in Windows
2. Create a folder on your local machine containing the `target.stl` and the `config.json`. Copy the whole folder from your local computer to cluster
```
scp -r <local-path-with/ply-and-config>/ <username>@kuma.hpc.epfl.ch:/home/<username>/
```
3. Do `ssh <username>@kuma.hpc.epfl.ch` to access the cluster terminal
4. To launch an optimization job, create a file (or use the existing one) called `optimize_slurm.sh`:
```bash
#!/bin/bash

# Default time limit
time_limit="00:45:00"

# Parse command line arguments
while [[ $# -gt 0 ]]; do
    case $1 in
        --time_limit=*)
            time_limit="${1#*=}"
            shift
            ;;
        *)
            config_file="$1"
            shift
            ;;
    esac
done

# Check if a config file is provided
if [ -z "$config_file" ]; then
    echo "Error: No config file provided. Please provide the path to the config file."
    echo "Usage: $0 [--time_limit=HH:MM:SS] config_file_path"
    exit 1
fi

# Create a unique temporary file name based on the current timestamp
temp_script=$(mktemp)

# Extract the folder name from the provided path
folder_name=$(basename "$(dirname "$config_file")")

# Write the Slurm script content to the temporary file
cat > "$temp_script" << EOF
#!/bin/bash
#SBATCH --job-name=$folder_name
#SBATCH --output=job_output.log
#SBATCH --error=job_error.log
#SBATCH --partition=l40s
#SBATCH --ntasks=1
#SBATCH --gpus-per-task=1
#SBATCH --mem=30G
#SBATCH --time=$time_limit
#SBATCH --cpus-per-task=16

LOGFILE="\`pwd\`/logs/\$(date '+%Y-%m-%d_%H-%M-%S')_$folder_name.log"

mkdir -p logs
# Run the command
apptainer run --bind /scratch/$USER --nv /home/$USER/container.sif drtvam \$1 >> "\$LOGFILE" 2>&1
EOF

# Make the temporary script executable
chmod +x "$temp_script"

# Submit the job using sbatch
sbatch "$temp_script" "$config_file"

# Remove the temporary script (optional, but recommended to avoid clutter)
rm "$temp_script"

```
4. (if the file is newly created, do `chmod +x optimize_slurm.sh`. You only need to do this once)
5. To run an optimization, call: `./optimize_slurm.sh /home/wechsler/TVAM_patterns/FVB02_sparse_2/config.json`
6. For example, inspect if your job is running. In this case my status is `ST R`, so it runs since 23mins.
```bash
[wechsler@kuma2 ~]$ squeue -u wechsler
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
            456134      l40s   drtvam wechsler  R      23:37      1 kl001
```
7. Run `cat logs/2025-04-24_11-48-26_FVJ5.log` to see your output. The filename has a timestamp and the folder name included, so use the right one.
8. Once finished, synchronize patterns back to the computer. On your local machine, execute 
```bash
scp -r <username>@kuma.hpc.epfl.ch:"/scratch/username/RR01" <local-path>/<where-you-want>
```
9. Perform analysis of the optimization (`histogram.png`, patterns).


## Container file for Dr.TVAM on Scitas
This needs to be done if we update [Dr.TVAM](https://github.com/rgl-epfl/drtvam) or cluster account has no container with the software installed.
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
