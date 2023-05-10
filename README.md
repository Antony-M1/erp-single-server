# erp-single-server
erp13, ERPNext-13 Production Setup, erp production setup

This Documentation Complete about the setup the `ERPNext-13` in a single server, which means you can setup in the `AWS EC2` or `Your Local Server`(PC) follow all the setup carefully line by line

# Reference Documentations:
* [Single Server](https://github.com/frappe/frappe_docker/blob/main/docs/single-server-example.md)
* [Custom App](https://github.com/frappe/frappe_docker/blob/main/docs/custom-apps.md)

For Complete `Local` and `Production` setup [CLICK HERE](https://github.com/frappe/frappe_docker#custom-images)

# Single Server Production Setup
We are assuming you are using linux Ubuntu 16+

### STEP 1
Open the terminal in your server(PC) and run this command

**Run Command**
```
git clone https://github.com/frappe/frappe_docker
```
we are assuming you have domain with the name of `zapptor.com` or some other domain for testing purpose only im giving this domain name.

Change The directory name `frappe_docker` to some Other name.
For example if im hosting a site with a name `zapptor.com` change the folder name
`frappe_docker` to `zapptor`

Run this command to change the directory.

**Run Command**
```
sudo mv frappe_docker zapptor
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
echo 'TRAEFIK_DOMAIN=zapptor.com' > ~/gitops/traefik.env
echo 'EMAIL=admin@zapptor.com' >> ~/gitops/traefik.env
echo 'HASHED_PASSWORD='$(openssl passwd -apr1 changeit | sed 's/\$/\\\$/g') >> ~/gitops/traefik.env
```
Note:

* Change the domain from `zapptor.com` to the one used in production. DNS entry needs to point to the Server IP.
* Change the letsencrypt notification email from `admin@zapptor.com` to correct email.
* Change the password from `changeit` to more secure.
env file generated at location `~/gitops/traefik.env` will look like following:
```
TRAEFIK_DOMAIN=zapptor.com
EMAIL=admin@zapptor.com
HASHED_PASSWORD=\$apr1\$POT9B0su\$f9WtnniQOwnPR0fUALZec.
```

