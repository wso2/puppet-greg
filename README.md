# WSO2 Governance Registry Puppet Module

This repository contains the Puppet Module for installing and configuring WSO2 Governance Registry on various environments. It supports multiple versions of WSO2 Governance Registry. Configuration data is managed using [Hiera](http://docs.puppetlabs.com/hiera/1/). Hiera provides a mechanism for separating configuration data from Puppet scripts and managing them in a separate set of YAML files in a hierarchical manner.

## Supported Operating Systems

- Debian 6 or higher
- Ubuntu 12.04 or higher

## Supported Puppet Versions

- Puppet 2.7, 3 or newer

## How to Contribute
Follow the steps mentioned in the [wiki](https://github.com/wso2/puppet-base/wiki) to setup a development environment and update/create new puppet modules.

## Add WSO2 Base Puppet Module

Run the following commands to get wso2base submodule.

````
git submodule init
git submodule update
````

## Packs to be Copied

Copy the following files to their corresponding locations.

1. WSO2 Governance Registry distribution (5.2.0 or 5.1.0 or 4.6.0) to `<PUPPET_HOME>/modules/wso2greg/files`
2. JDK 1.7_80 distribution to `<PUPPET_HOME>/modules/wso2base/files` 

Note: For wso2greg_pubstore puppet module use the WSO2 Governance Registry distribution (5.1.0) which has installed API Manager Publisher and Store features.

## Running WSO2 Governance Registry in the `default` profile
No changes to Hiera data are required to run the `default` profile.  Copy the above mentioned files to their corresponding locations and apply the Puppet Modules.

## Running WSO2 Governance Registry with clustering in specific profiles
No changes to Hiera data are required to run the distributed deployment of WSO2 Governance Registry, other than pointing to the correct resources such as the deployment synchronization and remote DB instances. For more details refer the [WSO2 Governance Registry 5.0.0](https://docs.wso2.com/display/CLUSTER44x/Clustering+Governance+Registry+5.0.0) and [WSO2 Governance Registry 4.6.0](https://docs.wso2.com/display/CLUSTER420/Clustering+Governance+Registry) clustering guide.

1. If the Clustering Membership Scheme is `WKA`, add the Well Known Address list.

   Ex:
    ```yaml
    wso2::clustering:
        enabled: true
        local_member_host: "%{::ipaddress}"
        local_member_port: 4000
        membership_scheme: wka
        sub_domain: mgt
        wka:
           members:
             -
               hostname: 192.168.100.103
               port: 4000
             -
               hostname: 192.168.100.104
               port: 4000
    ```

2. Uncomment and modify the MySQL based data sources to point to the external MySQL servers.

   Ex:
    ```yaml
    wso2::master_datasources:
      wso2_config_db:
        name: WSO2_CONFIG_DB
        description: The datasource used for config registry
        driver_class_name: "%{hiera('wso2::datasources::mysql::driver_class_name')}"
        url: jdbc:mysql://192.168.100.1:3306/WSO2_GREG_DB?autoReconnect=true
        username: "%{hiera('wso2::datasources::mysql::username')}"
        password: "%{hiera('wso2::datasources::mysql::password')}"
        jndi_config: jdbc/WSO2_CONFIG_DB
        max_active: "%{hiera('wso2::datasources::common::max_active')}"
        max_wait: "%{hiera('wso2::datasources::common::max_wait')}"
        test_on_borrow: "%{hiera('wso2::datasources::common::test_on_borrow')}"
        default_auto_commit: "%{hiera('wso2::datasources::common::default_auto_commit')}"
        validation_query: "%{hiera('wso2::datasources::mysql::validation_query')}"
        validation_interval: "%{hiera('wso2::datasources::common::validation_interval')}"

    ```

3. Uncomment (and optionally configure) registry mounting

   Ex:
    ```yaml
    wso2_config_db:
      path: /_system/config
      target_path: /_system/config
      read_only: false
      registry_root: /
      enable_cache: true

    wso2_gov_db:
      path: /_system/governance
      target_path: /_system/governance
      read_only: false
      registry_root: /
      enable_cache: true
    ```

## Running WSO2 Governance Registry with Secure Vault
WSO2 Carbon products may contain sensitive information such as passwords in configuration files. [WSO2 Secure Vault](https://docs.wso2.com/display/Carbon444/Securing+Passwords+in+Configuration+Files) provides a solution for securing such information.

For WSO2 Governance Registry 4.6.0, which is based on WSO2 Carbon Kernel 4.2.0, all the Carbon Kernel patches from patch0002 to patch0008 have to be applied first. Then the `org.wso2.ciphertool-1.0.0-wso2v2.jar` in Kernel patch [patch0010](http://dist.wso2.org/maven2/org/wso2/carbon/WSO2-CARBON-PATCH-4.2.0/0010/) has to be applied before enabling the Secure Vault. The `org.wso2.ciphertool-1.0.0-wso2v2.jar` in `WSO2-CARBON-PATCH-4.2.0-0009/lib` has to be copied to `wso2greg/files/configs/lib` folder and added to the `file_list` in hiera file as below:

```yaml
wso2::file_list :
  - lib/org.wso2.ciphertool-1.0.0-wso2v2.jar
```

Uncomment and modify the below changes in Hiera file to apply Secure Vault.

1. Enable Secure Vault

    ```yaml
    wso2::enable_secure_vault: true
    ```

2. Add Secure Vault configurations as below

    ```yaml
    wso2::secure_vault_configs:
      <secure_vault_config_name>:
        secret_alias: <secret_alias>
        secret_alias_value: <secret_alias_value>
        password: <password>
    ```

    For Governance Registry `5.1.0` and `5.2.0` which is based on WSO2 Carbon Kernel 4.4.x

    Ex:
    ```yaml
    wso2::secure_vault_configs:
      key_store_password:
        secret_alias: Carbon.Security.KeyStore.Password
        secret_alias_value: repository/conf/carbon.xml//Server/Security/KeyStore/Password,false
        password: wso2carbon
    ```

    For Governance Registry `4.6.0` which is based on WSO2 Carbon Kernel 4.2.0

    Ex:
    ```yaml
    wso2::secure_vault_configs:
      key_store_password:
        secret_alias: Carbon.Security.KeyStore.Password
        secret_alias_value: carbon.xml//Server/Security/KeyStore/Password,true
        password: wso2carbon
    ```

3. Add Cipher Tool configuration file templates to `template_list`

    ```yaml
    wso2::template_list:
      - repository/conf/security/cipher-text.properties
      - repository/conf/security/cipher-tool.properties
      - bin/ciphertool.sh
      - password-tmp
    ```

    Please add the `password-tmp` template also to `template_list` if the `vm_type` is not `docker` when you are running the server in `default` platform.

## Running WSO2 Governance Registry on Kubernetes
WSO2 Puppet Module ships Hiera data required to deploy WSO2 Governance Registry on Kubernetes. For more information refer to the documentation on [deploying WSO2 products on Kubernetes using WSO2 Puppet Modules](https://docs.wso2.com/display/PM210/Deploying+WSO2+Products+on+Kubernetes+Using+WSO2+Puppet+Modules).
