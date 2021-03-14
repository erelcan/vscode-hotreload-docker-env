# VS Code Remote Development

In this project, we demonstrate how to utilize VS Code Remote Development extension.

We walk through a docker-compose example which can be applied for both single and multiple service cases.
  - You may find instructions for dockerfile-only version at [1, 2].
  - Also, here are several critical resources to learn more on the concept: [1-5]

Please refer to [VS Code Docker Development Environment Setup](https://github.com/erelcan/vscode-primitive-docker-env/blob/master/Guide.md) for learning the basic concepts on container-based development/debugging.
- You may find the details for dockerfile, docker-compose file, launch.json etc. which we will not detail again in this guide.


## Remote Development with Hot-Reloead

One of the best benefit of VS Code Remote Development extension is that we can make changes in our source files and changes take place immediately in the containers.

We do not need to re-run or re-build the containers unless we make changes to dockerfile or docker-compose file.

**However, we must stop and re-build containers (when we choose "open in container", this is handled implicitly) whenever we make change on dockerfile and docker-compose files. Otherwise, changes will not take effect!!!**

## Environment Configuration (.devcontainer.json)

Define meta-information for the remote development environment with .devcontainer.json!
- Name of the environment.
- The location of dockerfile or docker-compose file
- Which service we intend to run in the environment (should be consistent with the docker-compose file).
- VS Code settings:
  - linux shell type (sh, bash etc.)
  - ptython path
  - linting paths etc.
- Extentions to be used:
  - Any desired VS Code extensions
    - E.g. see examples at [6].
- Ports to be forwarded
- Post create command:
  - Can be used to take actions after container is created.
    - We may use it to install any required packages for development (linters etc.).
- User information
  - Remote user, container user etc.
- Shut down action tells the container what to do after the "script"/service is executed.
  - Use "none" if you do not want the container to go down.
- Workspace folder indicates where to copy the files in the service on the container.


```json
// For format details, see https://aka.ms/devcontainer.json. For config options, see the README at:
// https://github.com/microsoft/vscode-dev-containers/tree/v0.158.0/containers/python-3
{
	"name": "Service1 DevEnv",
	"dockerComposeFile": ["../docker-compose.yml", "../docker-compose.devcontainer.yml"],
    "service": "service1",
    "shutdownAction": "none",
    "workspaceFolder": "/workspace",

	// Set *default* container specific settings.json values on container create.
	"settings": { 
		"terminal.integrated.shell.linux": "/bin/bash",
		"python.pythonPath": "/usr/local/bin/python",
		"python.linting.enabled": true,
		"python.linting.pylintEnabled": true,
		"python.formatting.autopep8Path": "/usr/local/bin/autopep8",
		"python.formatting.blackPath": "/usr/local/bin/black",
		"python.formatting.yapfPath": "/usr/local/bin/yapf",
		"python.linting.banditPath": "/usr/local/bin/bandit",
		"python.linting.flake8Path": "/usr/local/bin/flake8",
		"python.linting.mypyPath": "/usr/local/bin/mypy",
		"python.linting.pycodestylePath": "/usr/local/bin/pycodestyle",
		"python.linting.pydocstylePath": "/usr/local/bin/pydocstyle",
		"python.linting.pylintPath": "/usr/local/bin/pylint"
	},

	// Add the IDs of extensions you want installed when the container is created.
	"extensions": [
		"ms-python.python"
	],
 
	// Use 'forwardPorts' to make a list of ports inside the container available locally.
	// "forwardPorts": [],

	// Use 'postCreateCommand' to run commands after the container is created.
	"postCreateCommand": "pip3 install pylint",

	// Comment out connect as root instead. More info: https://aka.ms/vscode-remote/containers/non-root.
	"remoteUser": "root"
}
```


## Project Structure

*Here is an example project structure (for our example project):*

```txt
- docker-compose.yml
- docker-compose.devcontainer.yml
- service1
  - .vscode
    - launch.json
    - settings.json
  - sources
  - requirements.txt
  - Dockerfile
  - .devcontainer.json
- service2
  - .vscode
    - launch.json
    - settings.json
  - sources
  - requirements.txt
  - Dockerfile
  - .devcontainer.json
```

At the time being, VS Code cannot open more than 1 service in the same window. Hence, we should have .vscode (and its contents) and .devcontainer.json for each service.


## Step by Step

- Install extentions to VS Code:
  - Remote Development
  - Also having python and container extensions may be helpful.
- Press F1 and select "Open folder in container"
- Then, select the service to run/debug.
  - This will build (if not built or not fresh) and run the containers for each service defined in the docker-compose file.
  - Then, the selected service contents will be opened in another VS Code window.
  - Observe that the terminal shows the one which is in the container.
  - Hence, now we are inside the container!
- In separate windows, start other services that you would like to debug.
  - Note that all containers up, so VS Code will attach to running containers.
- Open a file/entrypoint that you would like to execute/debug.
- Then, at run/debug tab of VS Code select the right run configuration.
  - They should be defined in launch.json.
- Now, we can place breakpoints and debug our code!
- After execution of the code completed (or if a service when it is stopped manually), then we can make changes on our code and re-run.
  - We will see that sources are updated in the container immediately!
- In file tab (at toolbar), choose "Close Remote Connection" to stop your connection to the container. However, containers are still up after this.
- To bring down containers, "docker-compose down" at the project folder (where docker-compose.yml exists).


## Gotchas
- We may extend docker-compose file.
  - Handy, if we would like to separete development configuration logic from the docker-compose file.
  - See docker-compose.devcontainer.json
  - We must add paths of all docker-compose files in each .devcontainer.json file belonging to each service.
- The following is added as the command in the docker-compose.devcontainer.json to prevent container to shut down immediately:

```shell
/bin/sh -c "while sleep 1000; do :; done"
```

- We may also add volumes mapping our local paths.
  - Separating such local information from the main docker-compose file will keep it simple and allow collaboration.
- We may need to add a settings.json to .vscode.
  - E.g. for defining linter prefrences etc.


## Security

When we would like to develop on a container at a remote host, security should be considered carefully.

Here is a red flag we should all be aware of [7]:

```txt
When docker is installed on a machine, users with docker access (not necessarily root) can start containers. In particular, they can start containers in priviliged mode, giving the container access to all host devices.

More importantly, A user with access to docker can mount directories owned exclusively by machine root. Since by default, a root user inside the container will have access to mounted root-owned directories inside the container, this will allow any Docker container started by a non-root user to access critical machine stuff.

Therefore, the sequence of having a non-root user install Docker and start containers should not be allowed as it can compromise the whole machine.
```

**Then, what may be our options?**
- Rootless docker [8, 9]
  - Allows to run containers without root priviliges.
  - Please see its known limitations [8]
    - We may also need to setup hardware or other configurations to allow rootless docker.
    - E.g. to enable GPU access for rootless docker, set no-cgroups = true, in /etc/nvidia-container-runtime/config.toml [20].
  - We may run a rootless docker on the remote host with ssh and corresponding VS Code extensions.
  - How to accomplish that?
    - Should we allow the remote user to create rootless containers on the host?
    - Or, should we manage it by the host admin (or CI/CD pipeline)?
- Non-root users
  - We may create non-root users and ssh with that users to the container.
  - How can we accomplish that?
    - Can the admin (CI/CD pipeline) of the host creates a container with allowed users; and then we can attach to these containers with ssh etc.
    - Then, can we have a development container per developer for a given service?
    - Or, should we have a single container, but managed volumes?

## Why we need to develop on remote hosts?
- Need to access to machines with high computational power (GPU servers etc.).
  - Can we bypass this need via a service to execute preliminary tests/experiments on specific hardwares?
  - How about resource limitations?
    - Let's say that developers do not have a specific hardware (e.g. GPU) on their local machines, then how to develop hardware specific solutions?
      - Here, we do not mean to run hours of model training etc., but different behaviour of code on different hardware.
  - Should re-consider operational/security burden vs. buying hardware.
- The limitations of the company culture and infrastructure, along with hard security boundaries.
  - Let say you need to develop on non-admin pc.
    - How will you setup the development environment on such a pc?
    - How about accessing libraries, containers, configuring environment variables etc.?
    - In such a case, a solution is to develop on a remote host container.
      - The host may have docker and library registries (which are fed by an internet available other server, with a controlled flow~).


## Additional Resources
- Securing docker: [10, 11]
- Docker without root priviliges: [12-15]
- VS Code + Remote Host: [16-19]


We will experiment with different approaches and share to verify further.
- Postponed due to lack of hardware resources.
- Would appreciate any contributions on security.
- Also, any other contributions are welcomed:)


