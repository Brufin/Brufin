### TheHive Configuration

TheHive configuration was completed in `/etc/thehive/application.conf`.

Key settings included:

- Cassandra host address
- Cassandra cluster name
- Elasticsearch host address
- `application.baseUrl`

### Service Ownership

Permissions were updated so that the `thehive` user owned the relevant application directory.

```bash
chown -R thehive:thehive /opt/thp/thehive
```

## Prerequisites

Before setting up this lab, it helps to have a basic understanding of Windows and Linux administration, virtual machines, and cloud-hosted servers. Familiarity with networking concepts such as IP addressing, firewall rules, and service exposure is also useful, since the lab requires configuring communication between the endpoint, SIEM, and incident response systems.

You will also need a system capable of running VirtualBox and a Windows 11 virtual machine, along with access to a cloud environment such as Azure/AWS/Google Cloud for hosting the Linux-based security infrastructure.

### Wazuh Server Deployment

Provisioned an Ubuntu 24.04 LTS server on Azure with the following specifications to host the Wazuh server component:

- **vCPUs:** 2
- **RAM:** 8 GB
- **Disk Size:** 64 GB

The official Wazuh documentation recommends 4 vCPUs for environments with 1–25 agents, but these specifications are sufficient for this basic lab setup.

Allowed inbound SSH traffic so I could connect to the server after deployment. I then connected to the Ubuntu server over SSH using PuTTY and continued with the server-side setup.

Once on the server, I downloaded and ran the Wazuh installation assistant:

```BASH
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

After the installation completed, the Wazuh installer generated the credentials required to access the web portal. Since only port 22 was open for SSH access, I added an inbound network security rule to allow HTTPS traffic to the server. After updating the rule, I was able to reach the Wazuh web interface and sign in with the provided credentials.

![Wazuh_Dashboard.png](projects\SIEM_Pipeline_Lab\img\Wazuh_Dashboard.png)

### Endpoint Deployment

For the endpoint deployment, I downloaded the Windows 11 ISO and verified its hash to confirm the file was safe and unaltered. Using the verified ISO, I created a virtual machine in VirtualBox and completed the full installation process, selecting Windows 11 Pro as the operating system. The VM was configured with 2 vCPUs, 4 GB of RAM, and 64 GB of storage, which provides enough resources for a basic Windows 11 test environment.

To access the windows instance using a local account I did the following:

- Headed to Network setting on the VM on VirtualBox and toggle the "Virtual Cable Connected" off
- Then using `Shift + F10` opened a terminal and executed the command `oobe\bypassnro`, this will trigger a restart
- After the quick restart I walked through the set up process of the Windows 11 instance and on the Network settings it gave me the option to proceed without internet thanks to the command I ran.

Following these steps I was able to bypass login in with a Microsoft account and was able to use a local account instead.

Once I was on logged in I turned the "Virtual Cable Connected" option on and downloaded Sysmon from Microsoft's official site. Before installing, I grabbed a Sysmon config file from https://github.com/olafhartong/sysmon-modular. and then ran the following command from using an Admin terminal on the Sysmon directory:

```Powershell
.\Sysmon64.exe -i .\sysmonconfig.txt
```

I then checked the services on the host and verified that Sysmon was running successfully.

### Deploying Agent

Next, I logged in to the Wazuh web portal from the Windows VM and began the agent deployment process. I selected Windows 11 as the endpoint operating system, specified the Wazuh server’s public IP address as the manager, and assigned the agent the name `Host-01`. Wazuh then generated a PowerShell command to install the agent:

```Powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.6-1.msi -OutFile $env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER=<public_srv_ip> WAZUH_AGENT_NAME='Host-01'
```

Once the installation was completed I ran the following command to start the agent:

```Powershell
NET START Wazuh
```

Finally, I created inbound network security rules to allow traffic on ports 1515 and 1514:

- _1515_: The agent enrollment service, used to enroll Wazuh agents to the Wazuh manager
- _1514_: The agent connection service, listens for events from Wazuh agents to establish and maintain a persistent and secure communication channel.

Once the rules were created, I checked the Wazuh manager and the agent was being manager

![Agent_on_Wazuh](projects\SIEM_Pipeline_Lab\img\Agent_on_Wazuh.png)

### Configure Agent

The Wazuh manager was monitoring and logging events from the endpoint, but it was not receiving the telemetry I wanted. On the Windows 11 host, I navigated to `C:\Program Files (x86)\ossec-agent`, where the `agent.conf` file was located. I created a backup of the file and then modified the configuration under the log analysis section to include Sysmon event collection:

```conf
<localfile>
	<location>Microsoft-Windows-Sysmon/Operational</location>
	<log_format>eventchannel</log_format>
