<properties
	pageTitle="无法通过 SSH 连接到 Azure VM | Windows Azure"
	description="对运行 Linux 的 Azure 虚拟机的 Secure Shell (SSH) 连接进行故障排除。"
	services="virtual-machines"
	documentationCenter=""
	authors="dsk-2015"
	manager="timlt"
	editor=""
	tags="top-support-issue,azure-service-management,azure-resource-manager"/>

<tags
	ms.service="virtual-machines"
	ms.date="10/05/2015"
	wacn.date="11/27/2015"/>

# 对于基于 Linux 的 Azure 虚拟机的 Secure Shell (SSH) 连接进行故障排除

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]

有许多原因可能会导致基于 Linux 的 Azure 虚拟机 SSH 失败。本文将帮助你找出原因并予以更正。

> [AZURE.NOTE]本文仅适用于运行 Linux 的 Azure 虚拟机。有关对运行 Windows 的 Azure 虚拟机的连接进行故障排除，请参阅[此文](/documentation/articles/virtual-machines-troubleshoot-remote-desktop-connections)。

## 与 Azure 客户支持联系

如果你对本文中的任何观点存在疑问，可以联系 [MSDN Azure 和 CSDN 论坛](/support/forums/)上的 Azure 专家。

或者，你也可以发起 Azure 支持事件。请转到 [Azure 支持站点](/support/contact/)并单击“获取支持”。有关使用 Azure 支持的信息，请阅读 [Windows Azure 支持常见问题](/support/faq/)。


## 基本步骤 - 经典部署模型

若要解决使用经典部署模型创建的虚拟机中较常见的 SSH 连接失败，请尝试以下步骤：

