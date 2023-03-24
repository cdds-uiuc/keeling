`module avail`
- Shows all available modules.

`ls /usr/bin`
- Shows all commands.

`sinfo -l | more`
- Shows partitions and their availabilities, node amounts, nodes, and other information. Piped into more.
- Partition types
  - seseml - For python or MATLab
  - sesempi - 8 nodes
  - sesebig - 8-32 nodes.
 - Oversubs - Indicates that you can put more than one job on that node
 - `sinfo -Nel | more`
   - Gives info on each node
 
Nodes
- Each has 40 threads.
- Most computing can be done on keeling-j nodes.
  - 48 cores, 256 GB

`squeue`
- Prints currently queued jobs.
- `squeue -l`
  - Prints status of jobs, too.
- `sqq`
  - Also shows priority

`sbatch - p sesempi (or sesebig) -N 4 -n 192 -mem 72 G -time 24:00:00`
- To run a batch on a partition of choosing.
- -p - Partition
- -N - Number of nodes
- -n - Number of cores
- -mem - Memory
- -time - Time alotted