</localfile>
```

After saving the configuration changes, I restarted the Wazuh service on the host for the updates to take effect. I headed to the Wazuh web page, navigated to "Discover" and verified that the Wazuh manager was collecting Sysmon telemetry.

![Sysmon_Event_Wazuh](projects\SIEM_Pipeline_Lab\img\Sysmon_Event_Wazuh.png)

### TheHive Deployment

Provisioned an Ubuntu 24.04 LTS server on Azure with the following specifications to host TheHive:

- vCPUs: 4
- RAM: 16 GB
- Disk Size: 64 GB

After connecting to the server via SSH, I began the deployment of TheHive. This included installing the required components on the server:

- Java Virtual Machine.
- Cassandra.
- Elasticsearch.
- TheHive.

I followed the official Linux installation guide for Ubuntu for the installation portion of the setup: [TheHive installation guide](https://docs.strangebee.com/thehive/installation/installation-guide-linux-standalone-server/).

Once the required components were installed, I configured them to work together.

## Cassandra Configuration

Cassandra was configured by editing `/etc/cassandra/cassandra.yaml`. The following settings were reviewed and adjusted:

- `cluster_name`: Name of the Cassandra cluster.
- `listen_address`: Address or interface Cassandra binds to for other nodes to connect; set to `10.0.0.5` (private IP).
- `rpc_address`: Address or interface for the native transport server; set to `10.0.0.5`.
- `seed_providers`: Addresses of hosts that are deemed contact points `10.0.0.5:7000`.

Next, I stopped the service, removed existing data, and restarted it:

```BASH
systemctl stop cassandra
rm -rf /var/lib/cassandra/*
systemctl start cassandra
```

## Elasticsearch Configuration

Elasticsearch was configured by editing `/etc/elasticsearch/elasticsearch.yml`. The following settings were reviewed and adjusted:

- `cluster.name`: Set to `my-application`.
- `node.name`: Set to `TheHive-srv`.
- `network.host`: Set to `10.0.0.5`.
- `http.port`: Elasticsearch listens for HTTP traffic on port `9200` by default.

To allow Elasticsearch to communicate with TheHive over HTTP in this lab environment, the following security settings were disabled:

```yaml
xpack.security.enabled: false
xpack.security.enrollment.enabled: false
xpack.security.http.ssl:
  enabled: false
```

This configuration is not recommended for production or corporate environments, as it reduces security, but it was acceptable for this lab setup.

Once configured, I started and enabled the service:

```BASH
systemctl start elasticsearch
systemctl enable elasticsearch
```

## TheHive Configuration

First, I navigated to `/opt/thp`, where the `thehive` directory is located. This directory stores files related to the TheHive application, including configuration files, data, and indexes. I then updated the ownership so that only the `thehive` user could read and write the files:

```BASH
chown -R thehive:thehive /opt/thp/thehive
```

TheHive was then configured in `/etc/thehive/application.conf`. The following settings were reviewed and adjusted:

- `hostname` for Cassandra: set to the private IP address of the Cassandra host.
- `cluster_name`: set to the Cassandra cluster name.
- `hostname` for Elasticsearch: set to the private IP address of the Elasticsearch host.
- `application.baseUrl`: set to `http://<public_ip>:9000`.

Lastly, I allowed inbound TCP traffic on port 9000 so the application could be accessed externally.

![TheHive](projects\SIEM_Pipeline_Lab\img\TheHive_Login.png)
