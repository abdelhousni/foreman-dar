# Foreman/Satellite cloud-init provisioning (VMware & PRoxmox)

based on https://www.redhat.com/en/blog/how-prepare-content-red-hat-satellite

## Procedure

For RHEL8

### Adding Software Repositories to Satellite
Add following Red Hat Products :
* Red Hat Enterprise Linux 8 for x86_64 - AppStream RPMs 8
* Red Hat Enterprise Linux 8 for x86_64 - BaseOS RPMs 8
* Red Hat Satellite Client 6 for RHEL 8 x86_64 RPMs

![image](https://github.com/abdelhousni/foreman-dar/assets/23284113/9448cc4c-2585-4884-ac1f-73da5e3cdb45)
![image](https://github.com/abdelhousni/foreman-dar/assets/23284113/fa527626-b719-4ad6-8444-f63ec05df789)
![image](https://github.com/abdelhousni/foreman-dar/assets/23284113/8aec29a6-4887-4f5b-bd80-2a9b172c1e14)
![image](https://github.com/abdelhousni/foreman-dar/assets/23284113/40fb6276-ebe7-4c63-a73a-bfaf3fbb0df1)
![image](https://github.com/abdelhousni/foreman-dar/assets/23284113/b638257f-dc43-4d17-9f47-af88ee059c4a)
![image](https://github.com/abdelhousni/foreman-dar/assets/23284113/67c2c726-6a52-446d-a167-8a99aa7a245f)

### Creating Content Lifecycles in Satellite

create our first lifecycle environment and link it our organization :
`hammer lifecycle-environment create --description le-ops-rhel8-prem-server --prior Library --name le-ops-rhel8-prem-server --organization "operations"`

List the lifecycle environments :
`hammer lifecycle-environment list`

Add a content view to the lifecycle environment. Content view allows to control the specific content made available to environments
`hammer content-view create --description cv-rhel8-prem-server --name cv-rhel8-prem-server --organization "dar"`

List "shorter" view of the repositories listing only the repository ID and name
```sh
hammer repository list --fields THIN --organization-label dar
----|---------------------------------------------------------------
ID  | NAME
----|---------------------------------------------------------------
330 | Debian 10 main
332 | Debian 11 main
43  | EPEL8 x86_64
44  | EPEL9 x86_64
53  | gitlab_gitlab-ce_EL8
54  | gitlab_gitlab-ce_EL9
1   | Red Hat Ansible Automation Platform 2.4 for RHEL 8 x86_64 RPMs
10  | Red Hat Ansible Automation Platform 2.4 for RHEL 9 x86_64 RPMs
101 | Red Hat Ansible Engine 2.9 for RHEL 8 x86_64 RPMs
9   | Red Hat CodeReady Linux Builder for RHEL 8 x86_64 RPMs 8
8   | Red Hat CodeReady Linux Builder for RHEL 9 x86_64 RPMs 9
6   | Red Hat Enterprise Linux 8 for x86_64 - AppStream RPMs 8
5   | Red Hat Enterprise Linux 8 for x86_64 - BaseOS RPMs 8
4   | Red Hat Enterprise Linux 9 for x86_64 - AppStream RPMs 9
3   | Red Hat Enterprise Linux 9 for x86_64 - BaseOS RPMs 9
2   | Red Hat Satellite Client 6 for RHEL 8 x86_64 RPMs
7   | Red Hat Satellite Client 6 for RHEL 9 x86_64 RPMs
----|---------------------------------------------------------------`

Here, we need repository IDs 9,6,5 and 2. Your ID list may be different.
`hammer content-view update --repository-ids 9,6,5,2 --name "cv-rhel8-prem-server" --organization "dar"`

We publish the repositories to the library.
`hammer content-view publish --name "cv-rhel8-prem-server" --organization "dar" --async`
