After creation of the Google CloudSQL resources, the following is required:
* Create user "synapse"
* Obtain service account JSON for the CloudSQL Client Reader role for the instance.
* Encrypt the service account JSON and the synapse database password with ansible-vault.
* Configure your variables in `inventory/host_vars/<your-matrix-domain>/vars.yml:
  - matrix_postgres_enabled: false # this is so we use an external postgres instance
  - matrix_cloudsql_proxy_project: "test-installations-222013"
  - matrix_cloudsql_proxy_region: "europe-west1"
  - matrix_synapse_database_host: "172.17.0.1"
  - matrix_cloudsql_proxy_ip: "172.17.0.1:5432:5432"
  - matrix_cloudsql_instance_name: "your-db-name"
  - matrix_synapse_database_user: synapse
  - matrix_synapse_database_password: !vault |
  - matrix_cloudsql_service_account: !vault |
  - matrix_synapse_database_database: synapse

Log into database in the GCP Cloud Shell as the 'postgres' user and run the following commands:

```
GRANT "synapse" TO postgres;
CREATE DATABASE "synapse" WITH OWNER "synapse" ENCODING 'UTF8' LC_COLLATE = 'C' LC_CTYPE = 'C' TEMPLATE template0;
```

Run ansible role: 
`ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start --ask-vault-pass`

Wait about a minute for things to spin up before running the self-check.

Check to see if everything is healthy:
`ansible-playbook -i inventory/hosts setup.yml --tags=self-check --verbose`

Log into VM and ensure there are database connections:
`journalctl -u cloudsql-proxy`

Add your first user:
`ansible-playbook -i inventory/hosts setup.yml --extra-vars='username=r00t password=changeme admin=yes' --tags=register-user`