## References

[1] [Developing inside a Container](https://code.visualstudio.com/docs/remote/containers)

[2] [Attach to a running container](https://code.visualstudio.com/docs/remote/attach-container)

[3] [Create a development container](https://code.visualstudio.com/docs/remote/create-dev-container)

[4] [Advanced Container Configuration](https://code.visualstudio.com/docs/remote/containers-advanced)

[5] [devcontainer.json reference](https://code.visualstudio.com/docs/remote/devcontainerjson-reference)

[6] [Easy VS Code Docker Remote Containers](https://www.youtube.com/watch?v=KFyRLxiRKAc)

[7] [Stackoverflow - non-root user how to install docker?](https://stackoverflow.com/questions/48473444/non-root-user-how-to-install-docker)

[8] [Run the Docker daemon as a non-root user (Rootless mode)](https://docs.docker.com/engine/security/rootless/)

[9] [Experimenting with Rootless Docker](https://www.docker.com/blog/experimenting-with-rootless-docker/)

[10] [How to Secure Docker for Production Environment?](https://geekflare.com/securing-docker-for-production/)

[11] [10 Docker Security Best Practices](https://snyk.io/blog/10-docker-image-security-best-practices/)

[12] [Building a non-root Docker container](https://dev.to/phuihock/building-a-non-root-docker-container-29ah)

[13] [Docker Without Root Privileges](https://dzone.com/articles/docker-without-root-privileges)

[14] [New Type of Docker: Rootless + Safer: for every Docker user.](https://dev.to/manishfoodtechs/rootless-docker-kick-docker-hackers-and-make-docker-more-secure-new-concept-70g)

[15] [Rootless Docker](https://www.katacoda.com/courses/docker/rootless)

[16] [Connect to remote Docker over SSH](https://code.visualstudio.com/docs/containers/ssh)

[17] [VS Code + Remote Docker containers: Run docker of the remote server via SSH from VS Code](https://medium.com/@minkesh.asati/development-with-the-docker-on-the-remote-server-via-ssh-from-vs-code-ef9e0f2fcbe6)

[18] [Developing inside a container on a remote Docker host](https://code.visualstudio.com/docs/remote/containers-advanced#_developing-inside-a-container-on-a-remote-docker-host)

[19] [VS Code Development Using Docker Containers on Remote Host](https://leimao.github.io/blog/VS-Code-Development-Remote-Host-Docker/)

[20] [Stackoverlow - GPU not recognized with oci-nvidia-hook](https://github.com/containers/podman/issues/3155#issuecomment-573288597)
