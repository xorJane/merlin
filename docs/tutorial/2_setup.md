# Installation

!!! info "Prerequisites"

    - A desktop or laptop computer
    - shell (bash, csh, etc; if running on Windows, use [Git Bash](https://gitforwindows.org/) (part of "git for Windows"))

!!! info "Estimated Time"

    20 minutes

!!! abstract "You Will Learn"

    - How to build a Virtual Machine (VM) from a Vagrantfile
    - How to activate a Python virtual environment
    - How to configure Merlin.
    - How to test/verify the installation.

This section details the steps necessary to get build a Virtual Machine (VM) where Merlin and its dependencies are installed. Merlin will then be configured for the VM (a Linux environment) and the configuration will be checked to ensure a proper installation.


## Installing VirtualBox & Vagrant

First, install VirtualBox using the binary at https://www.virtualbox.org/wiki/Downloads best suited for your system.

Second, install [Vagrant](https://developer.hashicorp.com/vagrant/install).

You can run `which vagrant` to check that vagrant has properly installed and is now part of your path.

## Building your VM

Create a directory from which to run Vagrant. For example,

```bash
mkdir merlin-vm
```
Download this [Vagrantfile](Vagrantfile) to your new directory. This file must have the name `Vagrantfile`, with no extension such as `.txt`.

Enter the directory that you've created and run `vagrant up`:

```bash
cd merlin-vm
vagrant up
```

This will provision a virtual machine, install dependencies for python and singularity, install singularity, create a python virtual environment, and install merlin. 

Once `vagrant up` has finished running and the command prompt has returned to you, enter the VM via `vagrant ssh` and move to the `/vagrant` directory; this directory is the same as the directory you created on your local system, so data in `/vagrant` will be visible outside the VM. If the VM build was successful, you should see the path for singularity when you run `which singularity`. You should also have a directory called `merlin_venv` inside `/vagrant`:

```bash
vagrant ssh
cd /vagrant
which singularity
ls
```

Note that you can exit the virtual machine with `exit` and re-enter with `vagrant ssh`. `vagrant halt` will cause the VM to stop running. After running `vagrant halt`, you can turn the machine back on with `vagrant up`, but the installations and setup done previously will remain completely. `vagrant destroy` will remove the VM from your machine entirely, such that you will need to re-provision the machine (re-build the VM and all software) with `vagrant up` in order to use it again.

## Activating your Python virtual environment

From within the VM and `/vagrant` directory, you can activate your virtual environment via

```bash
source merlin_venv/bin/activate
```

This will add `(merlin)` before the VM's command prompt. Check that merlin has been installed by verifying you see its path after

```bash
which merlin
```

This virtual environment and merlin were built for you, but you could create a virtual environment with merlin yourself via

```bash
python3 -m venv --prompt merlin <new directory>
source <new directory>/bin/activate
pip install merlin
```

## Configuring Merlin

Merlin requires a configuration script for the Celery interface in order to know which server(s) to connect to. Run this configuration method to create the `app.yaml` configuration file.

```bash
merlin config --broker redis
```

The `merlin config` command above will create a file called `app.yaml` in the `~/.merlin` directory. If you are running a Redis server locally then you are all set, look in the `~/.merlin/app.yaml` file to see the configuration, it should look like the configuration below.

???+ abstract "app.yaml"

    ```yaml
    broker:
      name: redis
      server: localhost
      port: 6379
      db_num: 0

    results_backend:
      name: redis
      server: localhost
      port: 6379
      db_num: 0
    ```

More detailed information on configuring Merlin can be found in the [Configuration](../user_guide/configuration/index.md) page.

## Checking/Verifying Installation

First launch the Merlin server containers by using the `merlin server` commands.

Initialize the server files:

```bash
merlin server init
```

This will create a `merlin_server/` folder in the current run directory. The structure of this folder will look like so:

```bash
merlin_server/
|-- redis.conf
|-- redis.pass
|-- redis.users
`-- redis_latest.sif
```

The files in this folder are:

1. `redis.conf`: The Redis configuration file that contains all of the settings to be used for our Redis server
2. `redis.pass`: A password for the Redis server that we'll start up next
3. `redis.users`: A file defining the users that are allowed to access the Redis server and their permissions
4. `redis_latest.sif`: A singularity file that contains the latest Redis docker image that was pulled behind the scenes by Merlin

If you'd like to modify the configuration of your server, you can either modify the files directly or use:

```bash
merlin server config
```

Now that we have the necessary server files initialized, start the server:

```bash
merlin server start
```

With this command, the containerized server should now be started. Notice that two new files were added to the `merlin_server` folder:

1. `merlin_server.pf`: A process file containing information regarding the Redis process
2. `app.yaml`: A new `app.yaml` file configured specifically for the containerized Redis server that we just started

To have Merlin read this configuration, you could copy it to your current run directory:

```bash
cp merlin_server/app.yaml .
```

Alternatively, you could make this server container your main server configuration by replacing the one located in your home directory. Make sure you make back-ups of your current `app.yaml` file in case you want to use your previous configurations.

```bash
mv ~/.merlin/app.yaml ~/.merlin/app.yaml.bak
```

```bash
cp ./merlin_server/app.yaml ~/.merlin/
```

This will allow merlin to recognize and read the configuration originally in `merlin_server/app.yaml` even after you change directories. 

!!! note

    Since Merlin servers are created locally on your run directory you are allowed to create multiple instances of Merlin server with their unique configurations for different studies. Simply create different directories for each study and run the following command in each directory to create an instance for each:

    ```bash
    merlin server init
    ```

The `merlin info` command will check that the configuration file is installed correctly, display the server configuration strings, and check server access.

```bash
merlin info
```

If everything is set up correctly, you should see:

???+ success "Expected Output for Successful Config"

    ```
           *      
       *~~~~~                                       
      *~~*~~~*      __  __           _ _       
     /   ~~~~~     |  \/  |         | (_)      
         ~~~~~     | \  / | ___ _ __| |_ _ __  
        ~~~~~*     | |\/| |/ _ \ '__| | | '_ \ 
       *~~~~~~~    | |  | |  __/ |  | | | | | |
      ~~~~~~~~~~   |_|  |_|\___|_|  |_|_|_| |_|
     *~~~~~~~~~~~                                    
       ~~~*~~~*    Machine Learning for HPC Workflows                                 
              


    Merlin Configuration
    -------------------------

    config_file        | /path/to/app.yaml
    is_debug           | False
    merlin_home        | /path/to/.merlin
    merlin_home_exists | True
    broker server      | redis://default:******@127.0.0.1:6379/0
    broker ssl         | False
    results server     | redis://default:******@127.0.0.1:6379/0
    results ssl        | False

    Checking server connections:
    ----------------------------
    broker server connection: OK
    results server connection: OK

    Python Configuration
    -------------------------

    $ which python3
    /path/to/python3

    $ python3 --version
    Python x.y.z

    $ which pip3
    /path/to/pip3

    $ pip3 --version
    pip x.y.x from /path/to/pip (python x.y)

    "echo $PYTHONPATH"
    ```