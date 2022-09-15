# Migration guide for DataPower from local Docker deployment to conatainerized pods running on OpenShift

## Local Deployment

**Instructions**

1. Create a folder for the project and move to that directory.

```
mkdir datapower
cd datapower
```

2. Clone this repo into that parent folder.

```
git clone https://github.com/pbennettibm/datapower-local-dev-test-vm.git
```

3. Create a folder for the virtual DatatPower container volume mounts into your parent folder.

```
mkdir datapower-volume-mounts
```

4. Navigate to the folder you just created and create the following subdirectories inside of it:

- config
- local
- certs
  ```
  cd datapower-volume-mounts
  mkdir config local certs
  ```

4. Change the permissions on these sub directories to allow the container to write it's volume mounted data to them.

- Mac/Linux
  ```
  chmod -R 777 config local certs
  ```
- Windows
  ```
  icacls config local certs /grant *S-1-1-0:F
  ```
  _Note: You may have to use the full path on Windows to correctly change the permissions on the directories. We haven't had the ability to test this yet._

4. Pull the DataPower Docker image.

- Docker
  ```
  docker pull icr.io/integration/datapower/datapower-limited:10.0.4.0
  ```
- Podman
  ```
   podman pull icr.io/integration/datapower/datapower-limited:10.0.4.0
  ```
- You may have to Create a [DockerID](https://hub.docker.com/).
- And then [Login to Docker](https://docs.docker.com/engine/reference/commandline/login/).

5. Create and run the the DataPower container with an interactive shell from the local Docker image with volume mounts.

- Make sure you are in the directory that you created that contains the "config", "local", and "certs" folders.
- Also, note that exposing port 8001 is specific to the validation-flow zip and exposing port 9090 is for virtual DataPower's UI.
  - If you are importing your own flow, you will need to change `-p 8001:8001` to whatever port('s) you need to expose.
- If the below commands do not work and you are running them on a Linux VM, try removing the forward slashes and condensing the command into one line.
- Mac/Linux - Docker
  ```
  docker run -it -e DATAPOWER_ACCEPT_LICENSE=true -e \
  DATAPOWER_INTERACTIVE=true \
  -v $(pwd)/config:/opt/ibm/datapower/drouter/config \
  -v $(pwd)/local:/opt/ibm/datapower/drouter/local \
  -v $(pwd)/certs:/opt/ibm/datapower/root/secire/user/certs \
  -p 9090:9090 \
  -p 8001:8001 \
  --name datapower \
  icr.io/integration/datapower/datapower-limited:10.0.4.0
  ```
- Mac/Linux - Podman
  ```
  podman run -it -e DATAPOWER_ACCEPT_LICENSE=true -e \
  DATAPOWER_INTERACTIVE=true \
  -v $(pwd)/config:/opt/ibm/datapower/drouter/config \
  -v $(pwd)/local:/opt/ibm/datapower/drouter/local \
  -v $(pwd)/certs:/opt/ibm/datapower/root/secire/user/certs \
  -p 9090:9090 \
  -p 8001:8001 \
  --name datapower \
  icr.io/integration/datapower/datapower-limited:10.0.4.0
  ```
- Windows (PowerShell) - Docker
  ```
  docker run -it -e DATAPOWER_ACCEPT_LICENSE=true -e \
  DATAPOWER_INTERACTIVE=true \
  -v ${pwd}/config:/opt/ibm/datapower/drouter/config \
  -v ${pwd}/local:/opt/ibm/datapower/drouter/local \
  -v ${pwd}/certs:/opt/ibm/datapower/root/secire/user/certs \
  -p 9090:9090 \
  -p 8001:8001 \
  --name datapower \
  icr.io/integration/datapower/datapower-limited:10.0.4.0
  ```
- Windows (PowerShell) - Podman
  ```
  podman run -it -e DATAPOWER_ACCEPT_LICENSE=true -e \
  DATAPOWER_INTERACTIVE=true \
  -v ${pwd}/config:/opt/ibm/datapower/drouter/config \
  -v ${pwd}/local:/opt/ibm/datapower/drouter/local \
  -v ${pwd}/certs:/opt/ibm/datapower/root/secire/user/certs \
  -p 9090:9090 \
  -p 8001:8001 \
  --name datapower \
  icr.io/integration/datapower/datapower-limited:10.0.4.0
  ```

6. Enable the UI.

- login: `admin`
- Password: `admin`
- ```
  configure terminal
  ```
- ```
  web-mgmt
  ```
- ```
  admin-state "enabled"
  ```
- ```
  exit
  ```

7. Access the DataPower Gateway on [https://localhost:9090](https://localhost:9090).

- Username and password both default to "admin"

8. Import the validation-flow.zip file using the GUI.

- Initially select the "default" domain.
- There are several screens you will need to click through.
- Accept the configuration as is.

9. Now that you have imported your configuration, test that it is working by navigating to [https://localhost:8001](https://localhost:8001).

- You should see the [JSON Placeholder](https://jsonplaceholder.typicode.com/) page.
- If for some reason it isn't working, attempt to re-import the provided zip file again and ensure that you accept all steps appropriately.

10. Once your configuration is complete, export the config as a zip file and save everything to your mounted volumes.

- In the GUI, click 'Save Configuration'.
- In the CLI enter `write memory`
- Optional:
  - In the GUI, export the zip file if any changes were made to a configuration.

11. In a new terminal window, ensure that the config, and local subdirectories are no longer empty by checking their contents.

- If the certs subdirectory is empty, it is most likely because you did not correctly save your configuration in both the UI and the terminal.
  - To fix this, repeat the step 10.
  - If the certs subdirectory is still empty, it is probably because you are running on a VM that does not have temporary file write access.

12. In your original terminal window, exit the datapower CLI

- Type `exit` twice and then "control+C" to return to the command prompt.

13. (Optional) Delete the Docker container as well as the pulled DataPower Docker/Podman image if you wish.

- Docker
  - ```
    docker rm datapower
    ```
  - ```
    docker rmi icr.io/integration/datapower/datapower-limited:10.0.4.0
    ```
- Podman
  - ```
    podman rm datapower
    ```
  - ```
    podman rmi icr.io/integration/datapower/datapower-limited:10.0.4.0
    ```
