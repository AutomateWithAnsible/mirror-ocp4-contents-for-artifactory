Role Name
=========

A utility role to mirror OpenShift Container Platform 4.x (OCP4) content for disconnected install.
This only covers the Artifactory use case as this registry does not seem to follow the prescribed API resource layout for container registry v2. 
For instance the bundle generated by Code Sparta  (https://github.com/CodeSparta/openshift) does not work and we cannot use the oc adm release mirror command but the oc image mirror works.  
This will contains various tasks to mirror all types of contents (images and manifests, operators and manifests). 
This was originally developed to support mirroring contents to Artifactory but it can also be used to mirror and push to any docker registry v2.2 compliant registry.

Requirements
------------

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

Role Variables
--------------

For the bundle tasks:
- ansible_name_module: The name of the module this role is part of. This is used to track context when this role is run in a playbook as part of other plays.
- dir_bundle_location: Location of the bundle on the host the role/playbook is run against.
- bundle_file_name: Name of the xz format compressed artifact bundle file (e.g. artifactory-bundle.tar.xz)
- ocp_registry_pull_secret_file: Json formatted pull secret to access the required registries .
- ocp_release_image_registry: FQDN of the source registry content will be mirrored from (e.g. redhat.io)
- ocp_release_image_repository: Name of the repository being mirrored (e.g. openshift-release-dev/ocp-release)
- ocp_release_version: Release version being mirrored (e.g. 4.6.4)
- ocp_release_arch: Release architecture for the images being mirrored (e.g. x86_64)

For the unbundle tasks:
- ansible_name_module: The name of the module this role is part of. This is used to track context when this role is run in a playbook as part of other plays.
- bundle_file_location: Fully qualified location of the xz format compressed bundle on the controller or host the role/playbook is run against.
- dir_bundle_staging: Name of the staging diretory the compressed bundle will be uncompressed in and used for the container registry if needed.
- registry_admin_username: Username to use to authenticate to the Artifactory destination registry.
- registry_admin_password: Password to use to authenticate to the Artifactory destination registry.
- registry_host_fqdn: FQDN of the Artofactory registry. Note that this should not have the port associated withthe registry.
- local_repository: Name of the local repository mirrored images are pushed into (e.g. openshift4)
- release_image_repository: Name of the sub repository mirrored images are pushed into (e.g. openshift-release-dev)
- unpack: To skip unpacking if already unpacked.
- reg_port: To be used if the registry is not artifactory and a port is needed.

For the operator bundle tasks( reusing some of the variable from the images bundle above to keep minimal list of variables):
- registry_container_name: The name of the container registry to use to mirror operators related contenti (e.g. mirror-registry).
- registry_container_dir: The name of the directory structure on the controller host that will be mounted into the registry container to save the container image layers and blobs.
- registry_container_image: The name of the registry container to run (e.g. docker.io/library/registry:2).
- operator_registres_to_mirror: The various operators that will be mirrored (e.g.redhat-operators, community-operators, redhat-certified-operators...). This is a dictionary with keys like source, container_port, and mirrored_operartor_list where :
  - source is the FQDN of the registry and repository and version of the registry index being used to mirror the operator content (e.g. registry.redhat.io/redhat/redhat-operator-index:v4.6)
  - container_port is the name of the port on the controller to bind the container to (e.g. 50051). It has to be different for each operator index container.
  - mirrored_operator_list is the list of operators to be mirrored if pruning operators to provided comma separated list (e.g. '3scale-operator,apicast-operator').
- opm_client_binary: The path to the opm client binary to be used to mirror operators if not already installed on the controller. 
- grpcurl_binary: The path to the grpcurl client binary to be used to prune operators if pruning a subset of operators. 
- install_grpcurl: The boolean used to toggle the grpcurl binary install if not already install on host.
- install_opm: The boolean used to toggle the opm binary install if not already install on host.

For the operator unbundle tasks( reusing some of the variable from the images unbundle above to keep minimal list of variables):


Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------
To create the bundle for Artifactory or any v2.2 registry use the sample play below:

    - hosts: localhost
      tasks:
         - name: Create bundle for artifactory
           import_role:
             name: mirror-ocp4-contents-for-artifactory
             tasks_from: bundle-ocp-images.yml
             
To push the bundle content into the Artifactory registry or any v2.2 registry use the sample play below:

    - hosts: localhost
      tasks:
         - name: Push bundle content into artifactory
           import_role:
             name: mirror-ocp4-contents-for-artifactory
             tasks_from: unbundle-ocp-images.yml
             
To create the operator bundle for Artifactory use the sample play below:

    - hosts: localhost
      tasks:
         - name: Create operator bundle for artifactory
           import_role:
             name: mirror-ocp4-contents-for-artifactory
             tasks_from: bundle-operators.yml
             
To push the operator bundle content into the Artifactory registry use the sample play below:

    - hosts: localhost
      tasks:
         - name: Push operator bundle content into artifactory
           import_role:
             name: mirror-ocp4-contents-for-artifactory
             tasks_from: unbundle-operators.yml
           vars:
             unpack: "true"
             
To push the operator bundle content into a v2.2 registry use the sample play below:

    - hosts: localhost
      tasks:
         - name: Push operator bundle content into artifactory
           import_role:
             name: mirror-ocp4-contents-for-artifactory
             tasks_from: unbundle-operators.yml
           vars:
             reg_port: "{{ registry_host_port }}"
             unpack: "true"

Note that the registry port is being set if the registry is not artifactory.
Also if the bundle was already unpacked you can set unpack to false

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
