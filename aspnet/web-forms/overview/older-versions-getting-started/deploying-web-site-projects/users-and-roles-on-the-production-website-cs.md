---
uid: web-forms/overview/older-versions-getting-started/deploying-web-site-projects/users-and-roles-on-the-production-website-cs
title: "用户和生产网站 (C#) 上的角色 |Microsoft 文档"
author: rick-anderson
description: "ASP.NET 网站管理工具 (WSAT) 提供基于 web 的用户界面，用于配置成员资格和角色设置和创建，编辑，..."
ms.author: aspnetcontent
manager: wpickett
ms.date: 06/09/2009
ms.topic: article
ms.assetid: dbc54313-5d05-4285-98b3-726edea6d0c9
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/older-versions-getting-started/deploying-web-site-projects/users-and-roles-on-the-production-website-cs
msc.type: authoredcontent
ms.openlocfilehash: 0825b6bd6ca8d75f90cb7c4079e3af0213c5c4e9
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/10/2017
---
<a name="users-and-roles-on-the-production-website-c"></a><span data-ttu-id="8d140-103">用户和生产网站 (C#) 上的角色</span><span class="sxs-lookup"><span data-stu-id="8d140-103">Users and Roles On The Production Website (C#)</span></span>
====================
<span data-ttu-id="8d140-104">通过[Scott Mitchell](https://twitter.com/ScottOnWriting)</span><span class="sxs-lookup"><span data-stu-id="8d140-104">by [Scott Mitchell](https://twitter.com/ScottOnWriting)</span></span>

[<span data-ttu-id="8d140-105">下载 PDF</span><span class="sxs-lookup"><span data-stu-id="8d140-105">Download PDF</span></span>](http://download.microsoft.com/download/5/C/5/5C57DB8C-5DEA-4B3A-92CA-4405544D313B/aspnet_tutorial16_CustomAWAT_cs.pdf)

> <span data-ttu-id="8d140-106">ASP.NET 网站管理工具 (WSAT) 提供基于 web 的用户界面，用于配置成员资格和角色设置和用于创建、 编辑和删除用户和角色。</span><span class="sxs-lookup"><span data-stu-id="8d140-106">The ASP.NET Website Administration Tool (WSAT) provides a web-based user interface for configuring Membership and Roles settings and for creating, editing, and deleting users and roles.</span></span> <span data-ttu-id="8d140-107">遗憾的是，WSAT 时才有效访问从 localhost，这意味着你无法通过你浏览器访问生产网站管理工具。</span><span class="sxs-lookup"><span data-stu-id="8d140-107">Unfortunately, the WSAT only works when visited from localhost, meaning that you cannot reach the production website's Administration Tool through your browser.</span></span> <span data-ttu-id="8d140-108">好消息是解决方法，使用户可以管理用户和生产上的角色。</span><span class="sxs-lookup"><span data-stu-id="8d140-108">The good news is that there are workarounds that make it possible to manage users and roles on production.</span></span> <span data-ttu-id="8d140-109">本教程会查看这些解决方法和其他人。</span><span class="sxs-lookup"><span data-stu-id="8d140-109">This tutorial looks at these workarounds and others.</span></span>


## <a name="introduction"></a><span data-ttu-id="8d140-110">介绍</span><span class="sxs-lookup"><span data-stu-id="8d140-110">Introduction</span></span>

<span data-ttu-id="8d140-111">ASP.NET 2.0 引入了大量的*应用程序服务*，这是一套可添加到 web 应用程序的构建基块服务。</span><span class="sxs-lookup"><span data-stu-id="8d140-111">ASP.NET 2.0 introduced a number of *application services*, which are a suite of building block services that you can add to your web application.</span></span> <span data-ttu-id="8d140-112">我们添加后的成员资格和角色服务添加到图书评论网站[*配置网站，使用应用程序服务*教程](configuring-a-website-that-uses-application-services-cs.md)。</span><span class="sxs-lookup"><span data-stu-id="8d140-112">We added the Membership and Roles services to the Book Reviews website back in the [*Configuring a Website That Uses Application Services* tutorial](configuring-a-website-that-uses-application-services-cs.md).</span></span> <span data-ttu-id="8d140-113">成员资格服务有助于创建和管理用户帐户;角色服务提供用于对用户进行分类分组的 API。</span><span class="sxs-lookup"><span data-stu-id="8d140-113">The Membership service facilitates creating and managing user accounts; the Roles service offers an API for categorizing users into groups.</span></span> <span data-ttu-id="8d140-114">图书评论站点具有三个用户帐户-Scott、 Jisun，Alice-和单个角色，管理员，使用 Scott 和 Jisun 管理员角色中。</span><span class="sxs-lookup"><span data-stu-id="8d140-114">The Book Reviews site has three user accounts - Scott, Jisun, and Alice - and a single role, Admin, with Scott and Jisun in the Admin role.</span></span>

<span data-ttu-id="8d140-115">ASP。NET 的应用程序服务不限于特定的实现。</span><span class="sxs-lookup"><span data-stu-id="8d140-115">ASP.NET's application services are not tied to a specific implementation.</span></span> <span data-ttu-id="8d140-116">相反，指示要使用特定的应用程序服务*提供程序*，并该提供程序实现使用特定的技术的服务。</span><span class="sxs-lookup"><span data-stu-id="8d140-116">Instead, you instruct the application services to use a particular *provider*, and that provider implements the service using a particular technology.</span></span> <span data-ttu-id="8d140-117">我们配置图书评论 web 应用程序使用`SqlMembershipProvider`和`SqlRoleProvider`提供程序的成员资格和角色服务。</span><span class="sxs-lookup"><span data-stu-id="8d140-117">We configured the Book Reviews web application to use the `SqlMembershipProvider` and `SqlRoleProvider` providers for the Membership and Roles services.</span></span> <span data-ttu-id="8d140-118">这些两个提供程序用户帐户和角色信息存储在 SQL Server 数据库，并托管在 web 托管公司的基于 Internet 的 web 应用程序最常使用的提供程序。</span><span class="sxs-lookup"><span data-stu-id="8d140-118">These two providers store user account and role information in a SQL Server database and are the most commonly used providers for Internet-based web applications hosted at a web hosting company.</span></span>

<span data-ttu-id="8d140-119">常见的难题，对于使用成员资格和角色服务开发人员负责管理用户和角色在生产环境。</span><span class="sxs-lookup"><span data-stu-id="8d140-119">A common challenge for developers using the Membership and Roles services is managing the users and roles on the production environment.</span></span> <span data-ttu-id="8d140-120">你如何从生产网站中删除用户帐户、 添加新的角色，或将现有用户添加到现有的角色？</span><span class="sxs-lookup"><span data-stu-id="8d140-120">How do you delete a user account from the production website, add a new role, or add an existing user to an existing role?</span></span> <span data-ttu-id="8d140-121">本教程介绍了不同的技术来管理用户和生产网站上的角色。</span><span class="sxs-lookup"><span data-stu-id="8d140-121">This tutorial explores different techniques for managing users and roles on the production website.</span></span>

## <a name="using-the-aspnet-web-site-administration-tool"></a><span data-ttu-id="8d140-122">使用 ASP.NET 网站管理工具</span><span class="sxs-lookup"><span data-stu-id="8d140-122">Using the ASP.NET Web Site Administration Tool</span></span>

<span data-ttu-id="8d140-123">ASP.NET 包括[网站管理工具](https://msdn.microsoft.com/en-us/library/yy40ytx0.aspx)(WSAT)，可轻松创建和管理用户帐户和角色以及指定基于用户和角色的授权规则。</span><span class="sxs-lookup"><span data-stu-id="8d140-123">ASP.NET includes a [Web Site Administration Tool](https://msdn.microsoft.com/en-us/library/yy40ytx0.aspx) (WSAT) that makes it easy to create and manage user accounts and roles and to specify user- and role-based authorization rules.</span></span> <span data-ttu-id="8d140-124">若要使用 WSAT，单击 ASP.NET 配置图标，在解决方案资源管理器，或转到网站或项目菜单上，然后选择 ASP.NET 配置选项。</span><span class="sxs-lookup"><span data-stu-id="8d140-124">To use the WSAT, click the ASP.NET Configuration icon in the Solution Explorer, or go to the Website or Project menu and choose the ASP.NET Configuration option.</span></span> <span data-ttu-id="8d140-125">这两种方法来启动 web 浏览器并指向在这样的地址 WSAT:`http://localhost:portNumber/asp.netwebadminfiles/default.aspx?applicationPhysicalPath=pathToApplication`</span><span class="sxs-lookup"><span data-stu-id="8d140-125">Either approach launches a web browser and points it to the WSAT at an address like: `http://localhost:portNumber/asp.netwebadminfiles/default.aspx?applicationPhysicalPath=pathToApplication`</span></span>

<span data-ttu-id="8d140-126">WSAT 分为三个部分：</span><span class="sxs-lookup"><span data-stu-id="8d140-126">The WSAT is divided into three sections:</span></span>

- <span data-ttu-id="8d140-127">**安全**-管理用户、 角色和授权规则。</span><span class="sxs-lookup"><span data-stu-id="8d140-127">**Security** - manage users, roles, and authorization rules.</span></span>
- <span data-ttu-id="8d140-128">**应用程序配置**-管理&lt;appSettings&gt;和 SMTP 设置，从此处。</span><span class="sxs-lookup"><span data-stu-id="8d140-128">**ApplicationConfiguration** - manage the &lt;appSettings&gt; and SMTP settings from here.</span></span> <span data-ttu-id="8d140-129">你可以还使应用程序脱机和管理调试和跟踪从此处，设置，以及指定默认自定义错误页。</span><span class="sxs-lookup"><span data-stu-id="8d140-129">You can also take the application offline and manage debugging and tracing settings from here, as well as specify the default custom error page.</span></span>
- <span data-ttu-id="8d140-130">**ProviderConfiguration** -配置应用程序服务使用的提供程序。</span><span class="sxs-lookup"><span data-stu-id="8d140-130">**ProviderConfiguration** - configure the providers used by the application services.</span></span>

<span data-ttu-id="8d140-131">安全性部分 (中所示**图 1**) 包括链接，用于创建新用户、 管理用户、 创建和管理角色，以及创建和管理访问规则。</span><span class="sxs-lookup"><span data-stu-id="8d140-131">The Security section (shown in **Figure 1**) includes links for creating new users, managing users, creating and managing roles, and creating and managing access rules.</span></span> <span data-ttu-id="8d140-132">从此处你可以向系统添加新角色、 删除现有用户，或添加或删除特定的用户帐户角色。</span><span class="sxs-lookup"><span data-stu-id="8d140-132">From here you can add a new role to the system, delete an existing user, or add or remove roles from a particular user account.</span></span>

[![](users-and-roles-on-the-production-website-cs/_static/image2.png)](users-and-roles-on-the-production-website-cs/_static/image1.png)

<span data-ttu-id="8d140-133">**图 1**: WSAT 安全部分包含用于管理用户和角色的选项</span><span class="sxs-lookup"><span data-stu-id="8d140-133">**Figure 1**: The WSAT Security Section Includes Options for Managing Users and Roles</span></span>  
<span data-ttu-id="8d140-134">([单击以查看实际尺寸的图像](users-and-roles-on-the-production-website-cs/_static/image3.png))</span><span class="sxs-lookup"><span data-stu-id="8d140-134">([Click to view full-size image](users-and-roles-on-the-production-website-cs/_static/image3.png))</span></span>

<span data-ttu-id="8d140-135">遗憾的是，WSAT 才可访问本地。</span><span class="sxs-lookup"><span data-stu-id="8d140-135">Unfortunately, the WSAT is only accessible locally.</span></span> <span data-ttu-id="8d140-136">你无法在远程生产网站; 上访问 WSAT如果你访问`www.yoursite.com/asp.netwebadminfiles/default.aspx`获得 404 找不到响应。</span><span class="sxs-lookup"><span data-stu-id="8d140-136">You cannot visit the WSAT on your remote production website; if you visit `www.yoursite.com/asp.netwebadminfiles/default.aspx` you get a 404 Not Found response.</span></span> <span data-ttu-id="8d140-137">加强了 WSAT 使用的代码`Membership`和`Roles`类在.NET Framework 中，若要创建，编辑和删除用户和角色。</span><span class="sxs-lookup"><span data-stu-id="8d140-137">The code that powers the WSAT uses the `Membership` and `Roles` classes in the .NET Framework to create, edit, and delete users and roles.</span></span> <span data-ttu-id="8d140-138">这些类，请查阅 web 应用程序的配置信息来确定要使用; 哪些提供程序返回[*配置网站，使用应用程序服务*教程](configuring-a-website-that-uses-application-services-cs.md)我们设置图书评论网站以使用`SqlMembershipProvider`和`SqlRoleProvider`提供程序。</span><span class="sxs-lookup"><span data-stu-id="8d140-138">These classes consult the web application's configuration information to determine what provider to use; back in the [*Configuring a Website That Uses Application Services* tutorial](configuring-a-website-that-uses-application-services-cs.md) we setup the Book Reviews website to use the `SqlMembershipProvider` and `SqlRoleProvider` providers.</span></span> <span data-ttu-id="8d140-139">这需要执行添加`<membership>`和`<roleManager>`部分到`Web.config`。</span><span class="sxs-lookup"><span data-stu-id="8d140-139">This entailed adding `<membership>` and `<roleManager>` sections to `Web.config`.</span></span>

[!code-xml[Main](users-and-roles-on-the-production-website-cs/samples/sample1.xml)]

<span data-ttu-id="8d140-140">请注意，`<membership>`和`<roleManager>`部分引用`SqlMembershipProvider`和`SqlRoleProvider`中的提供程序其`type`特性，分别。</span><span class="sxs-lookup"><span data-stu-id="8d140-140">Note that the `<membership>` and `<roleManager>` sections reference the `SqlMembershipProvider` and `SqlRoleProvider` providers in their `type` attribute, respectively.</span></span> <span data-ttu-id="8d140-141">这些提供程序在指定的 SQL Server 数据库中存储的用户和角色信息。</span><span class="sxs-lookup"><span data-stu-id="8d140-141">These providers store the user and role information in a specified SQL Server database.</span></span> <span data-ttu-id="8d140-142">使用这些提供程序的数据库指定通过`connectionStringName`属性，`ReviewsConnectionString`中, 定义`~/ConfigSections/databaseConnectionStrings.config`文件。</span><span class="sxs-lookup"><span data-stu-id="8d140-142">The database used by these providers is specified by the `connectionStringName` attribute, `ReviewsConnectionString`, which is defined in the `~/ConfigSections/databaseConnectionStrings.config` file.</span></span> <span data-ttu-id="8d140-143">回想一下，`databaseConnectionStrings.config`在开发环境中的文件包含开发数据库的连接字符串，而`databaseConnectionStrings.config`上生产文件包含生产数据库的连接字符串。</span><span class="sxs-lookup"><span data-stu-id="8d140-143">Recall that the `databaseConnectionStrings.config` file in the development environment contains the connection string to the development database whereas the `databaseConnectionStrings.config` file on production contains the connection string to the production database.</span></span>

<span data-ttu-id="8d140-144">简而言之，必须开发环境中，通过本地访问 WSAT 和配合中指定的数据库中的用户和角色信息`databaseConnectionStrings.config`文件。</span><span class="sxs-lookup"><span data-stu-id="8d140-144">In a nutshell, the WSAT must be accessed locally through the development environment, and it works with the user and role information in the database specified in the `databaseConnectionStrings.config` file.</span></span> <span data-ttu-id="8d140-145">因此，如果我们更改中的连接字符串信息`databaseConnectionStrings.config`文件在开发环境我们可以使用 WSAT 本地来管理用户和生产环境中的角色。</span><span class="sxs-lookup"><span data-stu-id="8d140-145">Consequently, if we change the connection string information in the `databaseConnectionStrings.config` file on the development environment we can use the WSAT locally to manage users and roles in the production environment.</span></span>

<span data-ttu-id="8d140-146">若要演示此功能，打开`databaseConnectionStrings.config`上开发环境在 Visual Studio 中的文件并将开发数据库连接字符串替换为生产数据库连接字符串。</span><span class="sxs-lookup"><span data-stu-id="8d140-146">To illustrate this functionality, open the `databaseConnectionStrings.config` file in Visual Studio on the development environment and replace the development database connection string with the production database connection string.</span></span> <span data-ttu-id="8d140-147">然后启动 WSAT，请转到安全选项卡，并添加新用户密码"password ！"的名为 Sam</span><span class="sxs-lookup"><span data-stu-id="8d140-147">Then launch the WSAT, go the Security tab, and add a new user named Sam with password "password!"</span></span> <span data-ttu-id="8d140-148">（小于的引号）。</span><span class="sxs-lookup"><span data-stu-id="8d140-148">(less the quotation marks).</span></span> <span data-ttu-id="8d140-149">**图 2**显示 WSAT 屏幕时创建此帐户。</span><span class="sxs-lookup"><span data-stu-id="8d140-149">**Figure 2** shows the WSAT screen when creating this account.</span></span>

[![](users-and-roles-on-the-production-website-cs/_static/image5.png)](users-and-roles-on-the-production-website-cs/_static/image4.png)

<span data-ttu-id="8d140-150">**图 2**： 创建名为 Sam 在生产环境中的新用户</span><span class="sxs-lookup"><span data-stu-id="8d140-150">**Figure 2**: Create a New User Named Sam In the Production Environment</span></span>  
<span data-ttu-id="8d140-151">([单击以查看实际尺寸的图像](users-and-roles-on-the-production-website-cs/_static/image6.png))</span><span class="sxs-lookup"><span data-stu-id="8d140-151">([Click to view full-size image](users-and-roles-on-the-production-website-cs/_static/image6.png))</span></span>

<span data-ttu-id="8d140-152">因为我们更改中的连接字符串`databaseConnectionStrings.config`指向生产数据库服务器，Sam 已添加为生产环境中的用户。</span><span class="sxs-lookup"><span data-stu-id="8d140-152">Because we changed the connection string in `databaseConnectionStrings.config` to point to the production database server, Sam was added as a user in the production environment.</span></span> <span data-ttu-id="8d140-153">若要验证这一点，更改中的连接字符串`databaseConnectionStrings.config`回开发数据库文件，然后访问`Login.aspx`在开发环境中的页。</span><span class="sxs-lookup"><span data-stu-id="8d140-153">To verify this, change the connection string in the `databaseConnectionStrings.config` file back to the development database and then visit the `Login.aspx` page in the development environment.</span></span> <span data-ttu-id="8d140-154">尝试以 Sam 登录 (请参阅**图 3**)。</span><span class="sxs-lookup"><span data-stu-id="8d140-154">Try to sign in as Sam (see **Figure 3**).</span></span>

[![](users-and-roles-on-the-production-website-cs/_static/image8.png)](users-and-roles-on-the-production-website-cs/_static/image7.png)

<span data-ttu-id="8d140-155">**图 3**： 不能以在开发环境中的 Sam 登录</span><span class="sxs-lookup"><span data-stu-id="8d140-155">**Figure 3**: You Cannot Sign In As Sam in the Development Environment</span></span>  
<span data-ttu-id="8d140-156">([单击以查看实际尺寸的图像](users-and-roles-on-the-production-website-cs/_static/image9.png))</span><span class="sxs-lookup"><span data-stu-id="8d140-156">([Click to view full-size image](users-and-roles-on-the-production-website-cs/_static/image9.png))</span></span>

<span data-ttu-id="8d140-157">你无法登录，Sam 作为开发环境中因为本地数据库中不存在的用户帐户信息。</span><span class="sxs-lookup"><span data-stu-id="8d140-157">You cannot sign in as Sam in the development environment because the user account information does not exist in the local database.</span></span> <span data-ttu-id="8d140-158">相反，是已添加到生产数据库。</span><span class="sxs-lookup"><span data-stu-id="8d140-158">Rather, is was added to the production database.</span></span> <span data-ttu-id="8d140-159">若要验证这一点，查看的内容`aspnet_Users`开发和生产数据库中的表。</span><span class="sxs-lookup"><span data-stu-id="8d140-159">To verify this, view the contents of the `aspnet_Users` table in both the development and production databases.</span></span> <span data-ttu-id="8d140-160">在开发环境中应该有 Scott、 Jisun 和 Alice 的用户只有三个记录。</span><span class="sxs-lookup"><span data-stu-id="8d140-160">In the development environment there should be only three records for users Scott, Jisun, and Alice.</span></span> <span data-ttu-id="8d140-161">但是，`aspnet_Users`生产数据库中的表具有四个记录： Scott、 Jisun、 Alice 和 Sam。</span><span class="sxs-lookup"><span data-stu-id="8d140-161">However, the `aspnet_Users` table in the production database has four records: Scott, Jisun, Alice, and Sam.</span></span> <span data-ttu-id="8d140-162">因此，Sam 可以登录通过网站在生产中，但无法通过开发环境。</span><span class="sxs-lookup"><span data-stu-id="8d140-162">Consequently, Sam can sign in through the website in production, but not through the development environment.</span></span>

[![](users-and-roles-on-the-production-website-cs/_static/image11.png)](users-and-roles-on-the-production-website-cs/_static/image10.png)

<span data-ttu-id="8d140-163">**图 4**: Sam 可以登录生产网站</span><span class="sxs-lookup"><span data-stu-id="8d140-163">**Figure 4**: Sam Can Sign In On the Production Website</span></span>  
<span data-ttu-id="8d140-164">([单击以查看实际尺寸的图像](users-and-roles-on-the-production-website-cs/_static/image12.png))</span><span class="sxs-lookup"><span data-stu-id="8d140-164">([Click to view full-size image](users-and-roles-on-the-production-website-cs/_static/image12.png))</span></span>

> [!NOTE]
> <span data-ttu-id="8d140-165">不要忘记将更改中的连接字符串`databaseConnectionStrings.config`回开发数据库文件的连接字符串，完成后使用的 WSAT 否则为测试通过开发站点时将使用与生产数据环境。</span><span class="sxs-lookup"><span data-stu-id="8d140-165">Don't forget to change the connection string in the `databaseConnectionStrings.config` file back to the development database's connect string when you're done working with the WSAT otherwise you will be working with production data when testing the site through the development environment.</span></span> <span data-ttu-id="8d140-166">此外请记住，虽然我们刚刚讨论的技术使我们能够使用 WSAT 来远程管理用户和角色，则对任何其他 WSAT 配置选项 （访问规则、 SMTP 设置、 调试和跟踪设置，等等） 的更改将修改`Web.config`文件。</span><span class="sxs-lookup"><span data-stu-id="8d140-166">Also keep in mind that while the technique we just discussed allows us to use the WSAT to remotely manage users and roles, changes to any of the other WSAT configuration options (access rules, SMTP settings, debugging and tracing settings, and so on) modify the `Web.config` file.</span></span> <span data-ttu-id="8d140-167">因此，对设置进行任何更改应用于开发环境而不适用于生产环境。</span><span class="sxs-lookup"><span data-stu-id="8d140-167">Consequently, any changes made to the settings apply to the development environment and not to the production environment.</span></span>


## <a name="creating-custom-user-and-role-management-web-pages"></a><span data-ttu-id="8d140-168">创建自定义用户和角色管理 Web 页</span><span class="sxs-lookup"><span data-stu-id="8d140-168">Creating Custom User and Role Management Web Pages</span></span>

<span data-ttu-id="8d140-169">WSAT 外框系统提供的管理用户和角色，但仅本地启动，并且需要为了管理用户和角色在生产上的更改连接字符串信息。</span><span class="sxs-lookup"><span data-stu-id="8d140-169">The WSAT provides an out of the box system for managing users and roles, but can only be launched locally and requires making changes to the connection string information in order to manage the users and roles on production.</span></span> <span data-ttu-id="8d140-170">支持用户帐户的大多数网站还包括大量的用户和角色管理 web 页，使管理员能够管理从站点中的页面的用户和角色。</span><span class="sxs-lookup"><span data-stu-id="8d140-170">Most websites that support user accounts also include a number of user and role administration web pages that enable administrators to manage users and roles from pages within the site.</span></span> <span data-ttu-id="8d140-171">此类基于 web 的管理页使其可以更轻松地管理用户和角色以及站点必需其中可能有许多管理员或不具有访问或使用 Visual Studio 启动 WSAT 的技术背景的管理员。</span><span class="sxs-lookup"><span data-stu-id="8d140-171">Such web-based administration pages make it much easier to manage users and roles and are essential for sites where there may be many administrators or administrators that do not have access to or the technical background to use Visual Studio to launch the WSAT.</span></span>

<span data-ttu-id="8d140-172">ASP.NET 包括大量内置进行实现许多这些管理网页一样简单，如拖放的登录名相关的 Web 控件。</span><span class="sxs-lookup"><span data-stu-id="8d140-172">ASP.NET includes a number of built-in Login-related Web controls that make implementing many of these administrative web pages as easy as drag and drop.</span></span> <span data-ttu-id="8d140-173">例如，你可以创建管理员将通过拖动到该页面上然后设置的几个属性创建新的用户帐户的页。</span><span class="sxs-lookup"><span data-stu-id="8d140-173">For example, you can create a page for administrators to create a new user account by dragging the CreateUserWizard control onto the page and setting a few properties.</span></span> <span data-ttu-id="8d140-174">在事实中，在中所示 WSAT 中创建用户的页**图 2**使用相同的通过可添加到页面。</span><span class="sxs-lookup"><span data-stu-id="8d140-174">In fact, the page for creating users in the WSAT shown in **Figure 2** uses the same CreateUserWizard control that you can add to your pages.</span></span> <span data-ttu-id="8d140-175">此外，该成员资格和角色服务的功能能够以编程方式通过`Membership`和`Roles`.NET Framework 中的类。</span><span class="sxs-lookup"><span data-stu-id="8d140-175">Furthermore, the Membership and Roles services' functionalities are available programmatically through the `Membership` and `Roles` classes in the .NET Framework.</span></span> <span data-ttu-id="8d140-176">使用这些类可以编写代码以创建、 编辑和删除用户和角色，也可以添加或删除用户角色，以确定用户在哪些角色中，并执行其他用户和角色相关的任务。</span><span class="sxs-lookup"><span data-stu-id="8d140-176">With these classes you can write code to create, edit, and delete users and roles, as well as to add or remove users to roles, to determine what users are in what roles, and to perform other user- and role-related tasks.</span></span>

<span data-ttu-id="8d140-177">在[*配置网站，使用应用程序服务*教程](configuring-a-website-that-uses-application-services-cs.md)我添加到页`Admin`文件夹名为`CreateAccount.aspx`。</span><span class="sxs-lookup"><span data-stu-id="8d140-177">In the [*Configuring a Website That Uses Application Services* tutorial](configuring-a-website-that-uses-application-services-cs.md) I added a page to the `Admin` folder named `CreateAccount.aspx`.</span></span> <span data-ttu-id="8d140-178">此页面允许管理员将新的用户帐户添加到站点并指定新创建的用户是否是管理员角色中 (请参阅**图 5**)。</span><span class="sxs-lookup"><span data-stu-id="8d140-178">This page allows an administrator to add a new user account to the site and to specify whether or not the newly created user is in the Admin role (see **Figure 5**).</span></span>

[![](users-and-roles-on-the-production-website-cs/_static/image14.png)](users-and-roles-on-the-production-website-cs/_static/image13.png)

<span data-ttu-id="8d140-179">**图 5**： 管理员可以创建新用户帐户</span><span class="sxs-lookup"><span data-stu-id="8d140-179">**Figure 5**: Administrators Can Create New User Accounts</span></span>  
<span data-ttu-id="8d140-180">([单击以查看实际尺寸的图像](users-and-roles-on-the-production-website-cs/_static/image15.png))</span><span class="sxs-lookup"><span data-stu-id="8d140-180">([Click to view full-size image](users-and-roles-on-the-production-website-cs/_static/image15.png))</span></span>

<span data-ttu-id="8d140-181">了解生成用户和角色的管理页，以及使用的分步说明的更多详细信息`Membership`和`Roles`类和登录名相关的 ASP.NET Web 控件，请务必阅读我[网站的安全教程](../../older-versions-security/introduction/security-basics-and-asp-net-support-cs.md)。</span><span class="sxs-lookup"><span data-stu-id="8d140-181">For a more detailed look at building user and role administration pages, along with step-by-step instructions on using the `Membership` and `Roles` classes and the Login-related ASP.NET Web controls, be sure to read my [Website Security Tutorials](../../older-versions-security/introduction/security-basics-and-asp-net-support-cs.md).</span></span> <span data-ttu-id="8d140-182">那里你将如何生成用于创建新帐户、 创建和管理角色，网页将用户分配到角色，以及其他常见的管理任务找到指导。</span><span class="sxs-lookup"><span data-stu-id="8d140-182">There you'll find guidance on how to build web pages for creating new accounts, creating and managing roles, assigning users to roles, and other common administrative tasks.</span></span>

<span data-ttu-id="8d140-183">若要实现 WSAT 类似的功能，生产网站上始终可以生成实现 WSAT 的功能的网页的序列。</span><span class="sxs-lookup"><span data-stu-id="8d140-183">To implement WSAT-like functionality on the production website you can always build your own series of web pages that implement the WSAT's features.</span></span> <span data-ttu-id="8d140-184">若要帮助开始，请查看位于文件夹中的 WSAT 源代码`%WINDIR%\Microsoft.NET\Framework\v2.0.50727\ASP.NETWebAdminFiles`。</span><span class="sxs-lookup"><span data-stu-id="8d140-184">To help get started, check out the WSAT source code, which is located in the folder `%WINDIR%\Microsoft.NET\Framework\v2.0.50727\ASP.NETWebAdminFiles`.</span></span> <span data-ttu-id="8d140-185">另一个选项是使用他共享在他的文章中的 Dan Clem WSAT 备选[滚动你自己网站管理工具](http://aspnet.4guysfromrolla.com/articles/052307-1.aspx)。</span><span class="sxs-lookup"><span data-stu-id="8d140-185">Another option is to use Dan Clem's WSAT alternative, which he shares in his article, [Rolling Your Own Web Site Administration Tool](http://aspnet.4guysfromrolla.com/articles/052307-1.aspx).</span></span> <span data-ttu-id="8d140-186">Dan 指导通过生成一个自定义的 WSAT 类似的工具的进程的读取器，包括其应用程序的源代码 （在 C# 中)，下载和提供将其自定义 WSAT 添加到托管网站的分步说明。</span><span class="sxs-lookup"><span data-stu-id="8d140-186">Dan walks readers through the process of building a custom WSAT-like tool, includes his application's source code for download (in C#), and gives step-by-step instructions for adding his custom WSAT to a hosted website.</span></span>

## <a name="summary"></a><span data-ttu-id="8d140-187">摘要</span><span class="sxs-lookup"><span data-stu-id="8d140-187">Summary</span></span>

<span data-ttu-id="8d140-188">ASP.NET 网站管理工具 (WSAT) 可相继使用成员资格和角色应用程序服务来管理为你的网站的用户和角色信息。</span><span class="sxs-lookup"><span data-stu-id="8d140-188">The ASP.NET Web Site Administration Tool (WSAT) can be used in tandem with the Membership and Roles application services to manage user and role information for your website.</span></span> <span data-ttu-id="8d140-189">遗憾的是，WSAT 本地只能为可访问，并且不能从你的生产网站访问。</span><span class="sxs-lookup"><span data-stu-id="8d140-189">Unfortunately, the WSAT is only accessible locally and cannot be visited from your production website.</span></span> <span data-ttu-id="8d140-190">但是，通过更改连接字符串在开发环境以点到生产数据库可以使用 WSAT 来管理用户和生产网站上的角色。</span><span class="sxs-lookup"><span data-stu-id="8d140-190">However, by changing the connection string in the development environment to point to the production database you can use the WSAT to manage the users and roles on the production website.</span></span>

<span data-ttu-id="8d140-191">虽然 WSAT 方法提供了快速轻松地管理用户和角色，它要求启动到的连接字符串信息从 Visual Studio，以及临时更改 WSAT。</span><span class="sxs-lookup"><span data-stu-id="8d140-191">While the WSAT approach affords a quick and easy way to manage users and roles, it necessitates launching the WSAT from Visual Studio as well as temporary changes to the connection string information.</span></span> <span data-ttu-id="8d140-192">WSAT 提供了一个快速方法来管理用户和角色在生产环境，但繁琐，不与多个管理员或管理员不具有或不熟悉 Visual Studio 和 WSAT 适用于网站不适用。</span><span class="sxs-lookup"><span data-stu-id="8d140-192">The WSAT offers a quick way to manage users and roles on production, but is cumbersome and does not work well for websites with multiple administrators or with administrators who do not have or are not familiar with Visual Studio and the WSAT.</span></span> <span data-ttu-id="8d140-193">出于这些原因，支持用户帐户的大多数网站包括一组管理的 web 页面。</span><span class="sxs-lookup"><span data-stu-id="8d140-193">For these reasons, most websites that support user accounts include a set of administrative web pages.</span></span> <span data-ttu-id="8d140-194">此类组的网页无需进行 WSAT 且使用的各种管理用户从任何计算机。</span><span class="sxs-lookup"><span data-stu-id="8d140-194">Such a set of web pages eliminates the need for the WSAT and used by various administrative users from any computer.</span></span>

<span data-ttu-id="8d140-195">尽情享受编程 ！</span><span class="sxs-lookup"><span data-stu-id="8d140-195">Happy Programming!</span></span>

### <a name="further-reading"></a><span data-ttu-id="8d140-196">其他阅读材料</span><span class="sxs-lookup"><span data-stu-id="8d140-196">Further Reading</span></span>

<span data-ttu-id="8d140-197">在本教程中讨论的主题的详细信息，请参阅以下资源：</span><span class="sxs-lookup"><span data-stu-id="8d140-197">For more information on the topics discussed in this tutorial, refer to the following resources:</span></span>

- [<span data-ttu-id="8d140-198">正在检查 ASP。NET 的成员资格、 角色和配置文件</span><span class="sxs-lookup"><span data-stu-id="8d140-198">Examining ASP.NET's Membership, Roles, and Profile</span></span>](http://aspnet.4guysfromrolla.com/articles/120705-1.aspx)
- [<span data-ttu-id="8d140-199">滚动你自己的网站管理工具</span><span class="sxs-lookup"><span data-stu-id="8d140-199">Rolling Your Own Web Site Administration Tool</span></span>](http://aspnet.4guysfromrolla.com/articles/052307-1.aspx)
- [<span data-ttu-id="8d140-200">网站管理工具概述</span><span class="sxs-lookup"><span data-stu-id="8d140-200">Web Site Administration Tool Overview</span></span>](https://msdn.microsoft.com/en-us/library/yy40ytx0.aspx)
- [<span data-ttu-id="8d140-201">网站安全教程</span><span class="sxs-lookup"><span data-stu-id="8d140-201">Website Security Tutorials</span></span>](../../older-versions-security/introduction/security-basics-and-asp-net-support-cs.md)

>[!div class="step-by-step"]
<span data-ttu-id="8d140-202">[上一页](precompiling-your-website-cs.md)
[下一页](asp-net-hosting-options-vb.md)</span><span class="sxs-lookup"><span data-stu-id="8d140-202">[Previous](precompiling-your-website-cs.md)
[Next](asp-net-hosting-options-vb.md)</span></span>