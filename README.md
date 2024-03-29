# UW HYAK Code-Server Instructions

Solution to connect VS Code (`code-server`) to HYAK `klone` via a `docker` container using `apptainer`. 

### This solution: 

1. Provides a method to connect VS Code to a compute node on `klone`, preserving the login nodes for the community. While developing your code with connectivity to the server is a great usage of our services, connecting directly to the login node via the `Remote-SSH` extension will result in VS Code server processes running silently in the background and leading to node instability. *As a reminder, we prohibit users running processes on the login node.*

2. Uses a server to develop and execute your code reducing battery usage. `code-server` handles the VSCode background processes, preventing them from slowing down your local machine. 

3. Provides an alternative to a [ProxyJump](https://hyak.uw.edu/docs/hyak101/python/ssh), which for Windows users requires 2-factor authentication to login and change directory. 

### Background Reading

[Coder Home](https://coder.com/)

[Code-server github repo](https://github.com/coder/code-server)

[Code-server documentation](https://coder.com/docs/code-server/latest)

[DockerHub page](https://hub.docker.com/r/codercom/code-server)

[One of a few YouTube videos explaining the benefits of Code-server](https://www.youtube.com/watch?v=h17bHCCEcvI&pp=ygULY29kZS1zZXJ2ZXI%3D)

### Pull the Container

**Note: This solution was developed for UW HYAK users to use SLURM in an HPC environment. The instructions do not necessarily serve every purpose.** 


**Recommended:** perform the following steps after starting an interactive job with `salloc` and load the `apptainer` module. It is best to do this in a workspace directory rather than the storage limited `$HOME` directory. 

```shell terminal=true
$ apptainer pull docker://codercom/code-server
```
This will pull and build the `code-server` container file called `code-server_lastest.sif`

### Launch code-server with SLURM

Download the SLURM batch script.
```shell terminal=true
$ wget https://github.com/finchnSNPs/hyak-code-server/blob/main/code-server.job
```

Edit the job script (find comments "#update this line") to set your code-server session home directory and provide the name of the container if it does not match `code-server_latest.sif`, and edit the `SBATCH` directives as needed.  

```shell terminal=true
$ sbatch code-server.job
Submitted batch job 17440706
```
This script will start a batch job and launch the code-server container. The `SSH` tunneling instructions, including the code-server session password, will be written to the output file (`stdout`). Concatenate the output file for tunneling instructions. The following is an example output.

```shell terminal=true
$ cat code-server.job.17440706
1. SSH tunnel from your workstation using the following command:

   ssh -N -L 8080:n3088:59985 finchkn@klone.hyak.uw.edu

   and point your web browser to http://localhost:8080

2. log in to Code Server using the following credentials:

   password: +WwYzgh7YH/yHzUWNWNS

When done using Code Server, terminate the job by:

1. Sign out of Code Server (Find the three-lines icon Menu and select "Sign out of Code Server")
2. Issue the following command on the login node:

      scancel -f 17440706
```

You can monitor the job with `squeue` and your UWNetID like the following example.

```shell terminal=true
$ squeue -u finchkn
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          17440706   compute code-ser  finchkn  R       3:15      1 n3088
```

The output file will also contains messages from `code-server` as the connection is established. These messages include:

1. The storage location of session associated files - `.local/share/code-server`

2. The location of the configuration file for the session which contains the password that is also printed in the output file - `.config/code-server/config.yaml`

3. Which IP and Port `code-server` HTTP and session is listening to. 

As your session continues, more information will be printed to this output file. 

### Establish the SSH tunnel

Follow the instructions in the output file. Open a new terminal/powershell/Putty window and use the command:
```shell terminal=true
$ ssh -N -L 8080:n3088:59985 finchkn@klone.hyak.uw.edu
... provide UWNetID password
... Duo 2 Factor Authentication
```
The login will appear to hang, but your connection is now open. 

Open a new browser window to http://localhost:8080

Provide the password from the output file in the browser. Select theme and get coding. Extensions can be installed through the browser and will be stored in `.local/share/code-server/extensions`.

If you have trouble with this method, please report them here or to UW Help with HYAK in the message. 

### Using Code-Server without the SLURM script

This section details how to start code-server manually should issues arise using the script or if you would like to change the code-server environment. 

Perform the following steps after starting an interactive job with `salloc` and load the `apptainer` module.

Get the hostname for the compute node where the interactive job is active. 

```shell terminal=true
$ hostname
```

Find an open socket.

```shell terminal=true
$ /mmfs1/sw/pyenv/versions/3.9.5/bin/python -c 'import socket; s=socket.socket(); s.bind(("", 0)); print(s.getsockname()[1]); s.close()'
```

These two pieces become the components of the `--bind-addr` passed to `code-server`.

Start `code-server`. The following is an example that should be changed to fit your purposes:

```shell terminal=true
$ apptainer exec --cleanenv --home /gscratch/scrubbed/finchkn/ /gscratch/scrubbed/finchkn/code-server_latest.sif code-server --bind-addr=n3100:48473 --auth=password
```

Open a new terminal/powershell/Putty window and use the command:
```shell terminal=true
$ ssh -N -L 8080:n3100:48473 finchkn@klone.hyak.uw.edu
... provide UWNetID password
... Duo 2 Factor Authentication
```
The login will appear to hang, but your connection is now open. 

Open a new browser window to http://localhost:8080

The password is in the configuration file, which for this example is:

```shell terminal=true
$ cat /gscratch/scrubbed/finchkn/.config/code-server/config.yaml
bind-addr: 127.0.0.1:8080
auth: password
password: 8RgcAlYVbiGYp/zODwcA
cert: false
```
Paste the password from the config file into the browser.

Sign out of Code Server (Find the three-lines icon Menu and select "Sign out of Code Server"), and stop the process on the compute node (with Control + C ; `^C`) and `exit` the interactive job. 

This configuration file can be edited to change locations for where session files are stored among other settings and preferences. A preferred password can also be provided by changing `password:` in this file. 
