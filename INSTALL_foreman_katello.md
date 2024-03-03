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

### References
* <a id="1">[1]</a>: https://docs.theforeman.org/nightly/Installing_Server/index-foreman-el.html#system-requirements_foreman
