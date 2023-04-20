## Starting SWCI In web Mode 

`Ex which is used to run: Credit Card Fraud Detection`
<br><br>
Url to the same: <a href="https://github.com/HewlettPackard/swarm-learning/tree/master/examples/fraud-detection#-credit-card-fraud-detection">Fraud Detection </a>


- To do this if you have previously run the given example as stated above make sure to remove the docker volume and network by running these cmds 

```bash
docker network rm host-1-net
```

```bash
docker volume rm sl-cli-lib
```

**Remove the Workspace directory**

`To remove workspace dir make sure to run the su command and enter the password of the host user account through which you have logged in to ur desktop`
<br>

```bash
rm -rf workspace
```

Now try running the fraud detection Example

**If not in ur current dir**
```bash
cd swarm-learning
```

**On host-1, create a temporary workspace directory, fraud-detection example, and gen-cert utility.**
```bash
mkdir workspace
cp -r examples/fraud-detection workspace/
cp -r examples/utils/gen-cert workspace/fraud-detection/
```

**This example has a separate private `data-and-scratch` directories for each user or ML node. Create the respective directories and copy `data-and-scratch` directory. Running this example creates a scratch directory for each user and saves the trained Swarm model in the directory at the end of the training.**
```bash
mkdir workspace/fraud-detection/user1 workspace/fraud-detection/user2
mkdir workspace/fraud-detection/user3 workspace/fraud-detection/user4
cp -r workspace/fraud-detection/data-and-scratch workspace/fraud-detection/user1/
cp -r workspace/fraud-detection/data-and-scratch workspace/fraud-detection/user2/
cp -r workspace/fraud-detection/data-and-scratch workspace/fraud-detection/user3/
mv workspace/fraud-detection/data-and-scratch workspace/fraud-detection/user4/
```

```bash
./workspace/fraud-detection/gen-cert -e fraud-detection -i 1
```

```bash
gedit swop1_profile.yaml
```

and change the IP address to the Public IP of the machine in which you are running

**Search and replace all occurrences of <CURRENT-PATH> tag in swarm_fd_task.yaml and swop1_profile.yaml files with $(pwd)**
```bash
sed -i "s+<CURRENT-PATH>+$(pwd)+g" workspace/fraud-detection/swop/swop*_profile.yaml workspace/fraud-detection/swci/taskdefs/swarm_fd_task.yaml
```

**Create a docker volume and copy Swarm Learning wheel file.**
```bash
docker volume create sl-cli-lib
docker container create --name helper -v sl-cli-lib:/data hello-world
docker cp -L lib/swarmlearning-client-py3-none-manylinux_2_24_x86_64.whl helper:/data
docker rm helper
```

**Create a docker network for SN, SWOP, SWCI, SL, and user containers running on the same host.**
```bash
docker network create host-1-net
```

**Run Swarm Network node (SN1) - sentinel node.**
<br>
`Make sure to start the APLS server before you run this command`
<br>

```bash
./scripts/bin/run-sn -d --rm --name=sn1 \
--network=host-1-net --host-ip=sn1 --sentinel \
--key=workspace/fraud-detection/cert/sn-1-key.pem \
--cert=workspace/fraud-detection/cert/sn-1-cert.pem \
--capath=workspace/fraud-detection/cert/ca/capath \
--apls-ip=10.0.2.15 --sn-api-port=30304
```

`Wait until you get swarm.blCnt : INFO : Starting SWARM-API-SERVER on port: 30304 on the terminal`

**Meanwhile open a new terminal and check for docker logs to monitor the Sentinel SN node and wait for the node to finish initializing**
```bash
docker logs -f sn1
```

**Run Swarm Operator node (SWOP1)**
```bash
./scripts/bin/run-swop -d --rm --name=swop1 \
--network=host-1-net --usr-dir=workspace/fraud-detection/swop \
--profile-file-name=swop1_profile.yaml \
--key=workspace/fraud-detection/cert/swop-1-key.pem \
--cert=workspace/fraud-detection/cert/swop-1-cert.pem \
--capath=workspace/fraud-detection/cert/ca/capath \
-e SWOP_KEEP_CONTAINERS=True \
-e http_proxy= -e https_proxy= --apls-ip=10.0.2.15
```

