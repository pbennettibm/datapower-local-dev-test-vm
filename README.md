# Migration guide for DataPower from local Docker deployment to conatainerized pods running on OpenShift

## Local Deployment

**Pre-Reqs**

1. Install [Docker](https://docs.docker.com/get-docker/) or [Podman](https://podman.io/getting-started/installation) locally on your own device.
  - If you choose to use Podman, please change the appropriate commands below from `docker` to `podman`.
  
**Instructions**

1. Create a folder for the project.
2. Create the following subdirectories inside this folder.
  - config
  - local
  - certs
3. Change the permissions on these sub directories.
  - Mac 
    ```
    chmod -R 777 config local certs
    ```
  - Windows 
    ```
    icacls config local certs /grant *S-1-1-0:F
    ```
    _Note: You may have to use the full path on Windows to correctly change the permissions on the directories. We haven't had the ability to test this yet._
4. Pull the DataPower Docker image .
   ```
   docker pull icr.io/integration/datapower/datapower-limited:10.0.4.0
   ```
  - You may have to Create a [DockerID](https://hub.docker.com/).
  - And then [Login to Docker](https://docs.docker.com/engine/reference/commandline/login/).
5. Create and run the the DataPower container with an interactive shell from the local Docker image with volume mounts.
  - Make sure you are in the directory that you created that contains the "config", "local", and "certs" folders.
  - Mac
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
    _Note: Exposing port 8001 is specific to the validation-flow.zip.  Expose the correct port if importing a different flow._
  - Windows (PowerShell)
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
    _Note: Exposing port 8001 is specific to the validation-flow.zip.  Expose the correct port if importing a different flow._
6. Enable the UI.
  - login: `admin`
  - Password: `admin`
  - ```
    co
    ```
  - ```
    web-mgmt 0 9090 9090
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
  - In the CLI enter ```write memory```
  - Optional:
    - In the GUI, export the zip file if any changes were made to a configuration.
11. Exit the datapower CLI
  - Type `exit` twice and then "control+C".
12. Ensure that the config, local, and certs subdirectories are no longer empty.
13. Stop and delete the Docker container as well as remove the pulled DataPower Docker image if you wish.
  - ```
    docker stop datapower
    ```
  - ```
    docker rm datapower
    ```
  - ```
    docker rmi icr.io/integration/datapower/datapower-limited:10.0.4.0
    ```

## OpenShift Deployment

**Pre-Reqs**

1. An OpenShift installation on a cloud provider.
2. Login credentials and url address to access the OpenShift UI.

**Instructions**

1. Clone and run the [datapower-operator-scripts](https://github.ibm.com/Patrick-Bennett/datapower-operator-scripts-autogenerate) repo and follow its instructions to extract the necessary yaml files and deploy using your chosen method.
