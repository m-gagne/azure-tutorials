# How to Load Balance Virtual Machines in Azure

For this we will use the current portal at [https://portal.azure.com](portal.azure.com) and the [Azure Resource Manager](https://azure.microsoft.com/en-us/documentation/articles/resource-group-overview/) deployment model (not the [classic model](https://azure.microsoft.com/en-us/documentation/articles/azure-classic-rm/)).

## Step 1: Create a Resource Group

[Resource Groups](https://azure.microsoft.com/en-us/documentation/articles/resource-group-portal/) allow you to group connected or like resources together to make it easier to manage, update, delete etc. Think of it as giving a name and a container for all components for a given project. This way you can find them much easier, see how much it's costing you and manage them all from one place.

1. From the [portal](https://portal.azure.com) click **+ New > Management > Resource group**
1. Provide a *Name*, *Subscription* & *Location*
1. Click **Create**

> Tip: Keep the name of this resource group and it's location handy, you will reuse it as you create your environment.

## Step 2: Create a Storage Account

[Storage Accounts](https://azure.microsoft.com/en-us/documentation/articles/storage-create-storage-account/) are like telling Azure "hey I want all storage related stuff like files, hard drives and more kept here please under this name". So, much like Resource Groups that are a container for all like services & infrastructure, a storage account is a container for all related storage. By default your storage account will be setup for LRS or Locally Redundant Storage. What this means is Azure will maintain 3 copies of your data at all times to ensure even in a failure of one or more drives your data is still intact.

1. From the [portal](https://portal.azure.com] click **+ New > Data + Storage > Storage Account**
1. Provide a *Name*
1. Ensure *Deployment model* is set to *Resouce Manager*
1. For *Performance*, standard is fine unless you need SSD performance
1. For *Replication* the default will keep 6 copies of your data (3 per DC).
    * Select the model that makes the most sence for your needs
1. For *Resource Group* select the Resource Group you previously created
  * Note: this will automatically update the location to match that of your Resource Group.
1. Click **Create**



## Step 3: Create a Virtual Network

[Virtual Networks](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-vnet-arm-pportal/) or VNet is a representation of a network in the cloud. This allows you to control your Azure network settings and define DHCP address blocks, DNS settings, security policies, routing and more.

1. From the portal click **+ New > Networking > Virtual Network**
1. Click **Create**
1. Provide a *Name*
  - You can change the address space or change the default subnet name from '*default*' to something more appropriate
1. For *Resource Group* select the Resource Group you previously created
1. The location will update to the same location as defined in your resource group
1. Click **Create**

## Step 4: Create a network security group

[Network Security Groups](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-nsg-arm-pportal/) control traffic to one or more virtual machines (VMs), role instances, network adapters (NICs), or subnets in your virtual network. A NSG (network security group) contains access control rules that allow or deny traffic based on traffic direction, protocol, source address and port, and destination address and port. 

1. From the portal click **+ New > Networking > Network security group**
1. Click **Create**
1. Provide a *Name* such as 'FrontEndSG'
1. For *Resource Group* select the Resource Group you previously created
1. Click **Create**

### Define Inbound security rules

1. Open your newly created security group
1. Click **Inbound security rules**
1. Click **+ Add**
1. Enter a *Name*, *Source Port Range* & *Destination Port Range*
  - e.g.
    - "HTTP", "*", "80"
    - "HTTPS", "*",  "443"
  - Why the source port range of '*'? This is because of the way Azure manages inbound connections from internal sources which can come from different ports.
1. Click **Ok**

## Step 4: Create the VMs

1. From the portal click **+ New > Virtual Machines > ** and select the image type (Windows, Linux etc.)
1. Click **Create**
1. Enter the *basic settings* & click **OK**
  - For *Resource Group* select the Resource Group you previously created
1. Select a *Size* that's appropriate for your needs & click **Select**
1. For *Settings*
  - *Storage account*: select the previously created storage account
  - *Network*: select your previously created virtual network 
  - *Subnet*: select your previously created subnet
  - *Public IP address*: by default Azure will assign you a new dynamic public IP. If you need a static one (one that does not change) you can configure that here
  - *Network security group*: select your previously created NSG
  - ***Availability Set***
      - If you plan to load balance your VMs you will need to create an *availability set*. If this is the first VM click **availability set > Create new**
        - Provide a name such as 'FrontEndAS'
        - The default of 5 update domains & 3 fault domains is fine.
      - Click **OK**
      - For 2nd, 3rd etc. VMs simply select this Availability Set
1. Click **OK** & **OK** again to create the VM

Repeat the above as needed (for a second, third, fourth VM etc.), also you will need to setup your web server/web app/etc. as needed.

## Step 5: Create a load balancer (assuming public internet based load balancing)

Unfortunately at this time the [public documentation](https://azure.microsoft.com/en-us/documentation/articles/load-balancer-get-started-internet-arm-ps/) for setting up a load balancer only includes PowerShell, CLI & ARM Tempaltes. The steps below cover a basic web based (HTTP & HTTPS) setup.

1. Click **+ New > Networking > Load Balancer**
1. Supply a `name`
1. Select an **Public** for  `Scheme`
1. Choose a `Public IP Address` or create a new one, I would recommend a static one (one that does not change)
1. For *Resource Group* select the Resource Group you previously created
1. Click **Create**


### Setup a backend pool

The backend pool is the VMs you want load balanced. This can be configured after the LB has been created.

1. Open your newly created LB (you will find it in your resource group)
1. Navigate to the **Settings** blade
1. Click **Backend pools** then click **+ Add**
1. Enter a *Name* for this pool e.g. "ProdWebServers"
1. Click **+ Add a virtual machine"
1. Click **Choose an availability set**
1. Click on your previously created availability set (or the set that contains the VMs you wish to load balance)
1. Click **Choose the virtual machines**
1. Click on each VM you wish to balance then click **Select**
1. Click **OK**
1. Close the Backend address pools blade

### Setup a health check probe
1. Under **Settings**, click **Probes** then **+ Add**
1. Enter a *Name* for this probe e.g. "HealthProbe"
1. For *Path* enter an appropriate path or keep / to simply ping the default page
1. Click **OK**
1. Close the Probes blade

### Setup load balancing rules

1. Under **Settings**, click **Load balancing rules** then **+ Add**
1. Enter a *Name* for this rule e.g. "HTTP"
1. For port 80 (standard port for non-secure web traffic) leave the defaults for Port & Backend port
1. Note: By default the load balancer will distribute traffic in a round-robin fashion, you can change this by selecting a change to Session Persistence  
1. Click **OK**

Repeat the above for port 443 (SSL)


### Optional: Setup a Static IP & DNS Name for your load balancer

1. Under **Settings** for your Load Balancer, click the **Public IP address** (note: this is found in the **Essentials** panel, NOT in the settings blade)
1. Click **Configuration**
1. Change *Assignment* to **Static**
1. Enter a DNS name
1. Click **Save**

## Conclusion

You should now have a fully load balanced setup. I find creating test pages with test like "Server 1", "Server 2" or a page that displays the local IP handy to verify that the various VMs are truly being load balanced.