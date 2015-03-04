# TYPO3 Flow | Docker image
[![Circle CI](https://circleci.com/gh/million12/docker-typo3-flow.png?style=badge)](https://circleci.com/gh/million12/docker-typo3-flow)

This is a [million12/typo3-flow](https://registry.hub.docker.com/u/million12/typo3-flow/) Docker container with [TYPO3 Flow](http://flow.typo3.org) default *base* distribution.

It is an example of how you can build your own TYPO3 Flow installation, perhaps from your own repository, using the abstract [million12/typo3-flow-neos-abstract](https://github.com/million12/docker-typo3-flow-neos-abstract) Docker image. Please read extensive documentation there to learn more.
 
This container contains PHP/Nginx setup. You will need a separate container with database. See the instructions below.

**CAVEAT:** by default, this image contain development (master) version of Flow. To switch to different release, edit `T3APP_BUILD_BRANCH` (can be: branch name or tag name) in Dockerfile and re-build the image. See the [flow.typo3.org](http://flow.typo3.org/) website or [git repository](https://git.typo3.org/Flow/Distributions/Base.git) for info about latest stable version.

## Usage

By default, the container starts and fully configure TYPO3 Flow, incl. setting and migrating the database.

Launch required containers:

```
docker run -d --name=db --env="MARIADB_PASS=my-pass" million12/mariadb
docker run -d --name=flow -p=8080:80 --link=db:db --env="T3APP_VHOST_NAMES=flow dev.flow" million12/typo3-flow
```

You can inspect how the TYPO3 Flow provisioning is going on using `docker logs -f flow`. When you see something like `nginx entered RUNNING state`, you are ready to go.

Don't forget to **map vhost name(s)**, provided in above command via `T3APP_VHOST_NAMES` env variable, on your local machine. Do this by adding following line to your `/etc/hosts` file:  
```
YOUR_DOCKER_IP flow dev.flow
```

**Now go to [http://flow:8080/](http://flow:8080/)** (or respectively http://dev.flow:8080/ for *Development* environment) and enjoy playing with TYPO3 Flow!

#### Development

Launch separate SSH container and link it with running `flow` container:
``` 
docker run -d --name=dev --link=db:db --link=flow:web --volumes-from=flow -p=1122:22 --env="IMPORT_GITHUB_PUB_KEYS=your-github-username-here" million12/php-app-ssh
```  
Please provide your GitHub username to `IMPORT_GITHUB_PUB_KEYS` env variable; your public SSH key will be imported from there. After container is launched, your key will be automatically added inside container and you can log in using **www user**:  
```
ssh -p 1122 www@YOUR_DOCKER_IP
```

You will find TYPO3 Flow in `typo3-app` directory (by default). You can do all `./flow` commands from there, upload files via SFTP, in general: do all development using it. Learn more about the SSH container on [million12/docker-php-app-ssh](https://github.com/million12/docker-php-app-ssh) repository page.

Note: the good part with that side SSH container is that it is build on top of [million12/php-app](https://github.com/million12/docker-php-app) image, exactly the same which is used as a base iamge for [million12/typo3-flow-neos-abstract](https://github.com/million12/docker-typo3-flow-neos-abstract). Therefore you can be sure you have the same PHP configuration, the same software as inside container with running TYPO3 Flow. In practise it means: no quirk issues due to differences in environments.


## Usage with Docker Compose (old fig)

Instead of manually launching all containers like described above, you can use [Docker Compose](https://docs.docker.com/compose/). Docker Compose is an orchestration tool and it is very easy to use. If you do not have it yet, install it first. 

The [docker-compose.yml](docker-compose.yml) config file is already provided, so you can **launch TYPO3 Flow with just one command**:  
```
docker-compose up [-d]
```

And you're done.

You can uncomment the `dev` container if you need SSH access. Remember to supply your GitHub username to `IMPORT_GITHUB_PUB_KEYS` env variable. Provided account has to have your public SSH key added.


## Running tests

It is very easy to run unit, functional and Behat tests against TYPO3 Flow with this container. We will use [million12/php-app](https://github.com/million12/docker-php-app) image, link it with running Flow container and run the tests from there. Container million12/php-app is actually a base image for TYPO3 Flow container, so we can be sure we are working in the same environment.

Here's how you can run all tests:  
```
docker run -d --name=db --env="MARIADB_PASS=my-pass" million12/mariadb
docker run -d --name=flow-testing --link=db:db --env="T3APP_VHOST_NAMES=behat.dev.flow" --env="T3APP_DO_INIT_TESTS=true" --env="T3APP_DO_INIT=false" million12/typo3-flow

# Wait till Flow container is fully provisioned (docker logs -f flow-testing). Then launch tests:
docker run -ti --volumes-from=flow-testing --link=flow-testing:web --link=db:db million12/php-app "
  su www -c \"
    cd ~/typo3-app && \
    echo -e '\n\n======== RUNNING TYPO3 FLOW TESTS =======\n\n' && \
    bin/phpunit -c Build/BuildEssentials/PhpUnit/UnitTests.xml && \
    bin/phpunit -c Build/BuildEssentials/PhpUnit/FunctionalTests.xml
  \"
"
```  
and you should see all tests nicely passing ;-)

## Authors

Author: Marcin Ryzycki (<marcin@m12.io>)  

---

**Sponsored by** [Typostrap.io - the new prototyping tool](http://typostrap.io/) for building highly-interactive prototypes of your website or web app. Built on top of TYPO3 Neos CMS and Zurb Foundation framework.
