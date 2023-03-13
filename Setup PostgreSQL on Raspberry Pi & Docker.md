# Installing a PostgreSQL server on a Raspberry Pi using Docker.

## Pre-requisites
What we need:
1) Raspberry Pi
2) Docker
3) SSH enabled on the Raspberry Pi

## Step 1) Install PostgreSQL on Docker

### 1.1) Go to Portainer and install the PostgreSQL container.
Inside the Portainer portal, you can find PostgreSQL as an app template ready to download. From there you can set your servers username & password.

## Step 2) Check if PostgreSQL is installed

### 2.1) Interact with the container
Run the following:

    $ docker exec -it postgres_rpi bash

This will execute a bash shell in the 'postgres_rpi' container (I named the container 'postgres_rpi). The `-it` flag allows for an interactive sesion so you can interact with the container in real time. This is useful for running commands and debugging inside the container.

### 2.2) Connect to the PostgreSQL database server
Run the following command:

    $ psql -h localhost -U admin

This command is used to connect to the PostgreSQL database server using the command-line interface interface utility called, `psql`.

It taskes in a few parameters:

-  `psql` : The command used to start the PostgreSQL client program
- `-h localhost` : The hostname or IP address of the PostgreSQL server to connect to. In this case, it is `localhost`, which means the server is running on the same machine as the client.
- `-U admin` : The username top use when connecting to the server. In this case, it is `admin`.

### 2.3) List out the databases
Just for a quick measure, list out the databases using the `\l` command.

    $ \l

You should see something like this:

    List of databases
    Name | Owner | Encoding | Collate | Ctype | Access privileges
    ------+-------+----------+---------+-------+-------------------
    postgres | admin | UTF8 | en_US.UTF-8 | en_US.UTF-8 |
    template0 | admin | UTF8 | en_US.UTF-8 | en_US.UTF-8 | =c/admin
    template1 | admin | UTF8 | en_US.UTF-8 | en_US.UTF-8 | =c/admin
    (3 rows)

## Step 3) Connect to the PostgreSQL database server from a GUI client
Now that we have the PostgreSQL server running on Docker, we can connect to it from a GUI client. In this case, we will be using JetBrains DataGrip.

Before, we actually connect to the server in DataGrip, we need to change some files so that certain IP addresses can connect to the server.

### 3.1) Change the `pg_hba.conf` file
The `pg_hba.conf` file is used to control how the server determines which hosts are allowed to connect to it. By default, the file only allows connections from the localhost. We need to change this so that we can connect to the server from a GUI client.

#### 3.1.1) Find the `pg_hba.conf` file

To find where this file is, will run this while still connected to the database as the `user: admin`. Run the following command:

    $ SHOW hba_file;

You should see something like this:
    
        hba_file
        ------------------------
        /var/lib/postgresql/data/pg_hba.conf
        (1 row)

#### 3.1.2) Edit the `pg_hba.conf` file
Now that we know where the file is, we can edit it. Run the following command *(you may or may not need sudo)*:

    $ sudo nano /var/lib/postgresql/data/pg_hba.conf

This will open the file in the nano text editor. We need to include the following line:

    # IPv4 local connections:
    host     all     all     192.168.1.0/24     trust

This will allow you to be able to connect from any machine on your home network under IPv4 local connections within your network address and the CIDR notation of your subnet mask.

#### 3.1.3) Edit the `postgresql.conf` file
Now that we have edited the `pg_hba.conf` file, we need to edit the `postgresql.conf` file. This file is used to control the server's operation. We need to change the `listen_addresses` to `*` so that the server will listen on all available IP addresses.

*Note: This file should be in the same location as `pg_hba.conf`.*

Run the following command *(you may or may not need sudo)*:

    $ sudo nano /var/lib/postgresql/data/postgresql.conf

This will open the file in the nano text editor.

When you are in the file, go down to the `CONNECTIONS AND AUTHENTICATION` section.

We need to change or include the following line if it does not exist:

    listen_addresses = '*'

While you are there, make sure these are also uncommented:
    
        port = 5432
        max_connections = 100

### 3.2) Restart the PostgreSQL server
Restart the server through Portainer.

### 3.3) Connect to the PostgreSQL server from DataGrip
Now that we have the server running, we can connect to it from DataGrip. In DataGrip, go to `File > New > Data Source > PostgreSQL`.

#### 3.3.1) General settings
In the `Data Source` window, fill out the following:

- `Name`: Whatever you want
- `Host`: IP address of the container (can be found in Portainer)
- `Port`: Your designated port (default is 5432)
- `Database`: Name of database
- `User`: [username]
- `Password`: [password]

#### 3.3.2) SSH/SSL settings
In the `SSH/SSL` window:

1. Click `Use SSH tunnel`
2. Configure your SSH connection
    - `Host`: IP address of the Raspberry Pi
    - `Port`: Your designated port (default is 22)
    - `User`: [username]
    - `Password`: [password] 

#### 3.3.3) Test the connection
Test the connection and see if it works.