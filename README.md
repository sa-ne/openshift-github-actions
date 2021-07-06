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

**[deploy-openshift](#deploy-openshift)**<br>
**[deploy-acm](#deploy-acm)**<br>
**[configure-ssl-cert](#configure-ssl-cert)**<br>
**[remove-kubeadmin-user](#remove-kubeadmin-user)**<br>
**[remove-cluster](#remove-cluster)**<br>
**[force-remove-cluster](#force-remove-cluster)**<br>
**[prepull-windows-image](#prepull-windows-image)**<br>
**[deploy-netcandystore](#deploy-netcandystore)**<br>

---

### deploy-openshift

- Deploy a specified version of OpenShift to AWS, creating a "bare-bones" OpenShift instance.
- All deployment metadata is stored in an S3 bucket created in the beginning of the workflow.
- The kubeadmin user is removed and the `OC_USER` is created.
- SSH keys are generated, keys are sent to the S3 storage bucket.
- The OpenShift cluster certificate, by default will encrypt traffic.  However it is not signed by a CA so it will appear insecure in the browser.  If you check the cert on [SSL checker](https://www.sslshopper.com/ssl-checker.html), it will show secure until the very end of the chain by default.  This job will finish securing the certificate by using Let's Encrypt via the route53 plugin.

#### To support Windows containers
For a cluster that supports Windows containers, update the `networkType` to `OVNKubernetes` when inputting parameters for the `deploy-openshift` Action.  _NOTE:_ No Windows machineSets are deployed with this workflow.

---

### deploy-acm

- Deploys the ACM operator
- Creates a pull-secret associated with ACM

---

### configure-ssl-cert

**This job still needs to be tested**

Assuming the cluster is in AWS, using route 53 this job will use certbot + let's encrypt to act as a CA to sign the certificate.  

The certificate that is provisioned with OpenShift, while it is encrypted, it will show "un-secure" on a browser until signed by a CA due to how the certificate chain works.

---

### remove-kubeadmin-user

**This job still needs to be tested**

- Removes the kubeadmin user
- Sets htpasswd as oauth
- Uses the OC_USER and OC_PASSWORD secrets to configure a new user.

---

### remove-cluster

Destroy an OpenShift cluster using the metadata files from the deployment sourced from S3.

---

### force-remove-cluster

Destroy an OpenShift cluster without relying on any metadata from the original deployment.  It is a destroy hack!

---

### prepull-windows-image

- Pre-pull the Windows container image on the MachineSet.


Until the timeout is increased to 30 minutes for pulling an image, all Windows images need to be pulled in advance of the container deployment

This workflow assumes you have the metadata from the install and the ssh key used to configure the WMCO in S3.

---

### deploy-netcandystore

Deploy the [NetCandy Store](http://people.redhat.com/chernand/windows-containers-quickstart/ns-intro/) a mixed environment consisting of Windows Containers and Linux Containers using helm.

_NOTE: The helm install supports Windows Server 2019 Datacenter 1809_

This application consists of:

- Windows Container running a .NET v4 frontend, which is consuming a backend service.
- Linux Container running a .NET Core backend service, which is using a database.
- Linux Container running a MSSql database.

---

## Frequently-Asked-Questions

##### Can I use the Actions with RHPDS Open Environments?

- Yes, absolutely!

##### What is the difference between the `s3_storage` input and the `clusterConfigName`?

- The `s3_storage` input refers to the bucket name 
- The `clusterConfigName` refers to the object

![s3 screen grab](/assets/images/s3_storage_example.png)

##### Where can I find my pull-secret?

Login to [cloud.redhat.com](cloud.redhat.com), select `OpenShift`, then `Downloads` and scroll to the bottom of the page where `Tokens` are listed.  Hit the `copy` button and then paste into your `PULL_SECRET` repository secret.

![pull secret](/assets/images/pull_secret.png)

---

## Maintainers

- Giovanni Fontana (@giofontana)
- Dina Muscanell (@devopsdina)