**Run SWCI node (SWCI1). It creates, finalizes and assigns below task to task-framework for sequential execution: and connecting it to port 30306**
```bash
./scripts/bin/run-swci -ti --rm --name=swci1 \
--network=host-1-net --usr-dir=workspace/fraud-detection/swci \
--init-script-name=swci-init \
--key=workspace/fraud-detection/cert/swci-1-key.pem \
--cert=workspace/fraud-detection/cert/swci-1-cert.pem \
--capath=workspace/fraud-detection/cert/ca/capath \
-e http_proxy= -e https_proxy= --apls-ip=10.0.2.15 -e SWCI_MODE='WEB' -p 30306:30306
```

`Finally you will get a message that the Starting SWCI mode in 30306 after it executes all the tasks`

- A default `testContext` will be created
- So in your jupyter notebook you can simply switch to `testContext` instead of creating a new context however you are free to do so

## Simulating the Similar Workflow using Jupyter Notebook by using SWCI API's
<br>
*Prerequisites: *
- Swarm Learning Infrastructure is setup and ready.
- SWCI container is running in WEB mode (-e SWCI_MODE='WEB')
- There should be explicit port forwarding for SWCI_WEB_PORT while running the SWCI container (ex: -p 30306:30306)
- Swarm learning wheel package should be installed in the python environment where we run this file.

- For more reference regarding SWCI API's go through the documentation <a href="https://github.com/HewlettPackard/swarm-learning/blob/master/docs/User/SWCI_APIs.md">SWCI API</a>

**A reference code to the same can be viewed here:**
- Run these codes in different cells and match the output to the tasks getting executed on the terminal after starting the WEB mode

```bash
# Import swci from the swarmlearning whl package
import swarmlearning.swci as sw

swciServerName = '10.0.2.15'
snServerName = '10.0.2.15'
```

```bash
# Connect to the SWCI via SWCI_WEB_PORT
s = sw.Swci(swciServerName,port=30306) #30306 is the default port
```

```bash
# Switches the context to testContext which has already been created when u  ran the SWCI Node
print(s.switchContext('test-fd'))
```

```bash
s.cd('/platform/swarm/usr')
s.uploadTaskDefintion("/home/hpecty/Desktop/user_env_tf_build_task1.yaml")
s.registerTask('user_env_tf_build_task1.yaml',finalize=False)
```

**Note: Once you finalize the task you cannot make any further changes to the file**
```bash
s.finalizeTask('user_env_tf_build_task2')
```

```bash
s.getTaskInfo('user_env_tf_build_task2')
```

```bash
s.getTaskBody('user_env_tf_build_task2')
```

```bash
# Lists all the tasks that includes root task
print(s.listTasks())
```

```bash
s.assignTask('user_env_tf_build_task2','defaulttaskbb.taskdb.sml.hpe',1)
```

```bash
s.resetTaskRunner('defaulttaskbb.taskdb.sml.hpe')
```

```bash
#Executing Default Task
s.executeTask('user_env_tf_build_task2')
```

```bash
s.uploadTaskDefintion("/home/hpecty/Desktop/swarm_fd_task2.yaml")
s.registerTask('swarm_fd_task2.yaml',finalize=False)
```

```bash
s.finalizeTask('swarm_fd_task1')
```

```bash
s.getTaskInfo('swarm_fd_task1')
```

```bash
s.getTaskBody('swarm_fd_task1')
```

```bash
# Lists all the created Contexts
print(s.listTasks())
```

```bash
s.assignTask('swarm_fd_task1','defaulttaskbb.taskdb.sml.hpe',4)
```

```bash
s.resetTaskRunner('defaulttaskbb.taskdb.sml.hpe')
```

```bash
s.executeTask('swarm_fd_task1')
```

```bash
s.sleep(15)
```

```bash
s.resetTaskRunner('defaulttaskbb.taskdb.sml.hpe')
```

```bash
s.listTrainingContracts()
```

```bash
s.resetTrainingContract('defaultbb.cqdb.sml.hpe')
```



