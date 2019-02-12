# ark-ansible

This is a set of Ansible roles which allow you to:
* Download Ark and OpenShift Installer
* Create an install-config.yaml file from a set of defaults
* Launch an OCP4.0 cluster to AWS
* Install Ark on the OCP cluster
* Create Nginx test application
* Back up Nginx application using Ark
* Restore Nginx application using Ark

# Prerequisites

To get started, ensure that you have signed up for an account at
`try.openshift.com`. This will allow you to download a Pull Secret which we
expect to be present in the config directory `config/`.

1. Download the Pull secret from https://try.openshift.com (assume `~/Downloads`)
1. `cp ~/Downloads/pull-secret config/`
1. Ensure you have set the following ENVIRONMENT VARIABLES
    ```
      export AWS_ACCESS_KEY_ID=XXXX
      export AWS_SECRET_ACCESS_KEY=XXXX
      export AWS_DEFAULT_REGION=us-east-2
    ```
   * The Ansible tasks will write the credentials to `~/.aws/credentials` if they do not already exist.
1. Optional:  Update the public SSH key if not using: `~/.ssh/libra_rsa.pub`
    * If you want to use a different public ssh key, edit 'sshKey' in config/defaults.yml
    ```yaml
    sshKey: "{{ lookup('file', '~/.ssh/libra_rsa.pub')  }}"
    ```



## Launch OCP Cluster

To launch an OCP cluster, run:
```
$ ansible-playbook launch-ocp-cluster.yml --extra-vars="@config/defaults.yml"
```

This will take a long time... potentially 30-45 minutes.

## Installing Ark

To launch an Ark Server onto OCP, run:
```
$ ansible-playbook launch-ark.yml
```

Don't worry about logging into the cluster, as long as the previous playbook
was successful Ansible will read the Kubeconfig and kubeadmin password each
time.

### Destroying Ark

If you would like to start with a fresh installation of Ark, you can delete all
the Ark resources with:
```
$ ansible-playbook destroy-ark.yml
```

## Visit Minio

To confirm everything was installed propery, you can visit the created route
and login to the minio instance. To get the route:
```
$ export KUBECONFIG=<ark-ansible-directory>/auth/kubeconfig
$ oc get route -n heptio-ark
[dymurray@pups ark-ansible]$ oc get route
NAME      HOST/PORT                                   PATH      SERVICES   PORT      TERMINATION   WILDCARD
minio     minio-heptio-ark.apps.dtm.mg.dog8code.com             minio      9000                    None
```

Minio's default credentials are:
```
Access Key: minio
Secret Key: minio123
```

## Create an Nginx Application and back it up

As a test application, you can run a playbook that creates Nginx with a route
and creates an Ark backup:

```
$ ansible-playbook create-nginx.yml
```

## Simulate DR scenario and restore Nginx from backup

First, delete the project that nginx exists in:
```
$ oc delete project nginx-example
```

Run a restore:
```
$ ansible-playbook delete-and-restore-nginx.yml
```

Verify the nginx route exists:
```
$ oc get route -n nginx-example
```


## Destroying an OCP Cluster

If you wish to start with a fresh cluster, I recommend destroying the old
instance. I have not had much luck launching multiple instances in the same
directory unless you are certain you know what you are doing.

To destroy the instance:
```
$ ./openshift-install destroy cluster
```

# Tips/Known Issues

## MacOS Requires 'gnu tar' for Ansible's unarchive module
  * If running on MacOS, ensure you have 'gnu tar' installed and set as default 'tar' from your PATH
  * Example error
```
TASK [install_ark : get ark] ***************************************************************************************************************************************

fatal: [localhost]: FAILED! => {"changed": false, "msg": "Failed to find handler for \"/Users/jmatthews/.ansible/tmp/ansible-tmp-1549911393.33-241542192915430/ark-v0.10.1-linux-amd64.tar.gz\". Make sure the required command to extract the file is installed. Command \"/usr/bin/tar\" detected as tar type bsd. GNU tar required. Command \"/usr/bin/unzip\" could not handle archive."}
```
  * Solution is to install 'gnu tar'

```
# If you see the below (which is the default in MacOS) you will need to install gnu tar
$ /usr/bin/tar --version
bsdtar 2.8.3 - libarchive 2.8.3

# To install gnu tar
brew install gnu-tar

# Then add the below to your ~/.bashrc towards the end
PATH="/usr/local/opt/gnu-tar/libexec/gnubin:$PATH"

# Then source your ~/.bashrc to pick up the change
source ~/.bashrc

# Now confirm that when you run tar you are running the gnu version
$ tar --version
tar (GNU tar) 1.31
Copyright (C) 2019 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>.
...

```
