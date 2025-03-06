# SCITAS_drtvam

1. Open CMD in Windows
2. Do `ssh <username>@kuma.hpc.epfl.ch`
3. Copy a `config.json` from your local computer to cluster
```
scp <local-path>/config.json <username>@kuma.hpc.epfl.ch:/home/<username>/<path-to-config>/
```
4. You can also edit an existing file on the cluster with `nano <path-to-config>/config.json`
5. To launch an optimization job
```bash
srun --pty -p l40s -n 1 --gpus-per-task=1 --mem 30G --time=02:00:00 apptainer run --nv container_pip.sif drtvam /home/<username>/<path-to-config>/config.json
```
5. Do not close this open shell until the optimization is finished, otherwise the job is killed.
6. Synchronize patterns back to the computer
