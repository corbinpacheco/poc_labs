# GPU jobs with Metronome

To Run GPU jobs with Metronome, you need to make sure you cluster satisfies the required prerequisites:

1.  Has GPU machines
2.  Has the drivers needed to use the GPUs installed
3.  Has DC/OS 1.12.2 or later
4.  Enable the GPU scheduling capability for Metronome
5.  Submit jobs to Metronome that both use the UCR and have a container specified

Deploying a GPU enabled cluster

This is a bit wonky.  For the purposes of this lab, I've used the "old terraform repo" and specified an older AMI that has the GPU drivers installed.  

See main.tf.

to-do is to fix the terraform and use the universal installer.  The cluster as-is deploys agents with coreOS for the masters and CPU based private agent and centOS for the GPU machines.


Once deployed, a manual change is needed to the metronome config to enable GPU scheduling.  

Add:

`METRONOME_GPU_SCHEDULING_BEHAVIOR=restricted` to `/opt/mesosphere/etc/metronome` and restart metronome

`# sudo systemctl restart docs-metronome`

The UI for Metronome, doesn't yet support natively the UCR and GPU support, so all jobs need to be scheduled via the CLI or API.

The following job specification is an example:

{
    "id": "gpu-test",
    "description": "A job that runs nvidia-smi",
    "run": {
        "cmd": "nvidia-smi",
	"cpus": 1,
        "gpus": 1,
        "mem": 1024,
        "disk": 0,
        "ucr": {
           "image": {
              "id": "nvidia/cuda:9.0-base"
                }
         }
     }
}

Create the file and submit it:

`$ dcos job add gpu-test`

This job has no schedule so to run it:

`$ dcos job run gpu-test`

Run ID: 20190221181651Zz7OM

To query the results:

`$ dcos task --completed | grep 20190221181651Zz7OM`

20190221181651Zz7OM.gpu-test5          10.0.5.235  root    F    gpu-test5_20190221181651Zz7OM.fb5325c1-3604-11e9-b8f9-aaf24caedfea          fe4e7d3f-30be-46ee-9eea-040c7f3987b6-S1   ---    ---

`$ dcos task --completed log gpu-test5_20190221181651Zz7OM.fb5325c1-3604-11e9-b8f9-aaf24caedfea stdout`

|   0  Tesla M60           Off  | 00000000:00:1E.0 Off |                    0 |
| N/A   28C    P8    15W / 150W |      0MiB /  7613MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+



