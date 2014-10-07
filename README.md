# Spring XD Singlenode

The single node docker image is the easiest to get started with. It runs everything you need in a single container. The other component we will use is the XD Shell, this allows us to create, deploy and destroy streams and jobs.

## Retrieving the images

To retrieve the single node image execute the following :

    docker pull springxd/singlenode

To retrieve the shell image execute the following :

    docker pull springxd/shell

## Start the singlenode and the shell    
To start it, you just need to execute the following command:

        docker run --name singlenode \
            -d \
            -p 9393:9393\
            springxd/singlenode


> Note how we exposed the `9393` port above. This is the http port that will receive http requests from the shell.

Now let's observe singlenode's log by executing the following:

    sudo docker logs -f singlenode


Now lets start the shell

        docker run --name shell \
            -it \
            springxd/shell

Once you see the following text in the log 






Replacing `localhost` with your host running Docker (if running inside boot2docekr for example, one can use
`boot2docker ip` to get the IP to connect to).

**Note** that http://localhost:9393 is actually the shell default, so if running with a local Docker demon
