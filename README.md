# Automatic Failover with repmgr for PostgreSQL 🚀

This guide explains how to set up automatic failover for PostgreSQL using **repmgr**. The steps ensure high availability of your PostgreSQL cluster by promoting a standby server to primary if the primary server fails.

---

## Prerequisites ✅

- Install **PostgreSQL** on both **Master** and **Standby** servers.
- Install **repmgr** matching the PostgreSQL version on both servers.
- Ensure that **network connections** between the servers are properly configured for replication.

---

## Step-by-Step Guide 🛠️

### Step 1: Install PostgreSQL on both servers 🐘

1. Install the PostgreSQL YUM repository:
    ```bash
    yum -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
    ```

2. Install PostgreSQL 12 and related packages:
    ```bash
    yum -y install postgresql12-server postgresql12
    ```

3. Initialize PostgreSQL on the **Master** server:
    ```bash
    /usr/pgsql-12/bin/postgresql-12-setup initdb
    ```

4. Enable and start PostgreSQL:
    ```bash
    systemctl enable --now postgresql-12
    systemctl status postgresql-12
    ```

**Note**: The **Standby** server does not require initialization.

---

### Step 2: Install repmgr on both servers ⚙️

1. Install repmgr on **Master** and **Standby** servers:
    ```bash
    yum -y install repmgr12*
    ```

---

### Step 3: Configure PostgreSQL on the Master Server 🛠️

Modify the `postgresql.conf` file to enable replication:

1. Set replication parameters:
    ```bash
    max_wal_senders = 10
    max_replication_slots = 10
    wal_level = 'hot_standby' # or 'replica' or 'logical'
    hot_standby = on
    archive_mode = on
    archive_command = '/bin/true'
    shared_preload_libraries = 'repmgr'
    ```

---

### Step 4: Create Users 👨‍💻

1. On the **Master** server, create the **repmgr** user and a metadata database:
    ```sql
    create user repmgr;
    create database repmgr with owner repmgr;
    ```

---

### Step 5: Configure `pg_hba.conf` for Replication 🔒

1. Grant replication permissions to the **repmgr** user by editing the `pg_hba.conf` file:
    ```bash
    local      replication      repmgr    trust
    host       replication      repmgr    127.0.0.1/32    trust
    host       replication      repmgr    192.168.1.0/24  trust
    local       repmgr           repmgr    trust
    host        repmgr           repmgr    127.0.0.1/32    trust
    host        repmgr           repmgr    192.168.1.0/24  trust
    ```

---

### Step 6: Configure `repmgr.conf` on the Master 📑

1. Create the `repmgr.conf` file on the **Master** server with the following settings:
    ```bash
    cluster='failovertest'
    node_id=1
    node_name=node1
    conninfo='host=node1 user=repmgr dbname=repmgr connect_timeout=2'
    data_directory='/var/lib/pgsql/12/data/'
    failover=automatic
    promote_command='/usr/pgsql-12/bin/repmgr standby promote -f /var/lib/pgsql/repmgr.conf --log-to-file'
    follow_command='/usr/pgsql-12/bin/repmgr standby follow -f /var/lib/pgsql/repmgr.conf --log-to-file --upstream-node-id=%n'
    ```

---

### Step 7: Register the Primary Server with repmgr 📝

1. Register the **Master** server:
    ```bash
    /usr/pgsql-12/bin/repmgr -f /var/lib/pgsql/repmgr.conf primary register
    ```

2. Check the cluster status:
    ```bash
    /usr/pgsql-12/bin/repmgr -f /var/lib/pgsql/repmgr.conf cluster show
    ```

---

### Step 8: Clone the Standby Server 🏗️

1. On the **Standby** server, create the `repmgr.conf` file.
2. Verify the configuration with a dry run:
    ```bash
    /usr/pgsql-12/bin/repmgr -h 172.16.140.135 -U repmgr -d repmgr -f /var/lib/pgsql/repmgr.conf standby clone --dry-run
    ```

3. Clone the **Standby** server:
    ```bash
    /usr/pgsql-12/bin/repmgr -h 172.16.140.135 -U repmgr -d repmgr -f /var/lib/pgsql/repmgr.conf standby clone
    ```

---

### Step 9: Register the Standby Server with repmgr 📝

1. Register the **Standby** server:
    ```bash
    /usr/pgsql-12/bin/repmgr -f /var/lib/pgsql/repmgr.conf standby register
    ```

---

### Step 10: Start the repmgrd Daemon ⚡

1. Start the **repmgrd** daemon on both **Master** and **Standby** servers:
    ```bash
    /usr/pgsql-12/bin/repmgrd -f /var/lib/pgsql/repmgr.conf
    ```

---

### Step 11: Monitor Cluster Events 🔍

1. Check cluster events to verify the automatic failover:
    ```bash
    /usr/pgsql-12/bin/repmgr -f /var/lib/pgsql/repmgr.conf cluster event
    ```

---

## Conclusion 🎉

By following these steps, you’ve configured **automatic failover** with **repmgr** for PostgreSQL. The **repmgrd** daemon will monitor the cluster and, in case of a failure of the primary server, will automatically promote the **Standby** to be the new primary server, ensuring high availability for your PostgreSQL database.

---

For any questions or issues, feel free to reach out to our support team! 👨‍💻

