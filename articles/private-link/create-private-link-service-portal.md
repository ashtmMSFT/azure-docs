---
title: 'Quickstart - Create a Private Link service by using the Azure portal'
titleSuffix: Azure Private Link
description: Learn how to create a Private Link service by using the Azure portal in this quickstart
services: private-link
author: asudbring
# Customer intent: As someone with a basic network background who's new to Azure, I want to create an Azure Private Link service by using the Azure portal
ms.service: private-link
ms.topic: quickstart
ms.date: 08/18/2021
ms.author: allensu
ms.custom: mode-portal
---
# Quickstart: Create a Private Link service by using the Azure portal

Get started creating a Private Link service that refers to your service.  Give Private Link access to your service or resource deployed behind an Azure Standard Load Balancer.  Users of your service have private access from their virtual network.

## Prerequisites

* An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## Sign in to the Azure portal

Sign in to the Azure portal at https://portal.azure.com.

## Create an internal load balancer

In this section, you'll create a virtual network and an internal Azure Load Balancer.

### Virtual network

In this section, you create a virtual network and subnet to host the load balancer that accesses your Private Link service.

1. Sign in to the [Azure portal](https://portal.azure.com).

2. On the upper-left side of the screen, select **Create a resource > Networking > Virtual network** or search for **Virtual network** in the search box.

3. Select **Create**. 

4. In **Create virtual network**, enter or select this information in the **Basics** tab:

    | **Setting**          | **Value**                                                           |
    |------------------|-----------------------------------------------------------------|
    | **Project Details**  |                                                                 |
    | Subscription     | Select your Azure subscription                                  |
    | Resource Group   | Select **Create new**. Enter **CreatePrivLinkService-rg**. </br> Select **OK**. |
    | **Instance details** |                                                                 |
    | Name             | Enter **myVNet**                                    |
    | Region           | Select **(US) East US 2** |

5. Select the **IP Addresses** tab or select the **Next: IP Addresses** button at the bottom of the page.

6. In the **IP Addresses** tab, enter this information:

    | Setting            | Value                      |
    |--------------------|----------------------------|
    | IPv4 address space | Enter **10.1.0.0/16** |

7. Under **Subnet name**, select the word **default**.

8. In **Edit subnet**, enter this information:

    | Setting            | Value                      |
    |--------------------|----------------------------|
    | Subnet name | Enter **myBackendSubnet** |
    | Subnet address range | Enter **10.1.0.0/24** |

9. Select **Save**.

10. Select the **Review + create** tab or select the **Review + create** button.

11. Select **Create**.

### Create NAT gateway

In this section, you'll create a NAT gateway and assign it to the subnet in the virtual network you created previously.

1. On the upper-left side of the screen, select **Create a resource > Networking > NAT gateway** or search for **NAT gateway** in the search box.

2. Select **Create**. 

3. In **Create network address translation (NAT) gateway**, enter or select this information in the **Basics** tab:

    | **Setting**          | **Value**                                                           |
    |------------------|-----------------------------------------------------------------|
    | **Project Details**  |                                                                 |
    | Subscription     | Select your Azure subscription.                                  |
    | Resource Group   | Select **CreatePrivLinkService-rg**. |
    | **Instance details** |                                                                 |
    | Name             | Enter **myNATGateway**                                    |
    | Region           | Select **(US) East US 2**  |
    | Availability Zone | Select **None**. |
    | Idle timeout (minutes) | Enter **10**. |

4. Select the **Outbound IP** tab, or select the **Next: Outbound IP** button at the bottom of the page.

5. In the **Outbound IP** tab, enter or select the following information:

    | **Setting** | **Value** |
    | ----------- | --------- |
    | Public IP addresses | Select **Create a new public IP address**. </br> In **Name**, enter **myNATgatewayIP**. </br> Select **OK**. |

6. Select the **Subnet** tab, or select the **Next: Subnet** button at the bottom of the page.

7. In the **Subnet** tab, select **myVNet** in the **Virtual network** pull-down.

8. Check the box next to **myBackendSubnet**.

9. Select the **Review + create** tab, or select the blue **Review + create** button at the bottom of the page.

10. Select **Create**.

### Create load balancer

In this section, you create a load balancer that load balances virtual machines.

During the creation of the load balancer, you'll configure:

* Frontend IP address
* Backend pool
* Inbound load-balancing rules

1. In the search box at the top of the portal, enter **Load balancer**. Select **Load balancers** in the search results.

2. In the **Load balancer** page, select **Create**.

3. In the **Basics** tab of the **Create load balancer** page, enter, or select the following information: 

    | Setting                 | Value                                              |
    | ---                     | ---                                                |
    | **Project details** |   |
    | Subscription               | Select your subscription.    |    
    | Resource group         | Select **CreatePrivLinkService-rg**. |
    | **Instance details** |   |
    | Name                   | Enter **myLoadBalancer**                                   |
    | Region         | Select **(US) East US 2**.                                        |
    | Type          | Select **Internal**.                                        |
    | SKU           | Leave the default **Standard**. |

4. Select **Next: Frontend IP configuration** at the bottom of the page.

5. In **Frontend IP configuration**, select **+ Add a frontend IP**.

6. Enter **LoadBalancerFrontend** in **Name**.

7. Select **myBackendSubnet** in **Subnet**.

8. Select **Dynamic** for **Assignment**.

9. Select **Zone-redundant** in **Availability zone**.

    > [!NOTE]
    > In regions with [Availability Zones](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json#availability-zones), you have the option to select no-zone (default option), a specific zone, or zone-redundant. The choice will depend on your specific domain failure requirements. In regions without Availability Zones, this field won't appear. </br> For more information on availability zones, see [Availability zones overview](../availability-zones/az-overview.md).

10. Select **Add**.

11. Select **Next: Backend pools** at the bottom of the page.

12. In the **Backend pools** tab, select **+ Add a backend pool**.

13. Enter **myBackendPool** for **Name** in **Add backend pool**.

14. Select **NIC** or **IP Address** for **Backend Pool Configuration**.

15. Select **IPv4** or **IPv6** for **IP version**.

16. Select **Add**.

17. Select the **Next: Inbound rules** button at the bottom of the page.

18. In **Load balancing rule** in the **Inbound rules** tab, select **+ Add a load balancing rule**.

19. In **Add load balancing rule**, enter or select the following information:

    | Setting | Value |
    | ------- | ----- |
    | Name | Enter **myHTTPRule** |
    | IP Version | Select **IPv4** or **IPv6** depending on your requirements. |
    | Frontend IP address | Select **LoadBalancerFrontend**. |
    | Protocol | Select **TCP**. |
    | Port | Enter **80**. |
    | Backend port | Enter **80**. |
    | Backend pool | Select **myBackendPool**. |
    | Health probe | Select **Create new**. </br> In **Name**, enter **myHealthProbe**. </br> Select **HTTP** in **Protocol**. </br> Leave the rest of the defaults, and select **OK**. |
    | Session persistence | Select **None**. |
    | Idle timeout (minutes) | Enter or select **15**. |
    | TCP reset | Select **Enabled**. |
    | Floating IP | Select **Disabled**. |

20. Select **Add**.

21. Select the blue **Review + create** button at the bottom of the page.

22. Select **Create**.

## Create a private link service

In this section, you'll create a Private Link service behind a standard load balancer.

1. On the upper-left part of the page in the Azure portal, select **Create a resource**.

2. Search for **Private Link** in the **Search the Marketplace** box.

3. Select **Create**.

4. In **Overview** under **Private Link Center**, select the blue **Create private link service** button.

5. In the **Basics** tab under **Create private link service**, enter, or select the following information:

    | Setting | Value |
    | ------- | ----- |
    | **Project details** |  |
    | Subscription | Select your subscription. |
    | Resource Group | Select **CreatePrivLinkService-rg**. |
    | **Instance details** |  |
    | Name | Enter **myPrivateLinkService**. |
    | Region | Select **(US) East US 2**. |

6. Select the **Outbound settings** tab or select **Next: Outbound settings** at the bottom of the page.

7. In the **Outbound settings** tab, enter or select the following information:

    | Setting | Value |
    | ------- | ----- |
    | Load balancer | Select **myLoadBalancer**. |
    | Load balancer frontend IP address | Select **LoadBalancerFrontEnd (10.1.0.4)**. |
    | Source NAT subnet | Select **mySubnet (10.1.0.0/24)**. |
    | Enable TCP proxy V2 | Leave the default of **No**. </br> If your application expects a TCP proxy v2 header, select **Yes**. |
    | **Private IP address settings** |  |
    | Leave the default settings |  |

8. Select the **Access security** tab or select **Next: Access security** at the bottom of the page.

9. Leave the default of **Role-based access control only** in the **Access security** tab.

10. Select the **Tags** tab or select **Next: Tags** at the bottom of the page.

11. Select the **Review + create** tab or select **Next: Review + create** at the bottom of the page.

12. Select **Create** in the **Review + create** tab.

Your private link service is created and can receive traffic. If you want to see traffic flows, configure your application behind your standard load balancer.

## Create private endpoint

In this section, you'll map the private link service to a private endpoint. A virtual network contains the private endpoint for the private link service. This virtual network contains the resources that will access your private link service.

### Create private endpoint virtual network

1. On the upper-left side of the screen, select **Create a resource > Networking > Virtual network** or search for **Virtual network** in the search box.

2. In **Create virtual network**, enter or select this information in the **Basics** tab:

    | **Setting**          | **Value**                                                           |
    |------------------|-----------------------------------------------------------------|
    | **Project Details**  |                                                                 |
    | Subscription     | Select your Azure subscription                                  |
    | Resource Group   | Select **CreatePrivLinkService-rg** |
    | **Instance details** |                                                                 |
    | Name             | Enter **myVNetPE**                                    |
    | Region           | Select **(US) East US 2** |

3. Select the **IP Addresses** tab or select the **Next: IP Addresses** button at the bottom of the page.

4. In the **IP Addresses** tab, enter this information:

    | Setting            | Value                      |
    |--------------------|----------------------------|
    | IPv4 address space | Enter **11.1.0.0/16** |

5. Under **Subnet name**, select the word **default**.

6. In **Edit subnet**, enter this information:

    | Setting            | Value                      |
    |--------------------|----------------------------|
    | Subnet name | Enter **mySubnetPE** |
    | Subnet address range | Enter **11.1.0.0/24** |

7. Select **Save**.

8. Select the **Review + create** tab or select the **Review + create** button.

9. Select **Create**.

### Create private endpoint

1. On the upper-left side of the screen in the portal, select **Create a resource** > **Networking** > **Private Link**, or in the search box enter **Private Link**.

2. Select **Create**.

3. In **Private Link Center**, select **Private endpoints** in the left-hand menu.

4. In **Private endpoints**, select **+ Add**.

5. In the **Basics** tab of **Create a private endpoint**, enter, or select this information:

    | Setting | Value |
    | ------- | ----- |
    | **Project details** | |
    | Subscription | Select your subscription. |
    | Resource group | Select **CreatePrivLinkService-rg**. You created this resource group in the previous section.|
    | **Instance details** |  |
    | Name  | Enter **myPrivateEndpoint**. |
    | Region | Select **(US) East US 2**. |

6. Select the **Resource** tab or the **Next: Resource** button at the bottom of the page.
    
7. In **Resource**, enter or select this information:

    | Setting | Value |
    | ------- | ----- |
    | Connection method | Select **Connect to an Azure resource in my directory**. |
    | Subscription | Select your subscription. |
    | Resource type | Select **Microsoft.Network/privateLinkServices**. |
    | Resource | Select **myPrivateLinkService**. |

8. Select the **Configuration** tab or the **Next: Configuration** button at the bottom of the screen.

9. In **Configuration**, enter or select this information:

    | Setting | Value |
    | ------- | ----- |
    | **Networking** |  |
    | Virtual Network | Select **myVNetPE**. |
    | Subnet | Select **mySubnetPE**. |

10. Select the **Review + create** tab, or the **Review + create** button at the bottom of the screen.

11. Select **Create**.

### IP address of private endpoint

In this section, you'll find the IP address of the private endpoint that corresponds with the load balancer and private link service.

1. In the left-hand column of the Azure portal, select **Resource groups**.

2. Select the **CreatePrivLinkService-rg** resource group.

3. In the **CreatePrivLinkService-rg** resource group, select **myPrivateEndpoint**.

4. In the **Overview** page of **myPrivateEndpoint**, select the name of the network interface associated with the private endpoint.  The network interface name begins with **myPrivateEndpoint.nic**.

5. In the **Overview** page of the private endpoint nic, the IP address of the endpoint is displayed in **Private IP address**.

## Clean up resources

When you're done using the private link service, delete the resource group to clean up the resources used in this quickstart.

1. Enter **CreatePrivLinkService-rg** in the search box at the top of the portal, and select **CreatePrivLinkService-rg** from the search results.

2. Select **Delete resource group**.

3. In **TYPE THE RESOURCE GROUP NAME**, enter **CreatePrivLinkService-rg**.

4. Select **Delete**.

## Next steps

In this quickstart, you:

* Created a virtual network and internal Azure Load Balancer.
* Created a private link service.
* Created a virtual network and a private endpoint for the private link service.

To learn more about Azure Private endpoint, continue to:
> [!div class="nextstepaction"]
> [Quickstart: Create a Private Endpoint using the Azure portal](create-private-endpoint-portal.md)
