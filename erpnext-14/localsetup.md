# ERPNext-14 Local Setup or Development Setup

For Reference Documentation [CLICK HERE](https://github.com/frappe/frappe_docker/blob/main/docs/development.md)

We are assuming You have `Custom App` using databas as a `mariadb`

### Step 1
```
git clone https://github.com/frappe/frappe_docker.git
cd frappe_docker
```

### Step 2
```
cp -R devcontainer-example .devcontainer
```
```
cp -R development/vscode-example development/.vscode
```

### Step 3
```
code --install-extension ms-vscode-remote.remote-containers
```
```
code .
```

### Step 4
```
nvm use v16
```
```
PYENV_VERSION=3.10.5 bench init --skip-redis-config-generation --frappe-branch version-14 frappe-bench
```
```
cd frappe-bench
```

### Step 5
```
bench set-config -g db_host mariadb
bench set-config -g redis_cache redis://redis-cache:6379
bench set-config -g redis_queue redis://redis-queue:6379
bench set-config -g redis_socketio redis://redis-socketio:6379
```

### Step 6
```
bench new-site mysite.localhost --mariadb-root-password 123 --admin-password admin --no-mariadb-socket
```

### step 7
```
bench --site mysite.localhost set-config developer_mode 1
bench --site mysite.localhost clear-cache
```

### Step 8
Install The `erpnext-14`
```
bench get-app --branch version-14 --resolve-deps erpnext
bench --site mysite.localhost install-app erpnext
```

### Step 9
Install `hrms` moduele
```
bench get-app --branch version-14 hrms
bench --site mysite.localhost install-app hrms
```

### Step 10
Install `Payments` module
```
bench get-app --branch version-14 payments
bench --site ss-erp14.localhost install-app payments
```

### Step 11
If you want you can install this module it's `non_profit` module.
Better you can avoid if your not using it. 
```
bench get-app non_profit
bench --site mysite.com install-app non_profit
```

### Step 12
Install Your `Custom App`
```
bench get-app --branch version-12 https://github.com/myusername/myapp
bench --site mysite.localhost install-app myapp
```
*Note: In your case the custom app branch will vary so please change the respective branch*

### Start Project
```
bench start
```