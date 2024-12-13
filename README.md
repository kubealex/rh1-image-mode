# Red Hat One - RHEL Image mode session repo

This repo is a working asset to deliver and replicate the RHEL Image mode Session running at Red Hat One 2025.

- [Demo Setup](#demo-setup)
   * [Prerequisites](#prerequisites)
   * [Lab deployment](#lab-deployment)
   * [demo-setup-vars.yml](#demo-setup-varsyml)
   * [Configure Gitea Instance](#configure-gitea-instance)
   * [Configure AAP](#configure-aap)
- [Demo use cases](#demo-use-cases)
   * [Creating a SOE image](#creating-a-soe-image)
   * [Creating the application ISO image](#creating-the-application-iso-image)
   * [Deploy our ISO on our device](#deploy-our-iso-on-our-device)
   * [Update the image with a new version of the Java Runtime](#update-the-image-with-a-new-version-of-the-java-runtime)

## Demo Setup

The folder [demo-setup](./demo-setup/) contains all required bits to configure all templates, credentials and inventories as well as rulebook activations on EDA that are needed for the demo.

### Prerequisites

- 1x VM for Red Hat Ansible Automation Platform
- 1x VM for Gitea, httpd and the image building services
- AAP 2.4+ installed and exposing EDA and Controller APIs
- Retrieve the Red Hat Automation Hub URLs and token from [console.redhat.com](https://console.redhat.com/ansible/automation-hub/token), as you will need them to fill the **automation_hub_** variables below.
- A Red Hat Account Username and password or [Service Account for accessing registry.redhat.io](https://access.redhat.com/RegistryAuthentication#registry-service-accounts-for-shared-environments-4) as you will need them to fill the **redhat_registry_** variables below.

### Lab deployment

Install the required ansible collections by running:

```
$ ansible-galaxy install -r demo-setup/requirements.yml --force
```

### demo-setup-vars.yml

This file contains all required variables to populate projects, credentials and templates, edit the values according to your environment:

``` yaml
### Red Hat Container Registry authentication (use Red Hat ID Username/Password or service account)
redhat_registry_url: registry.redhat.io
redhat_registry_username:
redhat_registry_password:

### URL and creds for AAP2 Controller
aap2_controller_url: https://aap.rh-lab.labs:8443
aap2_controller_username: admin
aap2_controller_password: redhat

### URL and creds for AAP2 EDA Controller
eda_controller_url: https://aap.rh-lab.labs:8445
eda_controller_username: admin
eda_controller_password: redhat

### URL for Red Hat Automation Hub
automation_hub_url: https://console.redhat.com/api/automation-hub/content/published/
automation_hub_auth_url: https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token
automation_hub_token:

### Hostname/IP of the server hosting builds, webserver and Gitea instance
server_hostname: rhel9-server.rh-lab.labs

### Credentials for the server
server_username: sysadmin
server_password: redhat

### Custom Gitea settings for Git and Container registry hosting
gitea_server_hostname: rhel9-server.rh-lab.labs
gitea_server_username: gitea
gitea_server_password: redhat
```

It is pre-filled with the demo setup fields, it MUST be changed according to your own environment.

### Configure Gitea Instance

Running the following command will end up creating:

- Containerized Gitea instance with self-signed certs
- Clone of the demo repositories made available in Gitea
- Webhooks configured on Gitea

Run the following command to proceed:

```
$ ansible-playbook demo-setup/configure-gitea-server.yml -i inventory
```

The instance will be available at https://{{ gitea_server_hostname }}:3000

### Configure AAP

To configure AAP and all related resources (Projects, inventories, etc) simply run:

```
$ ansible-playbook demo-setup/configure-aap.yml -i inventory
```

### Configure the build server

Once the AAP2 Configuration is done, log-in to the Automation Controller and run the template **[RH1][Image Mode Demo] Prepare builder server** to install required tools on the server for:

- Image building
- Image scanning
- Repo cloning
- Certificates setup

## Demo use cases

Our organization is willing to standardize the way developers build and ship their applications.
We decided to use RHEL Image mode to streamline the OS configuration and build/setup and our infrastructure team created a base image (we will refer to it as the Standard Operating Environment - SOE - Image) that developers will use to embed their application, Petclinic.

### Creating a SOE image

In the first use case we will create a base image that we will use for the application image build.
Go to the SOE repository you forked before (https://github.com/YOURGITHUBUSERNAME/rh1-image-mode-soe) and review the Containerfile.

As you can see it has a limited set of informations, that are considered enough from our infrastructure team to provide a base image for developers.

In GitHub or with your CLI, create a tag (i.e. v1.0-soe) and this will trigger an EDA action that runs the **[RH1][Image Mode Demo] Build Image Mode SOE Image** workflow that will:

- Clone the repo
- Build the image
- Run a container image scan
- Ask for approval to review the outcome of the scan
- If approved, it will push the image (by default it will be called *rhel-image-mode-demo* and it will have a *soe* image tag and another tag based on the GH commit) to the registry you defined in the variables

### Creating the application ISO image

Now that we built the SOE image, we want to provide our developers with the latest version of it to embed their application.

Go to the *container-app* repository you forked before (https://github.com/YOURGITHUBUSERNAME/rh1-image-mode-container-app.git) and review the Containerfile.

As you can see, it installs Java 11, the application bits, a DB and populates the DB with some initial data that is needed by the application.

We are now ready to generate the ISO image that we need to deploy.

> [!WARNING]
> It is important that the tag name contains the word *iso* as the Rulebook Activation in EDA performs a check on the tag name to distinguish between the diffrent kind of workflows o run.

> [!CAUTION]
> Replace **YOUR_REPOSITORY_URL** with the value of *rhel_image_mode_registry_url* variable.

In GitHub or with your CLI, create a tag (i.e. v1.0-iso) and this will trigger an EDA action that runs the **[RH1][Image Mode Demo] Build Image Mode application ISO** workflow that will:

- Clone the repo
- Build the image
- Run a container image scan
- Ask for approval to review the outcome of the scan
- If approved, it will push the image (by default it will be called *rhel-image-mode-demo* and it will have a *app* image tag and another tag based on the GH commit) to the registry you defined in the variables
- Start the conversion from container image to ISO image
- Upload the ISO image to our *webserver*

### Deploy our ISO on our device

Now that we have our ISO image in place, we can use it to install RHEL on our device.
To do so, we can create a bootable device (USB stick) with the ISO with a tool like [Fedora Media Writer](https://fedoraproject.org/de/workstation/download/).

If you don't have a device and you want to use a virtualized version, use your favourite Hypervisor to spin up the Virtual Machine using the ISO image you've just created

Once the deploy is completed, you can access your brand new VM with the application on top of it using the following credentials:

```bash
    username: bootc-user
    password: redhat
```

To verify that the application and the DB is running you can run:

```bash
sudo systemctl status mysqld petclinic
```

Verify that the installed Java version is 11:

```bash
java -version
```

In the installed system you will also find the utility called [bootc](https://github.com/containers/bootc) that will be responsible for managing ugprades/rollbacks and status checks for your RHEL Image mode system.

You can verify that you are using the correct image you created by issuing the following command:

```bash
sudo bootc status
```

### Update the image with a new version of the Java Runtime

Our developers updated the application to work with Java 17, so they are planning to release an update to the image to match the new version.

Go to the *container-app* repository you forked before (https://github.com/YOURGITHUBUSERNAME/rh1-image-mode-container-app.git) and review the *Containerfile-app17* file that contains the modifications.

As you can see, our developers bumped the installed JDK to version 17 and we want to propagate the change to the installed systems.

> [!CAUTION]
> Replace **YOUR_REPOSITORY_URL** with the value of *rhel_image_mode_registry_url* variable
> Copy the content of the *Containerfile-app17* into the *Containerfile* and push the changes

In GitHub or with your CLI, create a tag (i.e. v1.1-j17) and this will trigger an EDA action that runs the **[RH1][Image Mode Demo] Build Image Mode application image** workflow that will:

- Clone the repo
- Build the image
- Run a container image scan
- Ask for approval to review the outcome of the scan
- If approved, it will push the image (by default it will be called *rhel-image-mode-demo* and it will have a *app* image tag and another tag based on the GH commit) to the registry you defined in the variables

Once pushed, we can initiate the update on the system:

```bash
sudo bootc upgrade
```

Bootc will fetch the new image and upon next reboot the new version will be applied.

After logging back, you can verify that the Java version is 17:

```bash
java -version
```

And that the application is running:

```bash
sudo systemctl status mysqld petclinic
```
