Collection of Ansible playbooks to install *stuff* on OpenShift.

Playbooks for:
* 3scale
* ArgoCD
* Gitea
* NooBaa
* OpenShift Pipelines
* OpenShift Serverless

### Installation of 3scale with S3 storage (through NooBaa) and SSO

#### Prerequisites
* OCP 4.6.x cluster running on AWS
* Access to S3
* Cluster admin access
* `oc` OpenShift CLI tool
* Ansible 2.9.x with Kubernetes `k8s` module installed

#### Installation of Noobaa operator and creation of S3 Buckets

* Make sure you are logged in into the cluster as cluster administrator.
* Change directory to the `noobaa` directory.
* Run the installation playbook:
  ```
  $ ansible-playbook playbook/noobaa.yml
  ```
* Make note of the AWS Access Key and the AWS Secret Access Key
  ```
  TASK [../roles/noobaa : noobaa s3 bucket aws access key id] ********************************************************************************************
  ok: [localhost] => {
      "msg": "AWS Access Key ID: OX3uxxxxxxxxxxxxxxxx"
  }

  TASK [../roles/noobaa : noobaa s3 bucket aws secret access key] ****************************************************************************************
  ok: [localhost] => {
      "msg": "AWS Secret Access Key: JS688kyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy"
  }
   ```

#### Installation of 3scale AMP

* Make sure you are logged in into the cluster as cluster administrator.
* Change directory to the `3scale` directory.
* Copy the `inventories/inventory.template` inventory file:
  ```
  $ cp inventories/inventory.template inventories/inventory
  ``` 
* Complete the inventory file. Pay attention to the following parameters:
  * **threescale_namespace**: OpenShift namespace where the 3scale platform will be installed.
  * **threescale_subscription_channel**: OpenShift OLM channel for the 3scale operators.
  * **threescale_admin_email**: email of the 3scale AMP admin. Should be a real email address if 3scale is integrated with SMTP.
  * **threescale_noobaa_aws_key**: NooBaa S3 bucket AWS Access Key ID (see above).
  * **threescale_noobaa_aws_secret**: NooBaa S3 bucket AWS Secret Access Key (see above).
  * **threescale_namespace_noobaa_operator**: nnamespace where NooBaa is deployed.
  * **threescale_smtp_**: SMTP settings for outgoing email. Leave blank if email integration is not needed.
* Run the installation playbook:
  ```
  $ ansible-playbook -i inventories/inventory playbooks/3scale.yml
  ```

#### Installation of 3scale tenants

* Complete the inventory file. Pay attention to the following parameters:
  * **threescale_start_tenant**, **threescale_end_tenant**: start and end sequence number for the generated tenants. The sequence number is appended to the **tenant_admin_name_base**.
  * **admin_email_user**, **admin_email_domain**: used to set email address of the tenant admin. The email address format is `{{ admin_email_user }}+{{ ocp_user_name_base }}{{ item }}@{{ admin_email_domain }}`, where `{{ item }}` is the sequence number of the tenant. Should be a real email address if 3scale is integrated with SMTP.
* Run the installation playbook:
  ```
  $ ansible-playbook -i inventories/inventory playbooks/3scale.yml -e ACTION=create_tenants
  ```

#### Installation of RHSSO

* Make sure you are logged in into the cluster as cluster administrator.
* Change directory to the `rhsso` directory.
* Copy the `inventories/inventory.template` inventory file:
  ```
  $ cp inventories/inventory.template inventories/inventory
  ``` 
* Complete the inventory file. Pay attention to the following parameters:
  * **rhsso_namespace**: OpenShift namespace where RHSSO will be installed.
* Run the installation playbook:
  ```
  $ ansible-playbook -i inventories/inventory playbooks/rhsso.yml
  ```

#### Installation of RHSSO realms
* Complete the inventory file. Pay attention to the following parameters:
  * **realm_first**, **realm_last**: first and  last sequence number for the generated realms. The sequence number is appended to **realm_base_name**.
  * **realm_base_name**: base name for the realm name.
* Run the installation playbook:
  ```
  $ ansible-playbook -i inventories/inventory playbooks/rhsso.yml -e ACTION=create_realms
  ```