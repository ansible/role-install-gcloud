Install gcloud 
==============

Use to install the Google Cloud SDK and its components, including Kubernetes' kubectl CLI. If you're deploying
things to Kubernetes, this role is for you, as it will automate the install of the CLI on your development and 
deployment hosts.

Determine which Archive to Install
----------------------------------

The name of the archive to install must be determined prior to running this role. 
Unfortunately there is no automated way to determine the archive name. You cannot install 
the SDK via the typical package managers. Instead, go to [the SDK web site](https://cloud.google.com/sdk/) 
and find the archive that fits your system. Once you have the archive name, plug it 
into this role. 

Installing the Archive
----------------------

This role can download the archive onto the target host(s) and install it **OR** you can have 
the archive already downloaded and available on the Ansible host, in which case the role will
copy the archive to the target host(s) and install it. To download and install, provide 
*gcloud_archive_name*. To copy and install, provide *gcloud_archive_path*. 


Where are components installed?
-------------------------------

The Google installer does not attempt to link or copy executables to /usr/local/bin or a 
bin directory that is typically part of the environment PATH. Instead the component 
executables live in {{ gcloud_install_path }}/bin. If you want to add links or copy files to a bin 
directory, add additional tasks to your playbook post role execution. 

The install process will update a login script to add {{ gcloud_install_path }}/bin to the 
environment PATH. If you want this behavior, leave *gcloud_update_path* to true. The installer 
will attempt to update the default login script for the user. To update a specific script, set the
value of *gcloud_profile_path*.

What components are available?
------------------------------
Use the Component ID value in *gcloud_additional_components* and *glcoud_override_components* to
control which components will be installed.

Component Name | Component ID | Size
--- | --- | ---:
Cloud Datastore Emulator | cloud-datastore-emulator | 15.9 MiB
Cloud Datastore Emulator (Legacy) | gcd-emulator | 38.1 MiB
Cloud Pub/Sub Emulator | pubsub-emulator | 10.8 MiB
gcloud Alpha Commands | alpha | < 1 MiB
gcloud Beta Commands | beta | < 1 MiB
gcloud app Java Extensions | app-engine-java | 131.0 MiB
gcloud app PHP Extensions (Mac OS X) | app-engine-php-darwin | 21.9 MiB
gcloud app Python Extensions | app-engine-python | 7.2 MiB
BigQuery Command Line Tool | bq | < 1 MiB
Cloud SDK Core Libraries | core | 4.1 MiB
Cloud Storage Command Line Tool | gsutil | 2.6 MiB
Default set of gcloud commands | gcloud |  
kubectl | kubectl | 8.1 MiB

Requirements
------------

The target host requires:

- tar 

And of course, if you intend to download the archive, the target host will need access to the 
the outside world.


Role Variables
--------------
gcloud_archive_name
> Defaults to ''. If set, the archive will be downloaded from Google onto the target host. Set to the name of 
> an archive file. Example: google-cloud-sdk-114.0.0-darwin-x86_64.tar.gz. 
> Visit [the SDK site](https://cloud.google.com/sdk/) to find the archive name for your system. 

gcloud_archive_path
> Defaults to ''. Path to the gcloud archive file on the Ansible host. If defined, the archive will be copied to the target host. 

gcloud_tmp_path
> Defaults to /tmp. Set to the path where the archive can be temporarily placed.

gcloud_force_download
> Default to yes. When downloading the archive, always perform the download, even if the archive already exists in {{ glcloud_tmp_path }}.

gcloud_install_path
> Defaults to "{{ ansible_env.HOME }}". Path on target host to place the unarchived files.

gcloud_usage_reporting
> Defaults to no. Enable usage reporting?

gcloud_profile_path
> Defaults to '~/.profile'. Path to the login script to be updated by the installer.

gcloud_command_completion
> Defaults to yes. Enable bash style command completion in the login script?

gcloud_update_path
> Defaults to yes. Update the PATH environment varialbe within the login script?

gcloud_override_components
> Defaults to []. Override the components to be installed, and install these instead. 

gcloud_additional_components
> Defaults to [kubectl]. Additional components to install. Will either be added to the default install list, or to the override-components list (if provided). 

Example Playbook
----------------

Here's an example playbook that executes our role:

    - name: Create temp intall path 
      hosts: localhost
      connection: local
      gather_facts: no
      tasks:
        - name: Create a tmp path
          file:
            path: "/tmp/install_gcloud"
            state: directory
            mode: 0777
    
    - name: Run the install
      hosts: localhost
      connection: local
      gather_facts: yes    # Run gather_facts to define ansible_env.HOME
      roles:
        - role: role-install-gcloud
          gcloud_tmp_path: /tmp/install_gcloud 
          gcloud_archive_name: google-cloud-sdk-114.0.0-darwin-x86_64.tar.gz

    - name: Remove temp install path
      hosts: localhost
      connection: local
      gather_facts: no
      tasks:
        - name: Remove the tmp path
          file:
            path: "/tmp/install_gcloud"
            state: absent

License
-------

MIT

Authors
-------

[@chouseknecht](https://github.com/chouseknecht)

