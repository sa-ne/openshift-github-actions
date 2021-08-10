# openshift-github-actions

This code is meant to show working examples of deploying OpenShift and OpenShift components using GitHub Actions.


**DISCLAIMER** 
This code is meant to show a low barrier to entry for automation in regards to deploying OpenShift and OpenShift components in AWS.  Beware! Some jobs may not demonstrate best practices.

### Table of Contents
**[How to use](#how-to-use)**<br>
**[Available Actions](#available-actions)**<br>
**[Frequently-Asked-Questions](#frequently-asked-questions)**<br>
**[Maintainers](#maintainers)**<br>

## How-to-use

In order to use the workflows and actions, you must fork this repo first.  A fork is required because you will need to set secrets in order for the workflows to complete successfully.  Only owners/administrators can edit secrets on repositories.

---
#### Set security for Actions after creating a fork
By default, when forking a new repository, all actions are allowed.  To better protect yourself, change the Action Permissions to `Allow select actions` and whitelist the specified actions.  For this code, the following actions need to be whitelisted:
```
actions/*,
aws-actions/*,
redhat-actions/*,
```
![allow select actions](/assets/images/allow_select_actions.png)

Additionally, you may want to update the `Workflow permissions` on the bottom of the Actions settings to `Read repository contents permission`

![workflow permissions](/assets/images/workflow_permissions.png)

---

**WARNING**: By default, Actions logs are public. Anyone can view them.

---
#### Create required Repository Secrets
Add the following secrets to the `Repository secrets`.  Once created, the secrets will not be shown again.

`AWS_ACCESS_KEY_ID`: Specifies the AWS access key used as part of the credentials to authenticate the user.

`AWS_SECRET_ACCESS_KEY`: Specifies the AWS secret key used as part of the credentials to authenticate the user.

`PULL_SECRET`: From cloud.redhat.com/openshift/downloads, under Tokens.  The pull secret can be used for any version of OpenShift.  The pull secret will change if the credentials used to login to cloud.redhat.com are changed.

`EMAIL`: E-mail to associate with OpenShift install-config.yaml and certbot.

`OC_USER`: Depending on the workflow, `OC_USER` is either created or used as an existing user to login after the `kubeadmin` user is removed.

`OC_PASSWORD`: Password associated with `OC_USER`.

To set secrets, go to `Settings`, `Secrets`, ` New repository secret` (_note: not all secrets that should be set are shown in the screen grab_):

![setting secrets](/assets/images/setting_secrets.png)

#### Enable workflows

To populate the Actions on your fork, they must be enabled.

![enable workflows](/assets/images/enable_workflows_on_fork.png)

---

### Available-Actions

**Deployment Actions**

**[deploy-openshift](#deploy-openshift)**<br>
**[deploy-argocd](#deploy-argocd)**<br>
**[deploy-acm](#deploy-acm)**<br>
**[deploy-acm-with-argo](#deploy-acm-with-argo)**<br>
**[deploy-odf](#deploy-odf)**<br>
**[deploy-acs-with-argo](#deploy-acs-with-argo)**<br>
**[deploy-quay-with-argo](#deploy-quay-with-argo)**<br>
**[deploy-bastion-host](#deploy-bastion-host)**<br>

**Configuration Actions**

**[configure-ssl-cert](#configure-ssl-cert)**<br>
**[remove-kubeadmin-user](#remove-kubeadmin-user)**<br>

**Deprovision Actions**

**[remove-cluster](#remove-cluster)**<br>
**[force-remove-cluster](#force-remove-cluster)**<br>

**Application Actions**

**[prepull-windows-image](#prepull-windows-image)**<br>
**[deploy-netcandystore](#deploy-netcandystore)**<br>

---

### deploy-openshift

- Deploy a specified version of OpenShift to AWS, creating a "bare-bones" OpenShift instance.
- All deployment metadata is stored in an S3 bucket created in the beginning of the workflow.
- The kubeadmin user is removed and the `OC_USER` is created.
- SSH keys are generated, keys are sent to the S3 storage bucket.
- The OpenShift cluster certificate, by default will encrypt traffic.  However it is not signed by a CA so it will appear insecure in the browser.  If you check the cert on [SSL checker](https://www.sslshopper.com/ssl-checker.html), it will show secure until the very end of the chain by default.  This job will finish securing the certificate by using Let's Encrypt via the route53 plugin.

#### For use with RHPDS Open Environments

Please see the e-mail you received to set the following:

Inputs:
`region`
`baseDomain`

Secrets:
`AWS_ACCESS_KEY_ID`
`AWS_SECRET_ACCESS_KEY`

![RHPDS Open Environment Email](/assets/images/RHPDS_Email.png)



#### To support Windows containers
For a cluster that supports Windows containers, update the `networkType` to `OVNKubernetes` when inputting parameters for the `deploy-openshift` Action.  

_NOTE:_ No Windows machineSets are deployed with the `deploy-openshift` workflow.

---

### deploy-openshift-plus

- Deploy OpenShift using the same workflow as [deploy-openshift](#deploy-openshift)
- Also deploy ArgoCD
- Using ArgoCD it deploys ACM, ACS and Quay
- Upload the `credentials.txt` file to S3 containing the URLs and Credentials generated during ACM/ACS and Quay deployment


### deploy-acm

- Deploys the ACM operator
- Creates a pull-secret associated with ACM
- Deploy a MultiClusterHub instance to install ACM

---

### deploy-odf

- Deploys a new MachineSet to provision 3 new servers for ODF
- Deploys the OpenShift Data Foundation (aka OCS) operator
- Deploys a StorageCluster instance to install ODF

---

### deploy-argocd

- Deploys the OpenShift GitOps operator and installs ArgoCD on project openshift-gitops
- After deployment the initial password for admin user can be obtained running this command: `oc extract secret/openshift-gitops-cluster -n openshift-gitops --to=-`

---

### deploy-acm-with-argo

- Also deploys Advanced Cluster Management (ACM) but using ArgoCD
- ArgoCD is required to be already installed and running on openshift-gitops project

---

### deploy-acs-with-argo

- Deploys Advanced Cluster Security (ACS/Stackrox) using ArgoCD
- ArgoCD is required to be already installed and running on openshift-gitops project

---

### deploy-quay-with-argo

- Deploys Quay using ArgoCD
- ArgoCD is required to be already installed and running on openshift-gitops project

---

### deploy-bastion-host

- The workflow assumes you used the `deploy-openshift` workflow to provision the cluster.  It will copy down the S3 bucket and use the key located in the `/ssh-keys/` folder for the key pair.
- This workflow will provision a Bastion host in AWS with a public IP.  
- To SSH into the Bastion host, use the key from the `/ssh-keys/` folder for the cluster.

_NOTE: The OpenShift Container Platform installer does not create any public IP addresses for any of the Amazon Elastic Compute Cloud (Amazon EC2) instances that it provisions for your OpenShift Container Platform cluster. To be able to SSH to your OpenShift Container Platform hosts, you must provision a Bastion (jump box) host._

---
### configure-ssl-cert

**NOTE: `OC_USER` and `OC_PASSWORD` must be a valid openshift login to run this Action**

- Assuming the cluster is in AWS, using route 53 this job will use certbot + let's encrypt to act as a CA to sign the certificate.  

_NOTE: The certificate that is provisioned with OpenShift, while it is encrypted, it will show "un-secure" on a browser until signed by a CA due to how the certificate chain works._

---

### remove-kubeadmin-user

- Removes the kubeadmin user
- Sets htpasswd as oauth
- Uses the `OC_USER` and `OC_PASSWORD` secrets to configure a new user.

---

### remove-cluster

- Destroy an OpenShift cluster using the metadata files from the deployment sourced from S3.

---

### force-remove-cluster

- Destroy an OpenShift cluster without relying on any metadata from the original deployment.  It is a destroy hack!

---

### prepull-windows-image

- Pre-pull the Windows container image on the MachineSet.
- This workflow assumes you have the metadata from the install and the ssh key used to configure the WMCO in S3.

_NOTE: Until the timeout is increased to 30 minutes for pulling an image, all Windows images need to be pulled in advance of the container deployment_

---

### deploy-netcandystore

- Deploy the [NetCandy Store](http://people.redhat.com/chernand/windows-containers-quickstart/ns-intro/) a mixed environment consisting of Windows Containers and Linux Containers using helm.

_NOTE: The helm install supports Windows Server 2019 Datacenter 1809_

This application consists of:

- Windows Container running a .NET v4 frontend, which is consuming a backend service.
- Linux Container running a .NET Core backend service, which is using a database.
- Linux Container running a MSSql database.

---

## Frequently-Asked-Questions

##### Can I use the Actions with RHPDS Open Environments?

- Yes, absolutely!

---
##### What is the difference between the `s3_storage` input and the `clusterConfigName`?

- The `s3_storage` input refers to the bucket name, this **MUST be unique to the region where the storage bucket is created**.
- The `clusterConfigName` refers to the object

![s3 screen grab](/assets/images/s3_storage_example.png)

---
##### Where can I find my pull-secret?

Login to [cloud.redhat.com](cloud.redhat.com), select `OpenShift`, then `Downloads` and scroll to the bottom of the page where `Tokens` are listed.  Hit the `copy` button and then paste into your `PULL_SECRET` repository secret.

![pull secret](/assets/images/pull_secret.png)

---
##### How do I login to my OpenShift instance locally after using the `deploy-openshift` action?

- The console URL will be output in Apply ssl cert against OpenShift step.

![console ur](/assets/images/console_url.png)

- Ensure the AWS CLI and OC CLI are installed locally.
- Remember, metadata from the deployment is stored in AWS assuming you used the `openshift-deploy` action to provision the cluster.  
- Pull down the data from S3 (you may need to first authenticate with your aws credentials by running `aws configure`).
    - `aws s3 sync s3://<your-s3-storage-name)/ocp-<insert-run-id>/ ./ocp-<insert-run-id>`
- Export the kube config:
    - `export KUBECONFIG=./ocp-<insert-run-id>/auth/kubeconfig`
    - `oc login -u <USERNAME> -p '<PASSWORD>'`

Alternatively, you can also obtain a token to login by visiting: `https://oauth-openshift.apps.ocp-<insert-run-id>.<base domain>/oauth/token/request` or
via the console (`https://console-openshift-console.apps.ocp-<insert-run-id>.<base domain>/`)

Via the console, after authentication, click on the user in the upper right and hit `copy login command`, this will push you to the api login, where you will need to authenticate again.

![oc login command](/assets/images/oc_login_command.png)

After logging in, you can copy and paste the `oc login command`

![tokens](/assets/images/tokens.png)

---

##### The `openshift-deploy` action is failing with AddressLimitExceeded

If you are using an RHPDS Open Environment Elastic IPs are limited to 5.  You may run into this error if you're deploying more than two clusters.

![elastic IPs exceeded](/assets/images/elastic_IPs_exceeded.png)

---
##### Invalid AMI
Every few months AWS updates AMI's, you may get an error if AWS removes an older AMI.   While we do our best to ensure the default AMI's are available, you may hit this issue before we do.  Please open a pull request or issue if you run into this!

---

## Maintainers

- Giovanni Fontana (@giofontana)
- Dina Muscanell (@devopsdina)
