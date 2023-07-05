# Summary
In this section, we are going to see how to setup,  the `production` setup `Without Docker`

*Note: This setup for `ERPNext-14` you can use this for `ERPNext-13` also*

# Reference Documentation
* [ERP Production with docker but treafik no need](https://discuss.frappe.io/t/erp-production-with-docker-but-treafik-no-need/106371/1)

# Get Started
### Step 1
**Clone frappe_docker**
```
git clone https://github.com/frappe/frappe_docker.git
cd frappe_docker
```
**Custom App**

**List Of Core Apps**

* [frappe](https://github.com/frappe/frappe/tree/version-14)
* [erpnext](https://github.com/frappe/erpnext/tree/version-14)
* [hrms](https://github.com/frappe/hrms/tree/version-14)
* [payments](https://github.com/frappe/payments/tree/version-14)
* [india-compliance](https://github.com/resilient-tech/india-compliance/tree/version-14)

If you have the `Custom App` add below

`frappe` app is automatically added when you install the `bench` no need to mention below `APPS_JSON`

```
export APPS_JSON='[
  {
    "url": "https://github.com/frappe/erpnext",
    "branch": "version-14"
  },
  {
    "url": "https://github.com/frappe/hrms",
    "branch": "version-14"
  },
  {
    "url": "https://github.com/frappe/payments",
    "branch": "version-14"
  },
  {
    "url": "https://github.com/resilient-tech/india-compliance",
    "branch": "version-14"
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

**Update compose.yaml**
```
x-customizable-image: &customizable_image
  image: customapp:1.0.0
  pull_policy: never
```
**Create Custom Image**
```
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=v14.40.1 \
  --build-arg=PYTHON_VERSION=3.10.12 \
  --build-arg=NODE_VERSION=16.16.0 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=customapp:1.0.0 \
  --file=images/custom/Containerfile .
```


### Step 2:
Start the `mariadb`

DB Password: `admin`
```
mkdir ~/gitops
```

```
echo "DB_PASSWORD=admin" > ~/gitops/mariadb.env
```

**Deploy the mariadb container**
```
docker compose --project-name mariadb --env-file ~/gitops/mariadb.env -f overrides/compose.mariadb-shared.yaml up -d
```

### Step 3:
**Create First Bench**

```
cp example.env ~/gitops/erpnext-one.env
sed -i 's/DB_PASSWORD=123/DB_PASSWORD=admin/g' ~/gitops/erpnext-one.env
sed -i 's/DB_HOST=/DB_HOST=mariadb-database/g' ~/gitops/erpnext-one.env
sed -i 's/DB_PORT=/DB_PORT=3306/g' ~/gitops/erpnext-one.env
sed -i 's/SITES=`erp.example.com`/SITES=\`ziptor.com\`/g' ~/gitops/erpnext-one.env
echo 'ROUTER=erpnext-one' >> ~/gitops/erpnext-one.env
echo "BENCH_NETWORK=erpnext-one" >> ~/gitops/erpnext-one.env
```
**Create erpnext-one.yaml**
```
docker compose --project-name erpnext-one \
  --env-file ~/gitops/erpnext-one.env \
  -f compose.yaml \
  -f overrides/compose.redis.yaml \
  -f overrides/compose.multi-bench.yaml \
  -f overrides/compose.multi-bench-ssl.yaml config > ~/gitops/erpnext-one.yaml
```

**Deploy erpnext-one containers:**
```
docker compose --project-name erpnext-one -f ~/gitops/erpnext-one.yaml up -d
```

**Create Site**

DB Password: `admin`

Site Password: `admin`

*Note: this command only install `frappe` and `erpnext` after that you have to inspect the container and install the apps like this inspecting container `docker exec -it <BACKEND_CONTAINER_NAME> bash` after that run this command `bench use ziptor.com` after that `bench install-app <OTHER_APPS>`*
```
docker compose --project-name erpnext-one exec backend \
  bench new-site ziptor.com --no-mariadb-socket --mariadb-root-password admin --install-app erpnext --admin-password admin
```

`OR`

If you want to install in `single shot` try this
```
docker compose --project-name erpnext-one exec backend \
  bench new-site ziptor.com \
  --no-mariadb-socket \
  --mariadb-root-password admin \
  --install-app erpnext,hrms,payments,indian_complianc \
  --admin-password admin

```
