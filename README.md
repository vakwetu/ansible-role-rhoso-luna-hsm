# rhoso-luna-hsm Role

In order to use HSMs, the barbican images need to be customized to include the HSM software.

The purpose of this role is to:
* Generate new images for the barbican-api and barbican-worker containing the HSM software
* Upload those images to a private repository for use in setting up a CI job.
* Create any required config to be mounted by the barbican images to interact with the HSM

For the Lunasa, we expect some preparatory steps to be completed prior to execution in order for the
role to complete successfully.
* The lunasa software is uploaded somewhere and will be fetched by the role
  * The contents of the minimal linux client in a zipped tar file should be made available at luna_minclient_src.
  * The lunasa binaries that need to be added to the image are made available at luna_binaries_src.
  * The lunasa HSM cacert file is made available at luna_server_cert_src.  For an HA configuration,
    this will be a concatenation of all the server certs for the servers in the HA partition.
  * The client certificate and key made available at luna_client_cert_src.  The files are expected
    to be of the form "(client_ip)".pem and "(client_ip)"Key.pem
* The certs will be retrieved and stored in a secret (luna_cert_secret)
* The password to log into the HSM partition will be stored in a secret (login_secret)

A minimal (one that takes the defaults) invocation of this role is shown below.  In this case, the lunaclient
software and certs are stored locally under /opt/luna.

- name: Set up Luna
  ansible.builtin.include_role: rhoso_luna_hsm
  vars:
    client_ip: "IP of the client - this could be the hypervisor where the Openshift nodes run"
    partition_password: "password to log into partition"
    kubeconfig_path: "path to kubeconfig file"
    oc_path: "path to oc binary"

You can also do the steps separately.

- name: Create new barbican image with the hsm software
  ansible.builtin.include_role: rhoso_luna_hsm
  tasks_from: create_image.yml

- name: Create secrets containing the certs and partition password
  ansible.builtin.include_role: rhoso_luna_hsm
  tasks_from: create_secrets.yml
  vars:
    client_ip: "IP of the client - this could be the hypervisor where the Openshift nodes run"
    partition_password: "password to log into partition"
    kubeconfig_path: "path to kubeconfig file"
    oc_path: "path to oc binary"


## Role Variables

### Role Parameters
* `cleanup`: (Boolean) Delete all resources created by the role at the end of the testing. Default value: `false`
* `working_dir`: (String) Working directory to store artifacts.  Default value: `/tmp/hsm-prep-working-dir`

### Image Generation Variables
* `barbican_src_image_registry`: (String) Registry of the source image. Default value: `quay.io`
* `barbican_src_image_namespace: (String) Namespace of the source image. Default value: `podified-antelope-centos9`
* `barbican_src_image_tag: (String) Tag of the source image. Default value: `current-podified`
* `barbican_dest_image_registry`: (String) Registry of the modified image. Default value: `quay.io`
* `barbican_dest_image_namespace: (String) Namespace of the modified image. Default value: `podified-antelope-centos9`
* `barbican_dest_image_tag: (String) Tag of the modified image. Default value: `current-podified-luna`
* `luna_minclient_src`: (String) Location of linux minimal client tarball. Default value: `file:///opt/luna/Linux-Minimal-Client.tar.gz`
* `luna_binaries_src`: (String) Location of the luna binaries. Default value: `file:///opt/luna/bin`

### Secret Generation Variables
* `luna_server_cert_src`: (String) Location of HSM server CA cert.  Default value: `file:///opt/luna/cert/server/cacert.pem`
* `luna_client_cert_src`: (String) Location of HSM client certs.  Default value: `file:///opt/luna/cert/client`
* `server_ca_file`: (String) Name of the cacert file in the container.  Default value: `cacert.pem`
* `client_ip`: (String) ip address or hostname of the client VM
* `luna_cert_secret`: (String) Name of the secret that stores all of the needed certs for luna.  Default value: `barbican-luna-certs`
* `luna_cert_secret_namespace`: (String) Namespace of the secret that stores all of the needed certs for luna.  Default value: `openstack`
* `login_secret`: (String) The secret to store the password to log into the HSM partition. Default: `hsm-login`
* `partition_password`: (String) Password to log into the HSM Partition
* `kubeconfig_path`: (String) Path to kubeconfig file
* `oc_path`: (String) Path to oc binary
