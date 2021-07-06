# This repo has been moved:

[https://github.com/sa-ne/openshift-github-actions](https://github.com/sa-ne/openshift-github-actions)

# ocp-ipi-aws-install

Example GitHub Actions workflows to:

- Deploy an OpenShift Cluster.
- Configuring the final leg of the SSL certificate to remove the website warning, using certbot and route53.
- Removing the kubeadmin user.
- Removing the OpenShift Cluster from AWS, with the metadata files created during deployment.
- Force removing the OpenShift Cluster from AWS, without any metadata files.

S3 is used for file storage of the pull secret required for installation.

S3 is used to store metadata for each run of the `deploy-openshift` workflow, assuming the installer completes successfully.

**DISCLAIMER**
This repo is not meant to demonstrate best practices or extensibility.  This repo is meant to show a low barrier to entry for automation in regards to deploying OpenShift in AWS.

## Workflows
### `deploy-openshift.yml`

- The installer is run in multiple steps to allow for creation of the `install-config` file.

- SSH keys are generated during each run for the cluster.  All keys are sent to the S3 storage bucket.

- The OpenShift cluster certificate, by default will encrypt traffic.  However it is not signed by a CA so it will appear insecure in the browser.  If you check the cert on [SSL checker](https://www.sslshopper.com/ssl-checker.html), it will show secure until the very end of the chain by default.  This job will finish securing the certificate by using Let's Encrypt via the route53 plugin.

#### Requirements for `deploy-openshift.yml`

An AWS account which has permission to create a S3 Storage bucket with will be used to store the files created during the installation. This github action will create this bucket automatically using the AWS Access Key set on the repository secrets.

**The following secrets are set:**

_Used to login to AWS:_
- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY

_Used for certificate configuration_
- EMAIL

_Used for oc login to apply certs_:
- OC_USER
- OC_PASSWORD

_Used for OpenShift deployment_:
- PULL_SECRET

### `configure-ssl-cert.yml`

**This job still needs to be tested**

Assuming the cluster is in AWS, using route 53 this job will use certbot + let's encrypt to act as a CA to sign the certificate.  The certificate that is provisioned with OpenShift, while it is encrypted, it will show "un-secure" on a browser until signed by a CA due to how the certificate chain works.

#### Requirements for `configure-ssl-cert.yml`

An AWS S3 Storage bucket with the following:

- S3 bucket with permissions set accordingly
- The OpenShift metadata files

The following secrets are used in this workflow:

_Used to login to AWS:_
- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY

_Used for certificate configuration_
- EMAIL

_Used for oc login to apply certs_:
- OC_USER
- OC_PASSWORD

### `remove-kubeadmin-user`

Removes the kubeadmin user, sets htpasswd as oauth and uses the OC_USER and OC_PASSWORD secrets to configure a new user.

**This job still needs to be tested**

The following secrets are used in this workflow:
- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY
- BASEDOMAIN
    - The base domain, usually your organizations or personal one.  Example: test.mycompany.com
- OC_INIT_USER
    - existing user (most likely kubeadmin)
- OC_INIT_PASSWORD
    - existing password
- OC_USER
    - New user to be configured
- OC_PASSWORD
    - Password for new user

### `remove-cluster`

This workflow will destroy the OpenShift cluster.  This workflow assumes you have the metadata files from the original deployment in an S3 bucket.

The following secrets are used in this workflow:
_required to pull the OpenShift installer and corresponding metadata files_
- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY

### `force-remove-cluster`

This workflow will destroy the OpenShift cluster.  This workflow does not rely on any metadata from the original deployment.  It is a destroy hack!

The following secrets are used in this workflow:
_required to pull the OpenShift installer_
- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY

### `prepull-windows-image`

This workflow will prepull the Windows container image on the MachineSet.  Until the timeout is increased to 30 minutes for pulling an image, all Windows images need to be pull in advance of the actual container deployment

This workflow assumes you have the metadata from the install and the ssh key used to configure the WMCO in S3 storage.

The following secrets are used in this workflow:
_required to pull the OpenShift metadata_
- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY

### `deploy-netcandystore`

Deploy the [NetCandy Store](http://people.redhat.com/chernand/windows-containers-quickstart/ns-intro/) a mixed environment consisting of Windows Containers and Linux Container using helm.

This application consists of:

- Windows Container running a .NET v4 frontend, which is consuming a backend service.
- Linux Container running a .NET Core backend service, which is using a database.
- Linux Container running a MSSql database.

The following secrets are used in this workflow:
- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY
- OC_USER
- OC_PASSWORD
