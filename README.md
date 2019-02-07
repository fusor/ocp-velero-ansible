# ark-ansible

This is a set of Ansible roles which allow you to:
* Download Ark and OpenShift Installer
* Create an install-config.yaml file from a set of defaults
* Launch an OCP4.0 cluster to AWS
* Install Ark on the OCP cluster

# Prerequisites

To get started, ensure that you have signed up for an account at
`try.openshift.com`. This will allow you to download a Pull Secret which we
expect to be present in the config directory `config/`.

1. Download the Pull secret from https://try.openshift.com (assume `~/Downloads`)
1. `cp ~/Downloads/pull-secret config/`

Modify the values in `config/defaults.yml` with the proper AWS credentials and
SSH Key to use for the install:
```yaml
aws_access_key_id: ******
aws_secret_access_key: *******
pullSecret: "{{ lookup('file', '{{ playbook_dir }}//config/pull-secret') | from_json }}" # Do not modify this line
sshKey: "ssh-rsa ************************"
baseDomain: mg.dog8code.com
masterReplicas: 1
workerReplicas: 1
awsRegion: us-east-2
```

The Ansible tasks will write the credentials to `~/.aws/credentials` if they do
not already exist. You may optionally leave these vars empty and we will
attempt to read the credentials from `~/.aws/credentials`.

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
$ ansible-playbook create-ark.yml
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
