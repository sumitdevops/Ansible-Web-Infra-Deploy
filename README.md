# Use Ansible to Install and Deploy Web Application Infrastructure Components

Ansible is a simple agentless automation technology that has caught the world of software configuration management by storm!  Ansible allows IT operators to automate repetitive infrastructure tasks such as installation, configuration and deployment of software using a simple yet powerful automation language *YAML*.  Ansible is an open source project and is the core automation engine powering Red Hat Ansible Tower.  Ansible provides close to 800+ (at the time of this writing) pre-built modules that can be used to automate complex infrastructure tasks ranging from web/application deployment to configuration management to workflow orchestration.

**Important Notes:** 
1.  This project assumes readers are familiar with writing simple Ansible automation scripts using the YAML language and/or have prior experience working with other IT infrastructure automation tools such as Chef and Puppet.
2.  This project was tested with Ansible v2.2.1 and Red Hat JBoss Web Server v3.1.  The provided Ansible scripts can be easily modified to deploy vanilla (Apache community bits) Apache Web/Httpd and Apache Tomcat 7/8 servers if needed.
3.  Apache Web/Httpd Server can be configured to use either 'mod_jk' or 'mod_cluster' load balancer plugins.  The Apache Httpd Server bundled with JBoss Core Services Collection is pre-configured to use 'mod_cluster' as the load balancer by default.  Therefore, the Apache Web and Tomcat servers deployed by the Ansible playbooks in this project will use 'mod_cluster' as the load balancer.

For easy and quick reference, readers can refer to the following on-line resources as needed.

