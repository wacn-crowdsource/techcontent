<properties 
pageTitle="设置 Azure AD 以便进行自助服务应用程序访问管理 | Windows Azure" 
description="介绍如何在 Azure AD 中管理组的主题。" 
services="active-directory" 
documentationCenter=" 
authors="femila" 
manager="swadhwa"" 
editor="" 
tags="azure-classic-portal"/>

<tags 
	ms.service="active-directory" 
	ms.date="07/13/2015" 
	wacn.date="08/29/2015"/>

#设置 Azure AD 以进行自助服务应用程序访问管理

自助服务组管理使用户能够在 Windows Azure Active Directory (AD) 中创建和管理安全组，并为用户提供请求安全组成员资格的可能性，随后可由组的所有者批准或拒绝这一请求。使用自助服务组管理功能，组成员资格的日常控制可委派给了解该成员资格的业务上下文的人员。

自助服务组管理当前由两个基本方案组成：委托组管理和自助服务组管理。


- “委托组管理”示例：管理对其公司正在使用的 SaaS 应用程序的访问的管理员。管理这些访问权限变得越来越繁琐，因此，此管理员要求业务所有者创建一个新组。管理员现在将应用程序的访问权限分配给业务所有者刚创建的新组，并将当前有权访问该应用程序的所有人员放入此组。业务所有者可以添加更多用户，而稍后这些用户会被自动提供给应用程序。业务所有者不需要等待管理员来执行工作，但其本身可以为用户管理访问权限。管理员可为不同业务组的行政助理执行相同的操作，现在业务所有者和此行政助理均可为其用户管理访问权限，而不能访问或查看对方的用户。管理员仍可查看有权访问应用程序的所有用户，并可根据需要屏蔽访问权限。


- “自助服务组管理”以两个用户为例，这两个用户都具有单独建立的 SharePoint Online 站点，但他们非常希望能够轻松地给予对方团队访问权限。因此他们在 Azure AD 中创建一个组，并各自在 SharePoint Online 中选取这个组以提供对其站点的访问权限。要进行访问时，可从“访问面板”请求访问，审批后便能够自动访问两个 SharePoint Online 站点。随后，其中一个用户决定访问其站点的所有人也将能够访问特定的 SaaS 应用程序。他要求此 SaaS 应用程序的管理员将此应用程序的访问权限添加到其站点。从那以后，他批准的任何请求均将提供对两个 SharePoint Online 站点及该 SaaS 应用程序的访问。



##使组可用于最终用户自助服务

在 Azure 管理门户中，在“配置”选项卡上，将“委派组管理”开关设为“启用”，再将“用户可以创建组”开关设为“启用”。

当“用户可以创建组”开关设为“启用”时，目录中的所有用户均可创建新的安全组并向这些组添加成员。请注意，这些新组还将显示在所有其他用户的“访问面板”中；并且如果组的策略设置允许，其他用户还可创建请求以加入这些组。如果此开关设置为“禁用”，则用户不能创建组且不能更改所拥有的现有组，但仍可管理这些组的成员资格并批准其他用户加入组的请求。

还可使用“可使用安全组的自助服务的用户”开关来实现对用户的自助服务组管理能力更为精细的访问控制。当“用户可以创建组”开关设为“启用”时，目录中的所有用户均可创建新的安全组并向这些组添加成员。另外，将“可使用安全组的自助服务的用户”开关设置为“一些”，这其实是将安全组管理限制为一组有限的用户。当此开关设为“一些”时，将在目录中创建名为 SSGMSecurityGroupsUsers 的组，只有成为此组成员的用户才可以创建新的安全组并在目录内向这些组添加成员。将“可使用安全组的自助服务的用户”开关设为“全部”，则目录中的所有用户都可以创建新的安全组。

还可使用“可使用安全组的自助服务的组”字段（默认设置为“SSGMSecurityGroupsUsers”）来指定组的自定义名称，该组将为能够使用自助服务并在目录中创建新安全组的所有用户。

下面的主题将提供有关 Azure Active Directory 的一些其他信息

* [使用 Azure Active Directory 组管理对资源的访问](/documentation/articles/active-directory-manage-groups)

* [什么是 Azure Active Directory？](/documentation/articles/active-directory-whatis)

* [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect)

<!---HONumber=67-->