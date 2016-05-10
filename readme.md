# About

[Let's Encrypt](https://letsencrypt.org/) is an [ACME](https://datatracker.ietf.org/wg/acme/documents/) certificate authority. ACME allows for automated certificate provisioning and renewal. This requires certain verification methods to be in place. One of those is DNS-01 in which case the domain in question is verified by setting up TXT records for your domain.

**letsencrypt-azure** is a DNS-01 hook for the 3rd party [letsencrypt.sh](https://github.com/lukas2511/letsencrypt.sh) client. It allows for obtaining and/or renewing certificates for domains managed by [Azure DNS](https://azure.microsoft.com/en-us/services/dns/).

You **need** to perform a few tasks by hand to prepare your environment correctly - this is not an _it just works_ application, so please follow this guide before attempting to use it, otherwise the attempt **will** fail.

# DNS Zones

Azure DNS is available via the [Azure Resource Manager](https://azure.microsoft.com/en-us/documentation/articles/resource-group-overview/) (ARM). This means that all DNS zones you host on Azure are part of a [Resource Group](https://azure.microsoft.com/en-us/documentation/articles/resource-group-overview/#resource-groups). If you haven't done so yet, create a Resource Group for your DNS zones in Azure, then copy over the records from your previous DNS server. For more details on how to do this, please refer to [Manage DNS records and record sets using the Azure portal](https://azure.microsoft.com/en-us/documentation/articles/dns-operations-recordsets-portal/). 

Once done, modify your NS records to point to the Azure DNS servers. This is usually done via the web interface of your DNS registrar. When finished, verify with **nslookup** that your records are indeed served by Azure. Example:

~~~
C:\Users\user>nslookup
Standardserver:  resolver1.opendns.com
Address:  208.67.222.222

> set type=ns
> foobar.com
Server:  resolver1.opendns.com
Address:  208.67.222.222

Nicht autorisierende Antwort:
foobar.com      nameserver = ns1-01.azure-dns.com
foobar.com      nameserver = ns2-01.azure-dns.net
foobar.com      nameserver = ns3-01.azure-dns.org
foobar.com      nameserver = ns4-01.azure-dns.info
~~~

# Azure App

You cannot access Azure resources directly so you need to add an Azure AD (AAD) application to your directory. You'll need **both** the classic and the new Azure portal to do this.

## Classic Portal

On the **[classic portal](https://manage.windowsazure.com/)**, open your directory, select the **Applications** tab and click **Add**:

- **Add an application my organization is developing**
- Name: **letsencrypt-auto**
- **Native Client Application**
- Redirect URI: unused, thus can be anything e.g. company website address

Take note of the **Client ID**.

Permissions to other applications:

 - add **Windows Azure Service Management API / Access Azure Service Management as organization**

## New Portal

Now open the **[new portal](https://portal.azure.com/)**:

 - Select the resource group which holds your DNS zones
 - **Users / Add**
 - Select a role: **DNS Zone Contributor**
 - Select **letsencrypt-auto**

# Usage

letsencrypt-azure needs **JRE 1.7** or later to work. To download letsencrypt.sh and letsencrypt-azure:

~~~
git clone https://github.com/lukas2511/letsencrypt.sh.git
cd letsencrypt.sh
wget https://remedian.vault-tec.info/letsencrypt-azure.zip
unzip letsencrypt-azure.zip
~~~

TODO

~~~
echo 'your.domain.name' > domains.txt
./letsencrypt.sh --cron --hook letsencrypt-azure/letsencrypt-azure.sh --challenge dns-01
~~~
