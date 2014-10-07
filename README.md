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
server-unknown:> admin config server http://ec2-54-91-89-117.compute-1.amazonaws.com:9393
```
> Replacing `localhost` with your host running Docker (if running inside boot2docekr for example, one can use
`boot2docker ip` to get the IP to connect to).

## Your first stream!







**Note** that http://localhost:9393 is actually the shell default, so if running with a local Docker demon
