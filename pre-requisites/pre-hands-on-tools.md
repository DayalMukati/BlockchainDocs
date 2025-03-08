---
icon: wrench
---

# Pre-Hands-on Tools

## Pre-Reading Material: Docker, Git, and Linux Ubuntu

This guide introduces three essential tools and platforms used in modern development and operations. It provides an overview along with hands-on command examples to help you get started.

***

### 1. Docker

Docker is a containerization platform that allows you to package applications and their dependencies into portable containers.

#### Basic Docker Commands

**Running a Container**

Run an interactive Ubuntu container:

```bash
docker run -it ubuntu:latest /bin/bash
```

_Explanation:_

* `-it`: Runs the container interactively with a TTY.
* `ubuntu:latest`: Uses the latest Ubuntu image.
* `/bin/bash`: Opens a bash shell inside the container.

**Listing Running Containers**

```bash
docker ps
```

_Sample Output:_

```
CONTAINER ID   IMAGE         COMMAND       CREATED          STATUS          PORTS     NAMES
c3f279d17e0a   ubuntu:latest "/bin/bash"   10 seconds ago   Up 9 seconds              charming_morse
```

**Building an Image from a Dockerfile**

Assuming you have a `Dockerfile` in your current directory:

```bash
docker build -t myapp:1.0 .
```

_Explanation:_

* `-t myapp:1.0`: Tags the image as "myapp" version "1.0".
* `.`: Specifies the current directory as the build context.

**Stopping and Removing a Container**

Stop a container:

```bash
docker stop c3f279d17e0a
```

Remove the container:

```bash
docker rm c3f279d17e0a
```

#### Advanced Docker Commands

**Running in Detached Mode**

Run an Nginx container in the background:

```bash
docker run -d --name my_container nginx:latest
```

_Explanation:_

* `-d`: Detached mode (runs in the background).
* `--name my_container`: Custom container name.

**Executing Commands Inside a Running Container**

```bash
docker exec -it my_container /bin/bash
```

_Explanation:_

* Opens an interactive shell inside the running container.

**Viewing Container Logs**

```bash
docker logs my_container
```

**Inspecting a Container**

```bash
docker inspect my_container
```

_Explanation:_

* Returns detailed JSON output about the container’s configuration and network settings.

#### Managing Docker Images

**Tagging an Image**

```bash
docker tag my_custom_image:1.0 myusername/my_custom_image:latest
```

_Explanation:_

* Tags the image for easier identification and pushes to a registry.

**Pushing an Image to Docker Hub**

```bash
docker push myusername/my_custom_image:latest
```

**Pulling an Image from a Registry**

```bash
docker pull redis:latest
```

#### Managing Networks and Volumes

**Listing Docker Networks**

```bash
docker network ls
```

**Creating a Custom Network**

```bash
docker network create my_network
```

**Connecting a Container to a Network**

```bash
docker network connect my_network my_container
```

**Listing Docker Volumes**

```bash
docker volume ls
```

**Creating and Mounting a Volume**

Create a volume:

```bash
docker volume create my_volume
```

Run a container with a volume mounted:

```bash
docker run -d --name my_db -v my_volume:/var/lib/mysql mysql:latest
```

#### Container Lifecycle Management

**Restarting a Container**

```bash
docker restart my_container
```

**Cleaning Up Unused Resources**

```bash
docker system prune
```

_Explanation:_

* Removes all stopped containers, dangling images, and unused networks. Use with caution.

***

### 2. Git

Git is a distributed version control system designed to handle everything from small to very large projects with speed and efficiency.

#### Basic Git Commands

**Initializing a Repository**

```bash
git init
```

_Explanation:_

* Creates a new `.git` subdirectory in your project folder.

**Cloning an Existing Repository**

```bash
git clone https://github.com/username/repository.git
```

_Explanation:_

* Downloads a complete copy of the repository from GitHub.

**Adding and Committing Changes**

Stage changes:

```bash
git add file.txt
```

Commit changes:

```bash
git commit -m "Add new feature in file.txt"
```

**Working with Branches**

Create and switch to a new branch:

```bash
git checkout -b feature-branch
```

Switch back to the main branch and merge:

```bash
git checkout main
git merge feature-branch
```

**Pushing Changes to a Remote Repository**

```bash
git push origin main
```

_Explanation:_

* `origin`: Default remote repository name.
* `main`: The branch you are pushing to.

**Pulling Updates from Remote**

```bash
git pull origin main
```

***

### 3. Linux Ubuntu

Ubuntu is a user-friendly, Debian-based Linux distribution widely used for both desktop and server environments.

#### Basic Ubuntu Commands

**Navigating the Filesystem**

Print current directory:

```bash
pwd
```

List files and directories:

```bash
ls -l
```

Change directory:

```bash
cd /var/log
```

**File and Directory Management**

Create a directory:

```bash
mkdir my_project
```

Create a new file:

```bash
touch my_project/readme.txt
```

Remove a file:

```bash
rm my_project/readme.txt
```

Remove a directory recursively:

```bash
rm -r my_project
```

**Viewing and Editing Files**

Display file contents:

```bash
cat /etc/hostname
```

Edit a file using Nano:

```bash
nano /etc/hosts
```

_Note:_ Replace `nano` with `vim` if preferred.



Update package lists:

```bash
sudo apt update
```

Upgrade installed packages:

```bash
sudo apt upgrade
```

Install a package (e.g., curl):

```bash
sudo apt install curl
```



Display system and kernel info:

```bash
uname -a
```

Monitor processes:

```bash
top
```

Check disk space usage:

```bash
df -h
```

**Permissions and Ownership**

Change file permissions:

```bash
chmod 755 script.sh
```

Change file ownership:

```bash
sudo chown user:group script.sh
```

***
