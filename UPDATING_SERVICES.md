# Updating services

The process is relatively simple. As soon as a new version of any of the services is released, it's enough to edit your `.env` file pointing to the new version and redeploy the service.

In a recurrent way we update the versions in the repository, so that we also run the automatic builds and we make sure that the deployment process with docker continues working.

However, if at some point there is a new version and the repository has not been updated, you can try to do it on your local and feel free to contribute by opening a merge request with your changes.

## How to update a service

In fact, you can do it the easy way, just delete all the containers and redeploy everything. However, our suggestion is to do it one at a time, double-checking the corresponding Dockerfile and making sure that the service starts correctly. Then you move on to the next one, and so on.

The steps are exactly the same for all of the services. So you can follow the steps below with the service you want to update. For this example, the service is `tor`.

Go to your `.env` file and edit the version parameter:

```conf
# TOR_VERSION=12.0.3 -- just update the variable corresponding to the version 
TOR_VERSION=12.0.4
```

From the root of the repository, stop the service and re-run it.

```shell
$ docker-compose stop tor
$ docker-compose up -d tor
```

Now we can check the logs for a moment to be sure that the service starts correctly.

```shell
$ docker ps | grep tor
06a96296854a   tor:12.0.4               "/bin/sh -c 'tor -f …"   5 minutes ago   Up 3 minute
$ docker logs 06a96296854a -f
```

If after a while you see that the service is working properly, and your node is working as it should, you can delete the old image.

```shell
$ docker images
REPOSITORY         TAG           IMAGE ID       CREATED         SIZE
tor                12.0.4        236c6c6b89da   7 minutes ago   90.2MB
tor                12.0.3        7ad38075c199   4 weeks ago     90.2MB
...
$ docker rmi 7ad38075c199
```

## Troubleshooting

In the case that it does not work well during this process, it can be for two reasons:

- Either something has changed in the service and the Dockerfile is not able to rebuild the image. In that case, we will have to review and update the build process in the project.
- The build process is fine but there is a problem with the service, your configuration or something specific in your machine.

In both cases, you can go back to the previous version that was working before. So simply revert the steps: put the previous version back in the `.env` file and re-run the service. The container you had previously will be deployed and everything should work again.

In the meantime, you can open a thread in the [Discussions](https://github.com/reverse-hash/bitcoin-full-node-with-docker/discussions) section so we can help you.