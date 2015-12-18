<properties 
   pageTitle="连接到 Azure 上的 SQL Server 虚拟机"
   description="本主题介绍如何连接到 Azure 中虚拟机上运行的 SQL Server。方案根据网络配置和客户端位置的不同而异。"
   services="virtual-machines"
   documentationCenter="na"
   authors="rothja"
   manager="jeffreyg"
   editor="monicar" />
<tags 
   ms.service="virtual-machines"
   ms.date="08/18/2015"
   wacn.date="09/18/2015" />

# 连接到 Azure 上的 SQL Server 虚拟机

## 概述

连接到 Azure 虚拟机上运行的 SQL Server 与连接到本地 SQL Server 所要执行的配置步骤差别不大。你仍然需要完成涉及防火墙、身份验证和数据库登录的配置步骤。

但在 SQL Server 连接方面，有一些 Azure VM 特定的设置。本文将介绍一些[常规连接方案](#connection-scenarios)，并提供[在 Azure VM 中配置 SQL Server 连接的详细步骤](#steps-for-configuring-sql-server-connectivity-in-an-azure-vm)。

>[AZURE.NOTE]本文的重点在于连接。关于预配和连接的完整演练，请参阅[在 Azure 上预配 SQL Server 虚拟机](/documentation/articles/virtual-machines-provision-sql-server)。

## 连接方案

客户端连接虚拟机上运行的 SQL Server 的方式取决于客户端的位置与计算机/网络配置。这些方案包括：

- [连接到同一云服务中的 SQL Server](#connect-to-sql-server-in-the-same-cloud-service)
- [通过 Internet 连接到 SQL Server](#connect-to-sql-server-over-the-internet)
- [连接到同一虚拟网络中的 SQL Server](#connect-to-sql-server-in-the-same-virtual-network)

### 连接到同一云服务中的 SQL Server

可以在同一云服务中创建多个虚拟机。若要了解此虚拟机方案，请参阅[如何将虚拟机连接到虚拟网络或云服务](/documentation/articles/cloud-services-connect-virtual-machine)。

首先，根据[此文章中的步骤来配置连接](#steps-for-configuring-sql-server-connectivity-in-an-azure-vm)。请注意，如果要连接同一云服务中的计算机，则不需要设置公共终结点。

可以在客户端连接字符串中使用 VM **主机名**。主机名是你在创建 VM 期间为 VM 指定的名称。例如，如果 SQL VM 名为 **mysqlvm**，云服务 DNS 名称为 **mycloudservice.cloudapp.net**，则同一云服务中的客户端 VM 可以使用以下连接字符串进行连接：

	"Server=mysqlvm;Integrated Security=false;User ID=<login_name>;Password=<your_password>"

### 通过 Internet 连接到 SQL Server

如果你想要通过 Internet 连接到 SQL Server 数据库引擎，则必须创建虚拟机终结点以进行传入 TCP 通信。此 Azure 配置步骤将传入 TCP 端口通信定向到虚拟机可以访问的 TCP 端口。

首先，根据[此文章中的步骤来配置连接](#steps-for-configuring-sql-server-connectivity-in-an-azure-vm)。然后，可访问 Internet 访问的任何客户端可通过指定云服务 DNS 名称（例如 **mycloudservice.cloudapp.net**）和 VM 终结点（例如 **57500**）来连接到 SQL Server 实例。

	"Server=mycloudservice.cloudapp.net,57500;Integrated Security=false;User ID=<login_name>;Password=<your_password>"

尽管客户端可通过 Internet 进行连接，但这并不意味着任何人都可以连接到 SQL Server。外部客户端必须有正确的用户名和密码。为了提高安全性，请不要对公共虚拟机终结点使用常用的 1433 端口。如果可能，请考虑在终结点上添加 ACL 以将流量限制为你允许的客户端。有关在终结点上使用 ACL 的说明，请参阅[管理终结点上的 ACL](/documentation/articles/virtual-machines-set-up-endpoints#manage-the-acl-on-an-endpoint)。

>[AZURE.NOTE]请务必注意，使用这种方法与 SQL Server 进行通信时，所有返回的数据被将视为数据中心的传出流量。这需要按一般的[出站数据传输价格](/pricing/details/data-transfer/)付费。即使在同一 Azure 数据中心内的另一个计算机或云服务中使用这种方法，也是如此，因为流量还是会通过 Azure 的公共负载均衡器。

### 连接到同一虚拟网络中的 SQL Server

[虚拟网络](/documentation/articles/virtual-networks-overview)支持其他方案。你可以连接同一虚拟网络中的 VM，即使这些 VM 位于不同的云服务中。使用[站点到站点 VPN](/documentation/articles/vpn-gateway-site-to-site-create)，可以创建连接 VM 与本地网络和计算机的混合体系结构。

虚拟网络还可让你将 Azure VM 加入域。这是对 SQL Server 使用 Windows 身份验证的唯一方式。其他连接方案需要使用用户名和密码进行 SQL 身份验证。

首先，根据[此文章中的步骤来配置连接](#steps-for-configuring-sql-server-connectivity-in-an-azure-vm)。如果你要配置域环境和 Windows 身份验证，则不需要使用本文中的步骤来配置 SQL 身份验证和登录。此外，在此方案中，公共终结点不是必需的。

假设你已配置 DNS，你可以在连接字符串中指定 SQL Server VM 的主机名来连接 SQL Server 实例。以下示例假设同时已配置 Windows 身份验证，并且用户已获得访问 SQL Server 实例的权限。

	"Server=mysqlvm;Integrated Security=true" 

请注意，在此方案中，你还可以指定 VM 的 IP 地址。

## 在 Azure VM 中配置 SQL Server 连接的步骤

[AZURE.INCLUDE [连接到 VM 中的 SQL Server](../includes/virtual-machines-sql-server-connection-steps.md)]

## 后续步骤

若要查看预配说明以及这些连接步骤，请参阅[在 Azure 上预配 SQL Server 虚拟机](/documentation/articles/virtual-machines-provision-sql-server)。

如果你还打算针对高可用性和灾难恢复使用 AlwaysOn 可用性组，你应该考虑实施侦听器。数据库客户端将连接到侦听器，而不是直接连接到一个 SQL Server 实例。侦听器将客户端路由到可用性组中的主副本。有关详细信息，请参阅[在 Azure 中配置 AlwaysOn 可用性组的 ILB 侦听器](/documentation/articles/virtual-machines-sql-server-configure-ilb-alwayson-availability-group-listener)。

请务必查看 Azure 虚拟机上运行的 SQL Server 的所有安全最佳实践。有关详细信息，请参阅 [Azure 虚拟机中 SQL Server 的安全注意事项](https://msdn.microsoft.com/zh-cn/library/azure/dn133147.aspx)。

有关其他与在 Azure VM 中运行 SQL Server 相关的主题，请参阅 [Azure 虚拟机上的 SQL Server](/documentation/articles/virtual-machines-sql-server-infrastructure-services)。

<!---HONumber=70-->