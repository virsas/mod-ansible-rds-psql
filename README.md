# AWS RDS Postgres configuration

## Role installation

### create requirements file

```bash
cd ANSIBLE_FOLDER
mkdir requirements
touch requirements/rds_psql.yml
```

### file content

```yaml
---
# ansible-galaxy install -p vss_galaxy_roles --force -r requirements/rds_psql.yml
- src: "https://github.com/virsas/mod-ansible-rds-psql"
  scm: git
  version: v1.0.0
  name: rds_psql
  path: vss_galaxy_roles
```

## Install role

### Installation

```bash
ansible-galaxy install -p galaxy_roles --force -r requirements/rds_psql.yml
```

### Best practices

If you are using git for your playbooks and sites configuration, add vss_galaxy_roles to your .gitignore file. You are free to download the role directly to your roles directory, but it will be then pushed to your repo too.

## Role access

### Direct access

```bash
$ cd vss_galaxy_roles
$ ls
rds_psql
$ cd rds_psql
$ ls -1
defaults
LICENSE
meta
README.md
tasks
$
```

### Access from ansible (ansible.cfg)

```bash
[defaults]
gathering = smart
roles_path=./roles:../roles:../../roles:./vss_galaxy_roles
log_path=./ansible_run.log
retry_files_enabled=False
[ssh_connection]
scp_if_ssh=True
host_key_checking=False
```

## Files

### Playbook (./playbooks/rds_psql.yml)

```yaml
---
---
- hosts: db
  connection: local
  roles:
    - rds_psql
```

### Inventory (./sites/NAME/inventory)

```txt
[rds_psql]
rds_psql_01
```

### Host vars (./sites/NAME/host_vars/rds_psql_01.yml)

```yaml
---
DBHOST: dbname.cluster-id.eu-west-1.rds.amazonaws.com
DBPASS: "dbpassword"
DBUSER: "dbusername"

POSTGRES_CONFIGURATION:
  [
    {
      db: newdb,
      users:
        [
          {
            name: newuser,
            privileges:
              [
                {
                  type: table,
                  objects: ALL_IN_SCHEMA,
                  privileges: "ALL",
                  schema: "public",
                  grant_option: False,
                },
                {
                  type: sequence,
                  objects: ALL_IN_SCHEMA,
                  privileges: "USAGE,SELECT",
                  schema: "public",
                  grant_option: False,
                },
              ],
          },
        ],
    },
  ]
```

### Group vars (./sites/NAME/group_vars/rds_psql.yml)

Please see the variables below to get the complete list of possible modifications. The YAML file below is just a basic example of a minimal configuration.

```yaml
---
DBPORT: 5432
```

## Usage

```bash
ansible-playbook -i sites/NAME/inventory playbooks/rds_psql.yml --check --diff
```

## Variables

```yml
# PORT OF THE DATABASE
DBPORT: 5432
# HOST OF THE DATABASE
DBHOST: dbname.cluster-id.eu-west-1.rds.amazonaws.com
# PASSWORD FOR THE ADMIN USER
DBPASS: "password"
# ADMIN USERNAME
DBUSER: "user"

POSTGRES_CONFIGURATION: [
    {
      # name of the database you want to apply privileges to. Can be existing one or ansible will create a new one based on the name
      db: database,
      # encoding of the database, this variable is optional
      encoding: UTF-8,
      # list of users
      users: [
          {
            # name of the user
            name: user,
            # password for remote access, this can be omitted if you use the rds_iam group and iam authentication
            password: password,
            # group the user is attached to. In case of IAM authentication, this is how you would configure it.
            groups: [{ name: rds_iam, admin_option: False }],
            # individual privileges
            privileges: [
                {
                  # supported types table, partition table, sequence, function or procedure
                  type: table,
                  # ALL_IN_SCHEMA or list of objects the privileges should be applied to like a list of tables in case of table type.
                  objects: ALL_IN_SCHEMA,
                  # privileges, it can be comma separated list like USAGE,SELECT or ALL
                  privileges: "ALL",
                  # name of the schema
                  schema: "public",
                  # Wheather role may grant privileges to others
                  grant_option: False,
                },
                # another example to grant usage, select on all sequences in public schema to user
                {
                  type: sequence,
                  objects: ALL_IN_SCHEMA,
                  privileges: "USAGE,SELECT",
                  schema: "public",
                  grant_option: False,
                },
              ],
          },
        ],
    },
  ]
```
