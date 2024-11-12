# Red Hat One - RHEL Image mode session repo

This repo is a working asset to deliver and replicate the RHEL Image mode Session running at Red Hat One 2025.

## AAP Setup

The folder [aap-setup](./aap-setup/) contains all required bits to configure all templates, credentials and inventories as well as rulebook activations on EDA that are needed for the demo.

### demo-setup-vars.yml

This file contains all required variables to populate projects, credentials and templates:

``` yaml
main_repo_url: https://github.com/kubealex/rh1-image-mode.git
image_mode_application_repo: https://github.com/kubealex/rh1-image-mode-container-app.git
image_mode_soe_repo: https://github.com/kubealex/rh1-image-mode-soe

rhel_image_mode_registry_url: service-vm.rh-lab.labs:3000
rhel_image_mode_registry_username: gitea
rhel_image_mode_registry_password: redhat

aap2_controller_url: https://aap.rh-lab.labs:8443
aap2_controller_username: admin
aap2_controller_password: redhat

eda_controller_url: https://aap.rh-lab.labs:8445
eda_controller_username: admin
eda_controller_password: redhat
```

It is pre-filled with the demo setup fields, that can be changed according to your own environment.

Running the following command will end up creating all required resources:

    ansible-playbook configure-aap.yml


## Inventory

The inventory file contains two groups, *webserver* and *build_host* that are respectively the server providing access to the ISO and kickstart and the host that is needed to build the images.

    [build_host]
    rhel-bootc.rh-lab.labs ansible_user=sysadmin ansible_password=redhat

    [webserver]
    rhel-bootc.rh-lab.labs ansible_user=sysadmin ansible_password=redhat