1.  [Open Source Ansible](https://www.ansible.com/)
2.  [Red Hat Ansible / Tower](https://www.redhat.com/en/technologies/management/ansible)
3.  [Ansible Documentation](https://docs.ansible.com/)
4.  [Red Hat JBoss Web Server](https://access.redhat.com/documentation/en/red-hat-jboss-web-server/)
5.  [Red Hat JBoss Core Services](https://access.redhat.com/documentation/en/red-hat-jboss-core-services/)
6.  [Apache Httpd Server](https://httpd.apache.org/)
7.  [Apache Tomcat 8 Application Server](https://tomcat.apache.org/tomcat-8.0-doc/index.html)
8.  [YAML](http://yaml.org/)
9.  [mod_cluster Apache Httpd Load Balancer](http://mod-cluster.jboss.org/)

## Description
This project provides Ansible automation scripts for implementing a multi-server (host) distributed web application deployment architecture.  Typically, in a 3-tier web application deployment architecture, web / proxy servers are deployed within tier 1.  These servers act as a reverse proxy and include a load balancer component which intelligently distributes the incoming http requests to backend application server instances running on multiple machines.  Tier 2 infrastructure components in this topology comprise of application server (Web/JEE) instances that execute web/enterprise applications.  And lastly, tier 3 components include (but are not limited to) database servers, Web/Rest services and/or backend/legacy applications.  In this project, we will be using Ansible to automate the installation, configuration and deployment of web infrastructure components within tiers 1 and 2.  The provided Ansible scripts can be easily adapted to deploy tier-3 components such as database servers with minimal effort.

For this project, the widely popular and battle tested *Apache Web/Httpd Server* will be deployed in tier 1.  And for tier 2, the tried and true *Apache Tomcat Server* will be used as the backend Java/Web application server.  Both of these web application infrastructure components are actively supported by the respective Apache open source communities and have stood the test of time.  Here are some cool facts 

1.  Apache Web (Httpd) Server currently powers approximately 70% of the web sites on the internet.  
2.  Apache Tomcat has close to 70% penetration in the enterprise data center.

Red Hat Inc., provides tested, certified and supported versions of the two most trusted web application server components (runtimes) listed above.  *Red Hat JBoss Web Server* (JWS) is a software bundle which includes the following *Open Source* components/libraries:

1.  Apache Web/Httpd Server (v2.4.x)
2.  Apache Tomcat application servers (v7.x & v8.x)
3.  Apache Portable Runtime (APR) native library
4.  Load balancer plugins - mod_jk and mod_cluster.  (In comparison to 'mod_jk', 'mod_cluster' requires minimal configuration and is an intelligent load balancer.  Refer to the mod_cluster project website for more details).

## Configure and Deploy the Web Application Infrastructure components
The steps outlined in the following sections (below) explain in detail how to configure the provided Ansible scripts in order to deploy a farm of Apache Web/Httpd and Apache Tomcat servers across multiple hosts/machines.

### A] Pre-Requisities
1.  Ansible must be installed on the *'Controller'* machine.  This is the machine where the Ansible engine runs the playbooks. Ansible is an agentless automation engine and therefore no Ansible components are required to be present on the target hosts/machines.  Software configuration tasks will be executed on the target machines via SSH.
2.  Ansible uses 'SSH' (Secure Shell) protocol to securely login into remote machines and execute software configuration tasks.  Therefore, ssh keys (private and public key) should be generated and configured on both a) The Ansible Controller host and  b) The target machines where the web infrastructure components will be deployed.
3.  Clone this git repository into a local directory on your Ansible *Controller* host.  Ensure the Linux user you will be using to run the Ansible playbooks has *execute* permissions for this directory and all sub-directories (and files) below it. 

### B] Modify Ansible scripts to configure the Apache Web Server instance(s)
Review and modify the following files/scripts to meet your requirements.
1.  **hosts**: Under section '[web-servers], specify IP Addresses (or host aliases) for the hosts/machines where the Apache Web/HTTPD Server needs to be installed and deployed.  Each host IP address or alias should be specified in a separate line within this file.  If you are specifying host aliases (instead of IP addresses), then the DNS server should be able to resolve the host aliases to IP addresses correctly.  As a result of running the Ansible playbook, an instance of Apache Web/Httpd Server will be deployed on each one of the hosts specified in this file.
2.  **groups_vars/web-servers**: Review the variables and their default values in this YAML file and specify appropriate values to meet your requirements.  A comment line (prefixed with '#') is included above each variable to denote it's purpose.
3.  **roles/apacheHttpd/files**: Download the Red Hat JBoss Core Services Httpd Server binaries from the Red Hat customer support portal and drop the *zip* file into this directory.
4.  **roles/apacheHttpd/files/00-mpm.conf**: Modify the Apache Httpd pre-fork worker (or MPM process) parameters as needed to suit your requirements.  By default, the Apache Httpd Server process will run in 'pre-fork' mode. The Httpd server process will spawn one worker process upon booting up and spawn a maximum of 256 worker processes (under maximum load).  Refer to the Apache Httpd Server documentation (website) for further details.
5.  **roles/apacheHttpd/templates/httpd.conf**: This is the main Apache Httpd server configuration file.  Review this file and make changes as per your requirements (logging, ServerAdmin ....).
6.  **roles/apacheHttpd/templates/mod_cluster.conf**: This is the configuration file for mod_cluster load balancer.  Go thru the mod_cluster documentation website and make changes as necessary.
7.  **roles/apacheHttpd/tasks/main.yml**: This file contains the set of configuration tasks required to install, configure and deploy Apache Httpd Server on remote machines.  Ansible (engine) will read the task list sequentially and execute each task on each configured host/machine. 

### C] Modify Ansible scripts to configure the Apache Tomcat Server instance(s)
Review and modify the following files/scripts to meet your requirements.
1.  **hosts**: Under section '[tomcat-servers], specify IP Addresses (or host aliases) for the hosts/machines where Apache Tomcat Server binaries needs to be installed and deployed.  Each host IP address or alias should be specified in a separate line within this file.  If you are specifying host aliases (instead of IP addresses), then the DNS server should be able to resolve the host aliases to IP addresses correctly.  As a result of running the Ansible playbook, an instance of Apache Tomcat Server will be deployed on each one of the hosts specified in this file.
2.  **groups_vars/tomcat-servers**: Review the variables and their default values in this YAML file and specify appropriate values to meet your requirements.  A comment line (prefixed with '#') is included above each variable to denote it's purpose.
3.  **roles/tomcat/vars/main.yml**: Use this file to configure multiple Tomcat server instances on each host/machine.  By default, only one Apache Tomcat Server instance (node) will be deployed on each host.  For deploying multiple Tomcat instances per host, simply copy and paste the node definition (starting at *'node01'*) stanza to deploy as many instances per host as needed.  Remember to provide unique values for the node name and port variables.
4.  **roles/tomcat/files/webapps**: Drop web application *WAR* files in this directory.  All WAR files will be deployed to every Apache Tomcat instance on every host/machine.

### D] Run Ansible playbook to deploy the web infrastructure components

### E] (Optional) Run Ansible playbook to un-deploy web infrastructure components

