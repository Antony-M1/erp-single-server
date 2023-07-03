# Summary
In this section, we are going to see how to setup,  the `production` setup `Without Docker`

*Note: This setup for `ERPNext-14` you can use this for `ERPNext-13` also*

# Reference Documentation
* [ERP Production with docker but treafik no need](https://discuss.frappe.io/t/erp-production-with-docker-but-treafik-no-need/106371/1)

# Get Started
### Step 1
**Custom App**

**List Of Core Apps**

* [frappe](https://github.com/frappe/frappe/tree/version-14)
* [erpnext](https://github.com/frappe/erpnext/tree/version-14)
* [hrms](https://github.com/frappe/hrms/tree/version-14)
* [payments](https://github.com/frappe/payments/tree/version-14)
* [india-compliance](https://github.com/resilient-tech/india-compliance/tree/version-14)

If you have the `Custom App` add below

```
export APPS_JSON='[
  {
    "url": "https://github.com/frappe/erpnext",
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
