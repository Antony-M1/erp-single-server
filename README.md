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

Create first bench called `erpnext-one` with `one.ziptor.com` and `two.ziptor.com`

Create a file called `erpnext-one.env` in `~/gitops`

**Run Command**
```
cp example.env ~/gitops/erpnext-one.env
sed -i 's/DB_PASSWORD=123/DB_PASSWORD=changeit/g' ~/gitops/erpnext-one.env
sed -i 's/DB_HOST=/DB_HOST=mariadb-database/g' ~/gitops/erpnext-one.env
sed -i 's/DB_PORT=/DB_PORT=3306/g' ~/gitops/erpnext-one.env
sed -i 's/SITES=`erp.ziptor.com`/SITES=\`one.ziptor.com\`,\`two.ziptor.com\`/g' ~/gitops/erpnext-one.env
echo 'ROUTER=erpnext-one' >> ~/gitops/erpnext-one.env
echo "BENCH_NETWORK=erpnext-one" >> ~/gitops/erpnext-one.env
```

This command sequence performs a series of operations on the `example.env` file and creates a new file `erpnext-one.env` with modified configurations. Here's an explanation of each step:

1. `cp example.env ~/gitops/erpnext-one.env`: Copies the contents of the `ziptor.env` file to a new file named `erpnext-one.env` in the `~/gitops` directory.

2. `sed -i 's/DB_PASSWORD=123/DB_PASSWORD=changeit/g' ~/gitops/erpnext-one.env`: Uses the `sed` command to replace the value of `DB_PASSWORD` in `erpnext-one.env` from `123` to `changeit`. *Note: change the password `changeit` to some secure one*

3. `sed -i 's/DB_HOST=/DB_HOST=mariadb-database/g' ~/gitops/erpnext-one.env`: Modifies the `DB_HOST` value in `erpnext-one.env` by appending `mariadb-database` as the host value.

4. `sed -i 's/DB_PORT=/DB_PORT=3306/g' ~/gitops/erpnext-one.env`: Updates the `DB_PORT` value in `erpnext-one.env` to `3306`.

5. `sed -i 's/SITES=`erp.ziptor.com`/SITES=\`one.ziptor.com\`,\`two.ziptor.com\`/g' ~/gitops/erpnext-one.env`: Replaces the `SITES` value in `erpnext-one.env` from `erp.ziptor.com` to `one.ziptor.com` and `two.ziptor.com`, using backticks to escape the values.

6. `echo 'ROUTER=erpnext-one' >> ~/gitops/erpnext-one.env`: Appends the line `ROUTER=erpnext-one` to the end of `erpnext-one.env` file.

7. `echo "BENCH_NETWORK=erpnext-one" >> ~/gitops/erpnext-one.env`: Appends the line `BENCH_NETWORK=erpnext-one` to the end of `erpnext-one.env` file.

In summary, these commands modify the `ziptor.env` file, creating a new `erpnext-one.env` file with updated values for database configuration (`DB_PASSWORD`, `DB_HOST`, `DB_PORT`), site URLs (`SITES`), and additional configurations (`ROUTER`, `BENCH_NETWORK`).

Note:

Change the password from `changeit` to the one set for MariaDB compose in the previous step.`(Refer STEP 4)`

env file is generated at location `~/gitops/erpnext-one.env`.

Create a yaml file called `erpnext-one.yaml` in `~/gitops` directory:

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

**Create sites `one.ziptor.com`:**

`one.ziptor.com`

**Run Commans**
```
docker compose --project-name erpnext-one exec backend \
  bench new-site one.ziptor.com --no-mariadb-socket --mariadb-root-password changeit --install-app erpnext --admin-password changeit
```

This command is used to execute a command inside a running container named `backend` in the `erpnext-one` project. Here's an explanation of each part of the command:

* `docker-compose`: The command to manage multi-container Docker applications using Compose.
* `--project-name erpnext-one`: Specifies the project name for the containers. In this case, the project name is set to `erpnext-one`.
* `exec backend`: Executes a command within the running container named `backend`.
* `bench new-site one.ziptor.com`: This is the command being executed inside the `backend` container. It creates a new site with the domain name `one.ziptor.com`.
* `--no-mariadb-socket`: Specifies not to use a Unix socket for the MariaDB database.
* `--mariadb-root-password changeit`: Sets the root password for the MariaDB database to `changeit`. *Note: Normaly db root password will be 123. so please avoid using 123 use some secure password or strong password `(Refer STEP 4)`*
* `--install-app erpnext`: Installs the ERPNext application on the newly created site.
* `--admin-password changeit`: Sets the admin password for the ERPNext site to `changeit`.

Overall, this command creates a new site with the domain `one.ziptor.com` using the ERPNext framework inside the `backend` container of the `erpnext-one` project. It configures the MariaDB database settings and installs the ERPNext application on the site.

**Create custom domain to existing site**

In case you need to point` custom domain` to existing site follow these steps. Also useful if custom domain is required for LAN based access.

**Create environment file**
```
echo "ROUTER=custom-one-example" > ~/gitops/custom-one-example.env
echo "SITES=\`custom-one.ziptor.com\`" >> ~/gitops/custom-one-example.env
echo "BASE_SITE=one.ziptor.com" >> ~/gitops/custom-one-example.env
echo "BENCH_NETWORK=erpnext-one" >> ~/gitops/custom-one-example.env
```

Note:

* Change the file name from `custom-one-example.env` to a logical one.
* Change `ROUTER` variable from `custom-one.ziptor.com` to the one being added.
* Change `SITES` variable from `custom-one.ziptor.com` to the one being added.  You can add multiple sites quoted in backtick (`) and separated by commas.
* Change `BASE_SITE` variable from `one.ziptor.com` to the one which is being pointed to.
* Change `BENCH_NETWORK` variable from `erpnext-one` to the one which was created with the bench.

env file is generated at location mentioned in command.

**Generate yaml to reverse proxy:**
```
docker compose --project-name custom-one-example \
  --env-file ~/gitops/custom-one-example.env \
  -f overrides/compose.custom-domain.yaml \
  -f overrides/compose.custom-domain-ssl.yaml config > ~/gitops/custom-one-example.yaml
```

For LAN setup do not override `compose.custom-domain-ssl.yaml`.

**Deploy `erpnext-two` containers:**
```
docker compose --project-name custom-one-example -f ~/gitops/custom-one-example.yaml up -d
```

Refer This [Site Operations](https://github.com/frappe/frappe_docker/blob/main/docs/site-operations.md)