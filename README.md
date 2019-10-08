cesnet.shibboleth_idp
======================

Ansible Galaxy role [cesnet.shibboleth_idp](https://galaxy.ansible.com/cesnet/shibboleth_idp) 
that installs [Shibboleth Identity Provider V3](https://wiki.shibboleth.net/confluence/display/IDP30/Home)
into system Tomcat.

Requirements
------------
Only Debian 10 is supported.

This role requires a number of files to be available. 
They will usually be located in the folder name **files/** relative to the playbook. 
If you install more than one IdP on a single machine, redefine the variables idp_*_file to point to other files.
The role provides dummy files with examples, but you have to provide your own files.

In Debian 10, applications running in system Tomcat cannot write outside of their folders for security reasons.
We recommend to use the role [cesnet.tomcat](https://galaxy.ansible.com/cesnet/tomcat) to enable it, see the example below. 

Encryption of files containings secrets
---------------------------------------

The files holding private keys (by default named **idp-encryption.key** and **idp-signing.key**, or the files referenced by 
variables **idp_encryption_key_file** and **idp_signing_key_file**) or password (**ldap.properties** or the file referenced by variable **idp_ldap_properties_file**)
should be encrypted as Ansible vaults, i.e. by issuing:
```bash
ansible-vault encrypt files/idp-encryption.key
ansible-vault encrypt files/idp-signing.key
ansible-vault encrypt files/ldap.properties
```
The dummy files are encrypted with the password "test". We recommend to put the vault password into a file named .password and
add the line "vault_password_file=.password" into ansible.cfg, then you will not have to type the password manually.

Role Variables
--------------

| variable                        | default       | description |
|---------------------------------|---------------|-------------|
| **idp_version**                 | 3.4.6         | software version|
| **idp_download_dir**            | /root         | directory to put downloaded file| 
| **idp_id**                      | idp           | identifier used in URL if multiple IdPs are installed|
| **idp_dir**                     | /opt/shibboleth-idp | directory where IdP is installed
| **idp_log_dir**                 | /var/log/idp  | directory with logs|
| **idp_entity_id**               | https://www.example.org/idp/shibboleth | SAML entityID|
| **idp_hostname**                | www.example.org | hostname in URL |
| **idp_scope**                   | example.org   | SAML scope |
| **idp_logo_file**               | example_logo.png | file with logo displayed on login screen |
| **idp_attribute_resolver_file** | attribute-resolver.xml | file with rules for getting SAML attribute values|
| **idp_attribute_filter_file**   | attribute-filter.xml | file with rules for releasing SAML attributes | 
| **idp_metadata_providers_file** | metadata-providers.xml | file with list of trusted metadata providers
| **idp_metadata_providers_signing_cert_files**| [ "metadata.eduid.cz.crt.pem" ] | list of files with certificates for checking validity of metadata |
| **idp_ldap_properties_file**    | ldap.properties | file with config for connection to LDAP server |
| **idp_messages_properties_file**| messages.properties | file modifying messages for page titles, footers, help links etc| 
| **idp_own_metadata_file**       | idp-metadata.xml | file containing IdP's own metadata |
| **idp_encryption_cert_file**    | idp-encryption.crt | file containing X509 certificate for encryption, must match the one in metadata| 
| **idp_encryption_key_file**     | idp-encryption.key | file containing private key for encryption, must match the encryption certificate | 
| **idp_signing_cert_file**       | idp-signing.crt | file containing X509 certificate for signing, must match the one in metadata|
| **idp_signing_key_file**        | idp-signing.key | file containing private key for signing, must match the encryption certificate |


Example Playbook
----------------
```yaml
- hosts: all
  tasks:
    - name: "install My IdP"
      import_role:
        name: cesnet.shibboleth_idp
      vars:
        idp_version: 3.4.6
        idp_download_dir: "/root"
        idp_id: "myidp"
        idp_dir: "/opt/my-idp"
        idp_log_dir: "/var/log/myidp"
        idp_entity_id: "https://www.example.org/myidp/shibboleth"
        idp_hostname: "www.example.org"
        idp_scope: "example.org"
        idp_logo_file: "example_logo.png"
    - name: "enable reading and writing outside of system Tomcat"
      import_role:
        name: cesnet.tomcat
      vars:
        tomcat_readwrite_paths: 
          - /opt/my-idp/metadata
          - /var/log/myidp
```
