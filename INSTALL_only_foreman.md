# Install Foreman on Hyper-V

## System Requirements 
The following requirements apply to the networked base operating system: [[1]](#1)

-   x86\_64 architecture
    
-   4-core 2.0 GHz CPU at a minimum
    
-   A minimum of 4 GB RAM is required for Foreman server to function. Foreman running with less RAM than the minimum value might not operate correctly.
    
-   Administrative user (root) access
    
-   Full forward and reverse DNS resolution using a fully-qualified domain name
    

Foreman only supports `UTF-8` encoding. If your territory is USA and your language is English, set `en_US.utf-8` as the system-wide locale settings. For more information about configuring system locale in Enterprise Linux, see [Configuring System Locale guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_basic_system_settings/assembly_changing-basic-environment-settings_configuring-basic-system-settings#proc_configuring-the-system-locale_assembly_changing-basic-environment-settings).

Foreman server and Smart Proxy server do not support shortnames in the hostnames. When using custom certificates, the Common Name (CN) of the custom certificate must be a fully qualified domain name (FQDN) instead of a shortname. This does not apply to the clients of a Foreman.

Before you install Foreman server, ensure that your environment meets the requirements for installation.

Foreman server must be installed on a freshly provisioned system that serves no other function except to run Foreman server. The freshly provisioned system must not have the following users provided by external identity providers to avoid conflicts with the local users that Foreman server creates:

-   apache
    
-   foreman
    
-   foreman-proxy
    
-   postgres
    
-   puppet
    
-   redis


### References
* <a id="1">[1]</a>: https://docs.theforeman.org/nightly/Installing_Server/index-foreman-el.html#system-requirements_foreman
