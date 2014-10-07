# Spring XD Singlenode

The single node docker image is the easiest to get started with. It runs everything you need in a single container. The other component we will use is the XD Shell.  The shell is a more user-friendly front end to the REST API which Spring XD exposes to clients. 

## Retrieving the images

To retrieve the single node image execute the following :

    docker pull springxd/singlenode

To retrieve the shell image execute the following :

    docker pull springxd/shell

## Start the singlenode and the shell  
### Start the singlenode
To start it, you just need to execute the following command:

        docker run --name singlenode \
            -d \
            -p 9393:9393\
            springxd/singlenode


> Note how we exposed the `9393` port above. This is the http port that will receive http requests from the shell.

Now let's observe singlenode's log by executing the following:

    sudo docker logs -f singlenode

### Start the shell
Now lets start the shell:

        docker run --name shell \
            -it \
            springxd/shell
You should see the following prompt :
```
 _____                           __   _______
/  ___|          (-)             \ \ / /  _  \
\ `--. _ __  _ __ _ _ __   __ _   \ V /| | | |
 `--. \ '_ \| '__| | '_ \ / _` |  / ^ \| | | |
/\__/ / |_) | |  | | | | | (_| | / / \ \ |/ /
\____/| .__/|_|  |_|_| |_|\__, | \/   \/___/
      | |                  __/ |
      |_|                 |___/
eXtreme Data
1.0.1.BUILD-SNAPSHOT | Admin Server Target: http://localhost:9393
-------------------------------------------------------------------------------
Error: Unable to contact XD Admin Server at 'http://localhost:9393'.
Please execute 'admin config info' for more details.
-------------------------------------------------------------------------------

Welcome to the Spring XD shell. For assistance hit TAB or type "help".
server-unknown:>
```
To connect to the singlenode execute the following command from the prompt:

```
server-unknown:> admin config server http://<host>:9393
```
> Replacing "<"host">" with your host running Docker.

## Create a ticktock stream
In this simple example, the time source simply sends the current time as a message each second, and the log sink outputs it using the logging framework at the WARN logging level.

From the shell ***xd:>*** prompt type the following and press ***return***
```
stream create --name ticktock --definition "time | log" --deploy
```
Now view the logs -f output from the singlenode you should see:
```
20:18:51,104  INFO DeploymentsPathChildrenCache-0 server.ContainerRegistrar - Deploying module 'time' for stream 'ticktock'
20:18:51,277  INFO DeploymentsPathChildrenCache-0 server.ContainerRegistrar - Deploying module [ModuleDescriptor@6ded4936 moduleName = 'time', moduleLabel = 'time', group = 'ticktock', sourceChannelName = [null], sinkChannelName = [null], sinkChannelName = [null], index = 0, type = source, parameters = map[[empty]], children = list[[empty]]]
20:18:52,360  INFO task-scheduler-1 sink.ticktock - 2014-10-07 20:18:52
20:18:52,384  INFO Deployer server.StreamDeploymentListener - Deployment status for stream 'ticktock': DeploymentStatus{state=deployed}
20:18:52,387  INFO Deployer server.StreamDeploymentListener - Stream Stream{name='ticktock'} deployment attempt complete
20:18:53,362  INFO task-scheduler-5 sink.ticktock - 2014-10-07 20:18:53
20:18:54,363  INFO task-scheduler-6 sink.ticktock - 2014-10-07 20:18:54
20:18:55,364  INFO task-scheduler-6 sink.ticktock - 2014-10-07 20:18:55
```
To destroy the stream go back to the shell and from the ***xd:>*** prompt type the following and press ***return***
```
stream destroy ticktock
```
> It is also possible to stop and restart the stream instead, using the undeploy and deploy commands. 

## Create a http source stream
In this example, we will create a http source that will listen for http posts on the 9000 port and will write the results to our log.
### Cleanup
First lets stop our last example.  Since we are monitoring our log we can hit the ***ctrl-c*** to stop the log tailing.  Now we want to stop our singlenode instance and it can be done by executing the following:
```
docker stop singlenode
```
### Create the hello world http stream
Now lets start up our singlenode with the 9000 port open and this time will set our name for the container to be httpSourceTest:

        docker run --name httpsourcetest \
            -d \
            -p 9393:9393\
            -p 9000:9000\
            springxd/singlenode
Again now let's find monitor our singlenode instance by executing the following:

    sudo docker logs -f singlenode
> Notice we are not restarting our shell.  This is because the shell is not making a sustained connection to the singlenode, but rather executing individual restful calls.  This behavior will change one you invoke XD security.

So to create the stream to receive http posts, go to the the shell and from the ***xd:>*** prompt type the following and press ***return***:
```
stream create httpsource --definition "http|log" --deploy
```
Now lets post a http "hello world" message to XD singlenode.  From the shell ***xd:>*** prompt type the following and press ***return***
```
http post --target http://<host>:9000 --data "hello world"
```
The result you will see in the singlenode log will be:
```
21:06:08,208  INFO pool-11-thread-4 sink.httpsource - hello world
```

