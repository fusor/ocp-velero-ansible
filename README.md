# ark-ansible

This is a set of Ansible roles which allow you to:
* Download Ark and OpenShift Installer
* Create an install-config.yaml file from a set of defaults
* Launch an OCP4.0 cluster to AWS
* Install Ark on the OCP cluster

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
```

The Ansible tasks will write the credentials to `~/.aws/credentials` if they do
not already exist. You may optionally leave these vars empty and we will
attempt to read the credentials from `~/.aws/credentials`.

