# Securely Using SSH Keys in Docker to Access Private Github Repositories - FastRuby.io | Rails Upgrade Service
[Securely Using SSH Keys in Docker to Access Private Github Repositories - FastRuby.io | Rails Upgrade Service](https://www.fastruby.io/blog/docker/docker-ssh-keys.html) 

 [Securely Using SSH Keys in Docker to Access Private Github Repositories - FastRuby.io | Rails Upgrade Service](https://www.fastruby.io/blog/docker/docker-ssh-keys.html) 

 [Securely Using SSH Keys in Docker to Access Private Github Repositories - FastRuby.io | Rails Upgrade Service](https://www.fastruby.io/blog/docker/docker-ssh-keys.html) 

 [Securely Using SSH Keys in Docker to Access Private Github Repositories - FastRuby.io | Rails Upgrade Service](https://www.fastruby.io/blog/docker/docker-ssh-keys.html) 

 [Securely Using SSH Keys in Docker to Access Private Github Repositories - FastRuby.io | Rails Upgrade Service](https://www.fastruby.io/blog/docker/docker-ssh-keys.html) 

 [Securely Using SSH Keys in Docker to Access Private Github Repositories - FastRuby.io | Rails Upgrade Service](https://www.fastruby.io/blog/docker/docker-ssh-keys.html) 

 If you search online for using SSH keys with Docker, to access private Github repositories, you will get a lot of search results, but the solutions you’ll find are almost all out of date, insecure, or fragmentary (i.e. they provide a small snippet of information but not a complete solution). Given how popular both Docker and Github are, I found this quite surprising. We recently had to set up Docker with a Rails application that fetches gems from private repositories. We’re also using Docker Compose, which added to the challenge. [This comment on the Docker project](https://github.com/docker/docker.github.io/issues/12378), which is from February 2021, unfortunately is still accurate:

> There are several questions and answers out there about how to pull from a private repository (using the hosts ssh key & config). A lot of them are not working, not secure or unanswered

After several hours of research and testing, we have a good solution to share. But first let’s take a look at the different approaches to consider.

Option 1: Pass your keys directly to the container
--------------------------------------------------

Don’t do this! You don’t want to upload your ssh keys to Github or anywhere else, as part of the Docker image. Older solutions you’ll find online will recommend copying your keys into the container, and then deleting them at the end of the setup work. This is not a good idea. [Your keys can be recovered by somebody examining the history of the image](https://docs.docker.com/engine/reference/builder/#arg)

![](https://www.fastruby.io/blog/assets/images/passing-secrets-to-docker.png)

Option 2: Using multi-stage builds
----------------------------------

A good overview of this solution is described in the post [Access Private Repositories from Your Dockerfile Without Leaving Behind Your SSH Keys](https://vsupalov.com/build-docker-image-clone-private-repo-ssh-key/). However, as noted in an update to that article, this approach is now considered out of date. It’s also fairly involved, as it requires passing your credentials to an “intermediate” container, before creating the final container, and then “squashing” the intermediate container (this eliminates your ssh keys from the history). We briefly attempted this approach, but had trouble getting it working, and we wanted to try newer, less complex approaches.

Option 3: Using Docker secrets
------------------------------

A newer feature of Docker is [secrets](https://docs.docker.com/engine/swarm/secrets/):

> …a secret is a blob of data, such as a password, SSH private key, SSL certificate, or another piece of data that should not be transmitted over a network or stored unencrypted in a Dockerfile or in your application’s source code. You can use Docker secrets to centrally manage this data and securely transmit it to only those containers that need access to it. Secrets are encrypted during transit and at rest in a Docker swarm. A given secret is only accessible to those services which have been granted explicit access to it, and only while those service tasks are running.

You’ll notice mention of Docker _swarm_ and _services_. The documentation goes on to say “Docker secrets are only available to swarm services, not to standalone containers.”

We were intrigued by the idea of using secrets, but didn’t want to add the complexity of swarm services. We came across the post [Use Your local SSH Keys Inside a Docker Container](https://medium.com/trabe/use-your-local-ssh-keys-inside-a-docker-container-ea1d117515dc) which says: “Docker secrets are meant to be used with Docker Swarm, not with standalone containers. Fear not. Docker compose does support secrets, so using a compose file similar to this will do the trick…” We spent a good deal of time trying this approach, but could not get it to work. The author goes on to say his team is using swarm services, so it’s possible the solution presented is untested - or it’s of course possible we missed something in our attempts 😉

Option 4: Passing a reference to your keys as a command line argument 🎉
------------------------------------------------------------------------

This is what worked for us. First, make sure your ssh key has been added to your ssh agent. If you’re not familiar with this, the [Set up an SSH key](https://support.atlassian.com/bitbucket-cloud/docs/set-up-an-ssh-key/) documentation from BitBucket provides a good overview. Note the `ssh-add` command on MacOS since Monterey (12.0) uses `--apple-use-keychain` instead of `-K`, for example:

```
ssh-add --apple-use-keychain ~/.ssh/id_ed25519 
```

Then, include the following in your Dockerfile:

```
RUN mkdir -p -m 0700 ~/.ssh && ssh-keyscan github.com >> ~/.ssh/known_hosts
RUN --mount=type=ssh bundle install 
```

If you’re wondering about the `--mount=type=ssh` option, [the Docker documentation](https://docs.docker.com/develop/develop-images/build_enhancements/) has a good explanation:

> \[It\] will set the SSH\_AUTH\_SOCK environment variable for that command to the value provided by the host to `docker build`, which will cause any programs in the RUN command which rely on SSH to automatically use that socket. Only the commands in the Dockerfile that have explicitly requested SSH access by defining type=ssh mount will have access to SSH agent connections. The other commands will have no knowledge of any SSH agent being available.

From this point, there are two possible ways to proceed:

### Option 4a: Using Dockerfile only

If you’re not using Docker Compose, you can call `docker build` like this:

```
DOCKER_BUILDKIT=1 docker build --ssh default=$HOME/.ssh/name_of_your_ssh_key . 
```

### Option 4b: Using Docker Compose

To use the `–ssh` option with Docker Compose, you will need to be on at least Docker Compose v2.5. You can check with `docker compose version`. If you are on a Mac using Homebrew, you can upgrade with `brew install docker-compose` (as of August 2022, 2.5 is the current version on Homebrew). Then you can run:

```
docker compose build --ssh default=$HOME/.ssh/name_of_your_ssh_key 
```

Here are sample `Dockerfile` and `docker-compose.yml` files for reference. Replace your_app with an appropriate name for your application. Note we want the Docker compose file to run the tests automatically. You may want yours to do something else.

#### Dockerfile

```
FROM ruby:2.1.10 # this is for an old project

RUN mkdir -p -m 0700 ~/.ssh && ssh-keyscan github.com >> ~/.ssh/known_hosts
ADD . /your_app
WORKDIR /your_app
RUN --mount=type=ssh bundle install 
```

#### docker-compose.yml

```
version: "3.8"
services:
  app:
    build: .
    command: bash -c "bundle exec rspec"
    image: your_app
volumes:
  - .:/your_app 
```
