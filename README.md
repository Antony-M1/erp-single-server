# erp-single-server
erp13, ERPNext-13 Production Setup, erp production setup

This Documentation Complete about the setup the `ERPNext-13` in a single server, which means you can setup in the `AWS EC2` or `Your Local Server`(PC) follow all the setup carefully line by line

# Reference Documentations:
* [Single Server](https://github.com/frappe/frappe_docker/blob/main/docs/single-server-example.md)
* [Custom App](https://github.com/frappe/frappe_docker/blob/main/docs/custom-apps.md)

For Complete `Local` and `Production` setup [CLICK HERE](https://github.com/frappe/frappe_docker#custom-images)

### ERP Discuss
* [How to add custom app in production setup?](https://discuss.frappe.io/t/how-to-add-custom-app-in-production-setup/104600)
* [How do I install the human resources module in a Docker setup? ERPNext-14](https://discuss.frappe.io/t/how-do-i-install-the-human-resources-module-in-a-docker-setup/102555)

# Custom App
If you have a `Custom App` follow this setup before start the **Single Server Production Setup**. For the offical Documentation of [Custom App](https://github.com/frappe/frappe_docker/blob/main/docs/custom-apps.md)
 ### Load custom apps through json
 `apps.json` needs to be passed in as build arg environment variable.
```
export APPS_JSON='[
  {
    "url": "https://github.com/frappe/payments",
    "branch": "develop"
  },
  {
    "url": "https://github.com/frappe/erpnext",
    "branch": "version-14"
  },
  {
    "url": "https://user:password@git.example.com/project/repository.git",
    "branch": "main"
  }
]'
export APPS_JSON_BASE64=$(echo ${APPS_JSON} | base64 -w 0)
```
The given commands are used to assign values to environment variables in the Linux terminal. Let's break down what each command does:

1. `export APPS_JSON='[ ... ]'`:
   This command assigns a JSON-formatted string to the `APPS_JSON` environment variable. The JSON represents an array of objects, each containing a URL and a branch name.

2. `export APPS_JSON_BASE64=$(echo ${APPS_JSON} | base64 -w 0)`:
   This command takes the value of the `APPS_JSON` variable, encodes it in Base64 format, and assigns the encoded value to the `APPS_JSON_BASE64` environment variable.

   Here's a step-by-step explanation of what's happening in this command:
   - `echo ${APPS_JSON}`: The `echo` command outputs the value of `APPS_JSON`.
   - `|`: This is a pipe symbol, which allows the output of one command to be used as the input for another.
   - `base64 -w 0`: The `base64` command is used to encode the input in Base64 format. The `-w 0` option prevents line wrapping, ensuring that the output is a single line of Base64-encoded text.
   - `$( ... )`: This is command substitution, where the output of the enclosed command is substituted into the outer command.

So, the second command takes the JSON string stored in `APPS_JSON`, passes it to `echo`, and then pipes the output to `base64` for encoding. The resulting Base64-encoded string is assigned to the `APPS_JSON_BASE64` environment variable.

This kind of encoding is commonly used when you need to pass complex or multi-line data as an environment variable, as some tools or systems may have limitations on handling certain characters or line breaks. By encoding the data in Base64, you ensure it can be safely stored and processed.

**You can also generate base64 string from json file:**
```
export APPS_JSON_BASE64=$(base64 -w 0 /path/to/apps.json)
```
Note: Before running this command make sure you have pulled the `frappe_docker` from this repo.
```
git clone https://github.com/frappe/frappe_docker
cd frappe_docker
```
In side the frappe docker if nevigate in to the `development` folder you can find the `apps-example.json` file you can create a one more file with the name of `app.json` and add all your `custom app` in that.

Note:

* url needs to be `http(s)` git url with `token/auth` in case of `private` repo.
* add dependencies manually in `apps.json` e.g. add `payments` if you are installing `erpnext`
* use fork repo or branch for `ERPNext` in case you need to use your fork or test a PR.

**Example for ERPNext-13**

**Run All The Commands**
```
git clone https://github.com/frappe/frappe_docker && cd frappe_docker
```
```
export APPS_JSON='[
  {
    "url": "https://github.com/frappe/erpnext",
    "branch": "v13.50.2"
  },
  {
    "url": "https://access_token@github.com/my-github/custom_app",
    "branch": "main"
  }
]'
```
```
export APPS_JSON_BASE64=$(echo ${APPS_JSON} | base64 -w 0)
```
```
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=v13.56.2 \
  --build-arg=PYTHON_VERSION=3.9.9 \
  --build-arg=NODE_VERSION=14.19.3 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=custom/app:1.0.0 \
  --file=images/custom/Containerfile .
```

The given command is used to build a Docker image using a specific Dockerfile (`Containerfile`) with various build arguments and tags. Let's break down the command and explain each part:

- `docker build`: This is the Docker command used to build an image.

- `--build-arg`: This option is used to set build-time variables. In this command, there are several `--build-arg` flags used to set different build arguments:
  - `FRAPPE_PATH`: Specifies the GitHub repository URL for the Frappe framework.
  - `FRAPPE_BRANCH`: Specifies the branch/tag of the Frappe framework to be used.
  - `PYTHON_VERSION`: Specifies the version of Python to be installed in the image.
  - `NODE_VERSION`: Specifies the version of Node.js to be installed in the image.
  - `APPS_JSON_BASE64`: Specifies the base64-encoded value of the `APPS_JSON` variable.

- `--tag`: Specifies the name and tag for the built image. In this case, the image will be tagged as `custom/app:1.0.0`.

- `--file`: Specifies the path to the Dockerfile to be used for building the image. In this command, the Dockerfile is located at `images/custom/Containerfile` relative to the current directory.

- `.`: Specifies the build context, which is the current directory where the `docker build` command is executed. The contents of this directory and its subdirectories will be sent to the Docker daemon for building the image.

Overall, this command builds a Docker image using the specified Dockerfile and build arguments, and tags it with a custom name and version.

# Single Server Production Setup
We are assuming you are using linux Ubuntu 16+

### STEP 1
Open the terminal in your server(PC) and run this command

**Run Command**
```
git clone https://github.com/frappe/frappe_docker
```
we are assuming you have domain with the name of `ziptor.com` or some other domain for testing purpose only im giving this domain name.

Change The directory name `frappe_docker` to some Other name.
For example if im hosting a site with a name `ziptor.com` change the folder name
`frappe_docker` to `ziptor`

Run this command to change the directory.

**Run Command**
```
sudo mv frappe_docker ziptor
```
### STEP 2
Create configuration and resources directory

**Run Command**
```
mkdir ~/gitops
```
The `~/gitops` directory will store all the resources that we use for setup. We will also keep the environment files in this directory as there will be multiple projects with different environment variables. You can create a private repo for this directory and track the changes there. The folder will created in the `home\$USER\gitops`.

### STEP 3
**Install Traefik**

Basic `Traefik` setup using docker compose.
Create a file called `traefik.env` in `~/gitops`

**Run Command**
```
echo 'TRAEFIK_DOMAIN=ziptor.com' > ~/gitops/traefik.env
echo 'EMAIL=admin@ziptor.com' >> ~/gitops/traefik.env
echo 'HASHED_PASSWORD='$(openssl passwd -apr1 changeit | sed 's/\$/\\\$/g') >> ~/gitops/traefik.env
```
Note:

* Change the domain from `ziptor.com` to the one used in production. DNS entry needs to point to the Server IP.
* Change the letsencrypt notification email from `admin@ziptor.com` to correct email.
* Change the password from `changeit` to more secure.
env file generated at location `~/gitops/traefik.env` will look like following:
```
TRAEFIK_DOMAIN=ziptor.com
EMAIL=admin@ziptor.com
HASHED_PASSWORD=\$apr1\$POT9B0su\$f9WtnniQOwnPR0fUALZec.
```

**Deploy the traefik container with letsencrypt SSL**

**Run Command**
```
docker compose --project-name traefik \
  --env-file ~/gitops/traefik.env \
  -f overrides/compose.traefik.yaml \
  -f overrides/compose.traefik-ssl.yaml up -d
```
This will make the traefik dashboard available on `ziptor.com` and all certificates will reside in /data/traefik/certificates on host filesystem.

For LAN setup deploy the traefik container without overriding `overrides/compose.traefik-ssl.yaml`.

This command is using Docker Compose to start a Traefik container with specific configurations.

Here's an explanation of the command:

* `--project-name traefik`: Sets the project name for the Docker Compose environment to "traefik".
* `--env-file ~/gitops/traefik.env`: Specifies the path to an environment file (`traefik.env`) that contains environment variables used by the Traefik container.
* `-f overrides/compose.traefik.yaml`: Specifies an additional compose file (`compose.traefik.yaml`) that overrides or extends the base configuration defined in the original `docker-compose.yaml` file.
* `-f overrides/compose.traefik-ssl.yaml`: Specifies another compose file (`compose.traefik-ssl.yaml`) that further overrides or extends the configuration.
* `up -d`: Starts the containers defined in the compose file(s) in the background (`-d` flag) and brings up the services.

In summary, this command starts the Traefik container using the specified environment variables, along with the additional compose files that customize the configuration.

### STEP 4
**Install MariaDB**

Basic `MariaDB` setup using docker compose.
Create a file called `mariadb.env` in `~/gitops`

**Run Command**
```
echo "DB_PASSWORD=changeit" > ~/gitops/mariadb.env
```
Note:
* Change the password from `changeit` to more `secure`.
env file generated at location `~/gitops/mariadb.env` will look like following:
```
DB_PASSWORD=changeit
```
**Example:** 
In the Secure way for the password
```
DB_PASSWORD=jOi8dkUtLu2l0guw
```

**Deploy the mariadb container**
```
docker compose --project-name mariadb --env-file ~/gitops/mariadb.env -f overrides/compose.mariadb-shared.yaml up -d
```
This will make `mariadb-database` service available under `mariadb-network`. Data will reside in `/data/mariadb`

This command is using Docker Compose to start a MariaDB container with specific configurations.

Here's an explanation of the command:
* `--project-name mariadb`: Sets the project name for the Docker Compose environment to "mariadb".
* `--env-file ~/gitops/mariadb.env`: Specifies the path to an environment file (`mariadb.env`) that contains environment variables used by the MariaDB container.
* `-f overrides/compose.mariadb-shared.yaml`: Specifies an additional compose file (`compose.mariadb-shared.yaml`) that overrides or extends the base configuration defined in the original `docker-compose.yaml` file.
* `up -d`: Starts the containers defined in the compose file(s) in the background (`-d` flag) and brings up the services.

In summary, this command starts the MariaDB container using the specified environment variables and the additional compose file that customizes the configuration. The `--project-name` flag allows you to give a specific name to the project, which can be useful when working with multiple Docker Compose environments.

### STEP 5
**Install ERPNext**

**Create First Bench**

Create first bench called `erpnext-one` with `ziptor.com`.

Create a file called `erpnext-one.env` in `~/gitops`

**Run Command**
```
cp example.env ~/gitops/erpnext-one.env
sed -i 's/DB_PASSWORD=123/DB_PASSWORD=changeit/g' ~/gitops/erpnext-one.env
sed -i 's/DB_HOST=/DB_HOST=mariadb-database/g' ~/gitops/erpnext-one.env
sed -i 's/DB_PORT=/DB_PORT=3306/g' ~/gitops/erpnext-one.env
sed -i 's/SITES=`erp.example.com`/SITES=\`ziptor.com\`/g' ~/gitops/erpnext-one.env
echo 'ROUTER=erpnext-one' >> ~/gitops/erpnext-one.env
echo "BENCH_NETWORK=erpnext-one" >> ~/gitops/erpnext-one.env
```
This command sequence performs a series of operations on the `example.env` file and creates a new file `erpnext-one.env` with modified configurations. Here's an explanation of each step:

1. `cp example.env ~/gitops/erpnext-one.env`: Copies the contents of the `ziptor.env` file to a new file named `erpnext-one.env` in the `~/gitops` directory.

2. `sed -i 's/DB_PASSWORD=123/DB_PASSWORD=changeit/g' ~/gitops/erpnext-one.env`: Uses the `sed` command to replace the value of `DB_PASSWORD` in `erpnext-one.env` from `123` to `changeit`. *Note: change the password `changeit` to some secure one*

3. `sed -i 's/DB_HOST=/DB_HOST=mariadb-database/g' ~/gitops/erpnext-one.env`: Modifies the `DB_HOST` value in `erpnext-one.env` by appending `mariadb-database` as the host value.

4. `sed -i 's/DB_PORT=/DB_PORT=3306/g' ~/gitops/erpnext-one.env`: Updates the `DB_PORT` value in `erpnext-one.env` to `3306`.

5. ``sed -i 's/SITES=`erp.example.com`/SITES=\`ziptor.com\`/g' ~/gitops/erpnext-one.env``: Replaces the `SITES` value in `erpnext-one.env` from `erp.example.com` to `ziptor.com`, using backticks to escape the values.

6. `echo 'ROUTER=erpnext-one' >> ~/gitops/erpnext-one.env`: Appends the line `ROUTER=erpnext-one` to the end of `erpnext-one.env` file.

7. `echo "BENCH_NETWORK=erpnext-one" >> ~/gitops/erpnext-one.env`: Appends the line `BENCH_NETWORK=erpnext-one` to the end of `erpnext-one.env` file.

In summary, these commands modify the `ziptor.env` file, creating a new `erpnext-one.env` file with updated values for database configuration (`DB_PASSWORD`, `DB_HOST`, `DB_PORT`), site URLs (`SITES`), and additional configurations (`ROUTER`, `BENCH_NETWORK`).

Note:

Change the password from `changeit` to the one set for MariaDB compose in the previous step.`(Refer STEP 4)`

env file is generated at location `~/gitops/erpnext-one.env`.

Create a yaml file called `erpnext-one.yaml` in `~/gitops` directory:

**Change The ERPNext Version**
By default the erpnext version is `ERPNEXT_VERSION=v14.24.0` *(Note: the documentation created at `2023-May-11`)* to check the version. Open another terminal navigate into the `gitops` folder using this command.
```
cd ~/gitops/
```
After view the file using this command
```
cat erpnext-one.env
```
You can Find `ERPNEXT_VERSION`. if you want you can change mannually or else use this command.
```
sed -i 's/ERPNEXT_VERSION=v14.24.0/ERPNEXT_VERSION=v13.50.1/g' ~/gitops/erpnext-one.env
```
*⚠️ Note⚠️ : Before proceed the another command make sure everything is changed*

**Run Command**
```
docker compose --project-name erpnext-one \
  --env-file ~/gitops/erpnext-one.env \
  -f compose.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.multi-bench.yaml \
  -f overrides/compose.multi-bench-ssl.yaml config > ~/gitops/erpnext-one.yaml
```
For LAN setup do not override `compose.multi-bench-ssl.yaml`.

Use the above command after any changes are made to `erpnext-one.env` file to regenerate `~/gitops/erpnext-one.yaml`. e.g. after changing version to migrate the bench.

**Deploy erpnext-one containers:**

**Run Command**
```
docker compose --project-name erpnext-one -f ~/gitops/erpnext-one.yaml up -d
```

This command uses Docker Compose to deploy and start the containers defined in the `erpnext-one.yaml` file. Here's an explanation of each part of the command:

* `docker-compose`: The command to manage multi-container Docker applications using Compose.
* `--project-name erpnext-one`: Specifies the project name for the containers. In this case, the project name is set to `erpnext-one`.
* `-f ~/gitops/erpnext-one.yaml`: Specifies the path to the Compose YAML file that defines the services and their configurations. In this case, the file is located at `~/gitops/erpnext-one.yaml`.
* `up -d`: Starts the containers defined in the Compose file in detached mode. The `-d` flag ensures that the containers run in the background.0

Overall, this command instructs Docker Compose to start the containers defined in the `erpnext-one.yaml` file, creating the necessary services for the ERPNext application with the project name set to `erpnext-one`.

**Create sites `ziptor.com`:**

`ziptor.com`

**Run Commans**
```
docker compose --project-name erpnext-one exec backend \
  bench new-site ziptor.com --no-mariadb-socket --mariadb-root-password changeit --install-app erpnext --admin-password changeit
```

This command is used to execute a command inside a running container named `backend` in the `erpnext-one` project. Here's an explanation of each part of the command:

* `docker-compose`: The command to manage multi-container Docker applications using Compose.
* `--project-name erpnext-one`: Specifies the project name for the containers. In this case, the project name is set to `erpnext-one`.
* `exec backend`: Executes a command within the running container named `backend`.
* `bench new-site ziptor.com`: This is the command being executed inside the `backend` container. It creates a new site with the domain name `ziptor.com`.
* `--no-mariadb-socket`: Specifies not to use a Unix socket for the MariaDB database.
* `--mariadb-root-password changeit`: Sets the root password for the MariaDB database to `changeit`. *Note: Normaly db root password will be 123. so please avoid using 123 use some secure password or strong password `(Refer STEP 4)`*
* `--install-app erpnext`: Installs the ERPNext application on the newly created site.
* `--admin-password changeit`: Sets the admin password for the ERPNext site to `changeit`.

Overall, this command creates a new site with the domain `ziptor.com` using the ERPNext framework inside the `backend` container of the `erpnext-one` project. It configures the MariaDB database settings and installs the ERPNext application on the site.

`--------------------------------------------SET UP DONE-------------------------------------------------`