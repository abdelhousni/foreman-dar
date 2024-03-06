# Install Foreman + Katello on RHEL8.9 + Hyper-V

- [ ] get from  https://docs.theforeman.org/nightly/Tuning_Performance/index-katello.html#Disable_Transparent_Hugepage_performance-tuning
 - [ ] System Requirements (DNS !! 20G of RAM !!) and to work around it ;)
 - [ ] Steps for installation
 - [ ] Issues, fixes and workarounds
- [ ] Hosts management
  - [ ] Steps for Content (CV, CCV, ...)
  - [ ] Redhat Subscriptions
  - [ ] Proxmox integration
  - [ ] Ansible integration

#### enable cockpit on different port (9099)
 ##### via Ansible (rhel-system-roles)
 - enable role 'rhel-system-roles.cockpit'
 - add both variables :
   - cockpit_manage_selinux	= true
   - cockpit_port = '9099'
 - apply role & enjoy !
 ##### Manually
 - systemctl edit cockpit.socket
   ```ini
   # https://cockpit-project.org/guide/latest/listen.html
   [Socket]
   ListenStream=
   ListenStream=9099
   #ListenStream=192.168.1.1:443
   #FreeBind=yes
   ```
-
  ```bash
  # per redhat 12.1. Allowing a new port on a system with active SELinux
  sudo semanage port -a -t websm_port_t -p tcp 9099
  sudo restorecon -Rv /etc/systemd/system
  # fix systemd error :  cockpit.socket: Failed with result 'resources'.
  sudo setsebool -P nis_enabled 1` # via https://github.com/cockpit-project/cockpit/issues/13419#issuecomment-652808768
  ```
via :
- https://cockpit-project.org/guide/latest/listen.html
- [12.1. Allowing a new port on a system with active SELinux](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/managing_systems_using_the_rhel_8_web_console/index#allowing-a-new-port-with-selinux_configuring-the-web-console-listening-port)
- https://github.com/cockpit-project/cockpit/issues/13419#issuecomment-652808768

### 
### 4.1. Adding Foreman server to AWX as a Dynamic Inventory Item

To add Foreman server to AWX as a dynamic inventory item, you must create a credential for a Foreman server user on AWX, add an AWX user to the credential, and then configure an inventory source.

Prerequisites

-   If your Foreman deployment is large, for example, managing tens of thousands of hosts, using a non-admin user can negatively impact performance because of time penalties that accrue during authorization checks. For large deployments, consider using an admin user.
    
-   For non-admin users, you must assign the `AWX Inventory Reader` role to your Foreman server user. For more information about managing users, roles, and permission filters, see [Creating and Managing Roles](https://docs.theforeman.org/2.5/Administering_Red_Hat_Satellite/index-foreman-el.html#sect-Administering-Users_and_Roles-Creating_and_Managing_Roles) in *Administering Foreman*.
    
-   You must host your Foreman server and AWX on the same network or subnet.
    

Procedure

1.  In the AWX web UI, create a credential for your Foreman. For more information about creating credentials, see [Add a New Credential](http://docs.ansible.com/ansible-tower/latest/html/userguide/credentials.html#add-a-new-credential) and [Foreman Credentials](http://docs.ansible.com/ansible-tower/latest/html/userguide/credentials.html#red-hat-satellite-6) in the *AWX User Guide*.
    
    Table 1. Foreman Credentials
    | **Credential Type**: | **Foreman** |
    | --- | --- |
    | **Foreman URL**: | https://*foreman.example.com*|
    | **Username**:| The username of the Foreman user with the integration role.|
    | **Password**:| The password of the Foreman user.|
    
2.  Add an AWX user to the new credential. For more information about adding a user to a credential, see [Getting Started with Credentials](http://docs.ansible.com/ansible-tower/latest/html/userguide/credentials.html#getting-started-with-credentials) in the *AWX User Guide*.
    
3.  Add a new inventory. For more information, see [Add a new inventory](http://docs.ansible.com/ansible-tower/latest/html/userguide/inventories.html#add-a-new-inventory) in the *AWX User Guide*.
    
4.  In the new inventory, add Foreman server as the inventory source, specifying the following inventory source options. For more information, see [Add Source](https://docs.ansible.com/ansible-tower/latest/html/userguide/inventories.html#add-source) in the *AWX User Guide*.
    
    Table 2. Inventory Source Options
    | **Source** | **Foreman** |
    | --- | --- |
    | **Credential** | The credential you create for Foreman server.|
    | **Overwrite**  | Select |
    | **Overwrite Variables** | Select|
    | **Update on Launch**| Select|
    | **Cache Timeout**| *90*|
    
5.  Ensure that you synchronize the source that you add.

> :warning: add `validate_certs: False` to inventory source for self-signed cert

### 4.2. Configuring Provisioning Callback for a Host

When you create hosts in Foreman, you can use AWX to run playbooks to configure your newly created hosts. This is called *provisioning callback* in AWX.

The provisioning callback function triggers a playbook run from AWX as part of the provisioning process. The playbook configures the host after Kickstart deployment.

For more information about provisioning callbacks, see [Provisioning Callbacks](http://docs.ansible.com/ansible-tower/latest/html/userguide/job_templates.html#provisioning-callbacks) in the *AWX User Guide*.

In Foreman server, the `Kickstart Default` and `Kickstart Default Finish` templates include three snippets:

1.  `ansible_provisioning_callback`
    
2.  `ansible_tower_callback_script`
    
3.  `ansible_tower_callback_service`
    

You can add parameters to hosts or host groups to provide the credentials that these snippets can use to run Ansible playbooks on your newly created hosts.

Prerequisites

Before you can configure provisioning callbacks, you must add Foreman as a dynamic inventory in AWX. For more information, see [Integrating Foreman and AWX](https://docs.theforeman.org/2.5/Configuring_Ansible/index-foreman-el.html#integrating-ansible-tower_ansible).

In the AWX web UI, you must complete the following tasks:

1.  Create a machine credential for your new host. Ensure that you enter the same password in the credential that you plan to assign to the host that you create in Foreman. For more information, see [Add a New Credential](https://docs.ansible.com/ansible-tower/latest/html/userguide/credentials.html#add-a-new-credential) in the *AWX User Guide*.
    
2.  Create a project. For more information, see [Projects](https://docs.ansible.com/ansible-tower/latest/html/userguide/projects.html) in the *AWX User Guide*.
    
3.  Add a job template to your project. For more information, see [Job Templates](https://docs.ansible.com/ansible-tower/latest/html/userguide/job_templates.html#create-a-job-template) in the *AWX User Guide*.
    
4.  In your job template, you must enable provisioning callbacks, generate the host configuration key, and note the *template\_ID* of your job template. For more information about job templates, see [Job Templates](http://docs.ansible.com/ansible-tower/latest/html/userguide/job_templates.html#) in the *AWX User Guide*.
    

Procedure

1.  In the Foreman web UI, navigate to **Configure** > **Host Group**.
    
2.  Create a host group or edit an existing host group.
    
3.  In the Host Group window, click the **Parameters** tab.
    
4.  Click **Add Parameter**.
    
5.  Enter the following information for each new parameter:
    
    Table 3. Host Parameters

    | Name | Value | Description |
    | --- | --- | --- |
    | `ansible_tower_provisioning` | true  | Enables Provisioning Callback.|
    | `ansible_tower_fqdn` | *tower.example.com* | The fully qualified domain name (FQDN) of your AWX. Do not add `https` because this is appended by AWX.|
    | `ansible_job_template_id` | *template\_ID* | The ID of your provisioning template that you can find in the URL of the template: `/templates/job_template/*5*`.|
    |`ansible_host_config_key`| *config\_KEY* | The host configuration key that your job template generates in AWX.|
    
7.  Click **Submit**.
    
8.  Create a host using the host group.
    
9.  On the new host, enter the following command to start the `ansible-callback` service:
    
    `# systemctl start ansible-callback`
    
10.  On the new host, enter the following command to output the status of the `ansible-callback` service:
    
    `# systemctl status ansible-callback`
    
    Provisioning callback is configured correctly if the command returns the following output:
    
    SAT\_host systemd\[1\]: Started Provisioning callback to AWX...
    

Manual Provisioning Callback

You can use the provisioning callback URL and the host configuration key from a host to call AWX. For example:

```bash
# curl -k -s --data curl --insecure --data host\_config\_key=*my\_config\_key* \\
 https://*tower.example.com*/api/v2/job\_templates/*8*/callback/
```

Ensure that you use `https` when you enter the provisioning callback URL.

This triggers the playbook run specified in the template against the host.

ref : https://docs.theforeman.org/2.5/Configuring_Ansible/index-foreman-el.html#integrating-ansible-tower_ansible

### References
* <a id="1">[1]</a>: https://docs.theforeman.org/nightly/Installing_Server/index-foreman-el.html#system-requirements_foreman