1. 从 [Azure 门户](https://manage.windowsazure.cn)**重置远程访问**。单击“浏览全部”>“虚拟机(经典)”> 你的 Windows 虚拟机 >“重置远程访问”。

2. **重新启动**虚拟机。在 [Azure 门户](https://manage.windowsazure.cn)中，单击“全部浏览”>“虚拟机(经典)”> 你的 Windows 虚拟机 >“重新启动”。在 [Azure 管理门户](https://manage.windowsazure.com)中，打开“虚拟机”>“实例”，然后单击“重新启动”。

3. [**调整**虚拟机的大小](https://msdn.microsoft.com/zh-cn/library/dn168976.aspx)。

4. 按照<!--[-->如何为基于 Linux 的虚拟机重置密码或 SSH<!--](/documentation/articles/virtual-machines-linux-use-vmaccess-reset-password-or-ssh) -->中的说明，在虚拟机上执行以下操作：

	- 重置密码或 SSH 密钥。
	- 创建新的 sudo 用户帐户。
	- 重置 SSH 配置。


## 基本步骤 - 资源管理器部署模型

若要解决使用资源管理器部署模型创建的虚拟机的常见 SSH 问题，请尝试以下步骤。

1. 使用 Azure CLI 或 Azure PowerShell，在命令行上为 Linux VM **重置 SSH 连接**。确保已安装 <!--[-->Microsoft Azure Linux Agent<!--](virtual-machines-linux-agent-user-guide.md)--> 2.0.5 或更高版本。

	**使用 Azure CLI**

	a.如果尚未安装 Azure CLI，请使用 `azure login` 命令[安装 Azure CLI 并连接到 Azure 订阅](/documentation/articles/xplat-cli-install)。

	b.切换到资源管理器模式。

	```
	azure config mode arm
	```

	c.使用以下方法之一重置 SSH 连接。

	* 如以下示例所示使用 `vm reset-access` 命令。

	```
	azure vm reset-access -g TestRgV2 -n TestVmV2 -r
	```

	这将在虚拟机上安装 `VMAccessForLinux` 扩展。

	* 或者，使用以下内容创建名为 PrivateConf.json 的文件：

	```
	{  
	"reset_ssh":"True"
	}
	```

	然后手动运行 `VMAccessForLinux` 扩展以重置 SSH 连接。

	```
	azure vm extension set "testRG" "testVM" VMAccessForLinux Microsoft.OSTCExtensions "1.2" --private-config-path PrivateConf.json
	```

	**使用 Azure PowerShell**

	a.如果尚未安装 Azure PowerShell，请使用 Azure AD 方法[安装 Azure PowerShell 并连接到 Azure 订阅](/documentation/articles/powershell-install-configure)。

	b.切换到资源管理器模式。

	```
	Switch-AzureMode -Name AzureResourceManager
	```

	c.如以下示例所示，运行 `VMAccessForLinux` 扩展以重置 SSH 连接。

	```
	Set-AzureVMExtension -ResourceGroupName "testRG" -VMName "testVM" -Location "West US" -Name "VMAccessForLinux" -Publisher "Microsoft.OSTCExtensions" -ExtensionType "VMAccessForLinux" -TypeHandlerVersion "1.2" -SettingString "{}" -ProtectedSettingString '{"reset_ssh":true}'
	```

2. 从门户**重新启动** Linux VM。在 [Azure 门户](https://manage.windowsazure.cn)中，单击“全部浏览”>“虚拟机”> 你的 Windows 虚拟机 >“重新启动”。

3. 使用 Azure CLI 或 Azure PowerShell，在命令行上为 Linux VM **重置密码或 SSH 密钥**。你还可以使用 sudo 授权创建新的用户名和密码，如以下示例所示。

	**使用 Azure CLI**

	如上所述安装并配置 Azure CLI。切换到资源管理器模式，然后使用以下方法之一运行扩展。

	* 运行 `vm reset-access` 命令以设置任何 SSH 凭据。

	```
	azure vm reset-access TestRgV2 TestVmV2 -u NewUser -p NewPassword
	```

	在命令行上键入 `azure vm reset-access -h` 可以查看有关此命令的详细信息。

	* 或者，使用以下内容创建名为 PrivateConf.json 的文件。
	```
	{
	"username":"NewUsername", "password":"NewPassword", "expiration":"2016-01-01", "ssh_key":"", "reset_ssh":false, "remove_user":""
	}
	```

	然后使用上述文件运行 Linux 扩展。

	```
	$azure vm extension set "testRG" "testVM" VMAccessForLinux Microsoft.OSTCExtensions "1.2" --private-config-path PrivateConf.json
	```

	请注意，你可以遵循[如何为基于 Linux 的虚拟机重置密码或 SSH](/documentation/articles/virtual-machines-linux-use-vmaccess-reset-password-or-ssh) 中的类似步骤来尝试其他不同的做法。请记得修改资源管理器模式的 Azure CLI 指令。

	**使用 Azure PowerShell**

	如上所述安装并配置 Azure PowerShell。切换到资源管理器模式，然后如下所示运行扩展。

	```
	$RGName = 'testRG'
	$VmName = 'testVM'
	$Location = 'West US'

	$ExtensionName = 'VMAccessForLinux'
	$Publisher = 'Microsoft.OSTCExtensions'
	$Version = '1.2'

	$PublicConf = '{}'
	$PrivateConf = '{"username":"NewUsername", "password":"NewPassword", "ssh_key":"", "reset_ssh":false, "remove_user":""}'

	Set-AzureVMExtension -ResourceGroupName $RGName -VMName $VmName -Location $Location -Name $ExtensionName -Publisher $Publisher -ExtensionType $ExtensionName -TypeHandlerVersion $Version -SettingString $PublicConf -ProtectedSettingString $PrivateConf

	```

	请务必将 $RGName、$VmName、$Location 和 SSH 凭据的值替换为特定于安装的值。

## 详细的疑难解答

如果 SSH 客户端仍然无法连接到虚拟机上的 SSH 服务，原因可能是多方面的。下面是这种失败所涉及到的组件。

![](./media/virtual-machines-troubleshoot-ssh-connections/ssh-tshoot1.png)

以下部分将帮助你查明失败的原因，并得出解决方法或应对措施。

### 故障排除之前的步骤

首先，在 Azure 门户中检查虚拟机的状态。

在 [Azure 管理门户](https://manage.windowsazure.cn)中，针对采用经典部署模型的虚拟机：

1. 单击“虚拟机”>“VM 名称”。
2. 单击 VM 的“仪表板”以查看 VM 的状态。
3. 单击“监视器”，以查看计算、存储和网络资源的最近活动。
4. 单击“终结点”以确保 SSH 流量有终结点。
若要验证网络连接，请分析所配置的终结点，并确定是否可通过其他协议（例如 HTTP 或其他已知服务）连接到该虚拟机。


在执行这些步骤之后，重新尝试 SSH 连接。


### 疑难解答步骤

如果计算机上的 SSH 客户端无法连接到 Azure 虚拟机上的 SSH 服务，则可能是以下问题或不当配置所造成的：

- SSH 客户端计算机
- 组织边缘设备
- 云服务终结点和访问控制列表 (ACL)
- 网络安全组
- 基于 Linux 的 Azure 虚拟机

#### 来源 1：SSH 客户端计算机

若要将你的计算机从失败原因中排除，请检查你的计算机是否能够与另一台基于 Linux 的本地计算机建立 SSH 连接。

![](./media/virtual-machines-troubleshoot-ssh-connections/ssh-tshoot2.png)

如果不能，请检查你的计算机上是否有以下项：

- 本地防火墙设置阻止了入站或出站 SSH 流量 (TCP 22)
- 本地安装的客户端代理软件阻止了 SSH 连接
- 本地安装的网络监视软件阻止了 SSH 连接
- 监视流量或允许/禁止特定类型流量的其他类型的安全软件

对于所有这些情况，请暂时禁用可疑软件，然后尝试与本地计算机建立 SSH 连接以找出原因。然后，与网络管理员合作以更正软件的设置，从而允许 SSH 连接。

如果使用的是证书身份验证，则需验证你是否具有这些权限以访问主目录中的 .ssh 文件夹：

- Chmod 700 ~/.ssh
- Chmod 644 ~/.ssh/\*.pub
- Chmod 600 ~/.ssh/id_rsa（或存储私钥的其他任何文件）
- Chmod 644 ~/.ssh/known_hosts（包含已通过 SSH 连接到的主机）

#### 来源 2：组织边缘设备

若要使你的组织边缘设备不会成为失败的原因，请检查直接连接到 Internet 的计算机是否可以与 Azure VM 建立 SSH 连接。如果是通过站点到站点 VPN 或 ExpressRoute 连接来访问 VM，请跳转到[来源 4：网络安全组](#nsg)。

![](./media/virtual-machines-troubleshoot-ssh-connections/ssh-tshoot3.png)

如果没有直接连接到 Internet 的计算机，可以轻松地在其自己的资源组或云服务中创建新的 Azure 虚拟机，然后进行使用。有关详细信息，请参阅[在 Azure 中创建运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-tutorial-portal-rm)。测试完成后，请删除资源组或虚拟机以及云服务。

如果可以创建与直接连接到 Internet 的计算机之间的 SSH 连接，则检查你的组织边缘设备中是否存在以下问题：

- 内部防火墙阻止了与 Internet 的 SSH 连接
- 代理服务器阻止了 SSH 连接
- 边界网络中的设备上运行的入侵检测或网络监视软件阻止了 SSH 连接

与网络管理员合作以更正组织边缘设备的设置，从而允许与 Internet 建立 SSH 流量连接。

#### 来源 3：云服务终结点和 ACL

> [AZURE.NOTE]此来源仅适用于使用经典部署模型创建的虚拟机。对于使用资源管理器中创建的虚拟机，请跳转到[来源 4：网络安全组](#nsg)。

若要将云服务终结点和 ACL 从失败原因中排除，请检查同一虚拟网络中的其他 Azure VM 是否可与使用[经典部署模型](/documentation/articles/resource-manager-deployment-model)创建的 VM 建立 SSH 连接。

![](./media/virtual-machines-troubleshoot-ssh-connections/ssh-tshoot4.png)

如果同一虚拟网络中没有其他 VM，你可以轻松创建一个新 VM。有关详细信息，请参阅[在 Azure 中创建运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-tutorial-portal-rm)。测试完成后，请删除多余的 VM。

如果可以与同一虚拟网络中的某个 VM 建立 SSH 连接，请检查：

- 目标 VM 上 SSH 流量的终结点配置。终结点的专用 TCP 端口应该与 VM 上 SSH 服务正在其上侦听的 TCP 端口（默认为 22）匹配。对于在资源管理器部署模型中使用模板创建的 VM，请通过“浏览”>“虚拟机(v2)”>“VM 名称”>“设置”>“终结点”，验证 Azure 预览门户中的 SSH TCP 端口号。
- 目标虚拟机上的 SSH 流量终结点的 ACL。ACL 允许你指定基于源 IP 地址允许或拒绝的从 Internet 传入的流量。错误配置的 ACL 可能会阻止 SSH 流量传入终结点。检查你的 ACL 以确保允许从你的代理服务器或其他边缘服务器的公共 IP 地址传入的流量。有关详细信息，请参阅[关于网络访问控制列表 (ACL)](/documentation/articles/virtual-networks-acl)。

要使终结点不会成为问题来源，请删除当前终结点，然后创建一个新的终结点并指定 **SSH** 名称（公用和专用端口号为 TCP 端口 22）。有关详细信息，请参阅[在 Azure 中的虚拟机上设置终结点](/documentation/articles/virtual-machines-set-up-endpoints)。

<a id="nsg">
#### 来源 4：网络安全组

通过使用网络安全组，可以对允许的入站和出站流量进行更精细的控制。你可以创建跨 Azure 虚拟网络中的子网和云服务的规则。检查你的网络安全组规则，以确保允许来自和去往 Internet 的 SSH 流量。
有关详细信息，请参阅[关于网络安全组](/documentation/articles/virtual-networks-nsg)。

#### 来源 5：基于 Linux 的 Azure 虚拟机

最后一个可能出现问题的来源是 Azure 虚拟机本身。

![](./media/virtual-machines-troubleshoot-ssh-connections/ssh-tshoot5.png)

如果尚未这样做，请按照[如何为基于 Linux 的虚拟机重置密码或 SSH](/documentation/articles/virtual-machines-linux-use-vmaccess-reset-password-or-ssh) 中的说明，在虚拟机上执行操作。

再次尝试从你的计算机建立连接。如果仍然失败，则可能存在以下问题：

- 目标虚拟机上未运行 SSH 服务。
- TCP 端口 22 上未侦听 SSH 服务。若要测试这一点，可在本地计算机上安装一个远程登录客户端，然后运行“telnet *cloudServiceName*.chinacloudapp.cn 22”。这将确定虚拟机是否允许与 SSH 终结点进行入站和出站通信。
- 目标虚拟机上的本地防火墙具有阻止入站或出站 SSH 流量的规则。
- Azure 虚拟机上运行的入侵检测或网络监视软件阻止了 SSH 连接。


## 其他资源

对于采用经典部署模型的虚拟机，请参阅[如何为基于 Linux 的虚拟机重置密码或 SSH](/documentation/articles/virtual-machines-linux-use-vmaccess-reset-password-or-ssh)

[对与基于 Windows 的 Azure 虚拟机的 Windows 远程桌面连接进行故障排除](/documentation/articles/virtual-machines-troubleshoot-remote-desktop-connections)

[对在 Azure 虚拟机上运行的应用程序的访问进行故障排除](/documentation/articles/virtual-machines-troubleshoot-access-application)

<!---HONumber=82-->