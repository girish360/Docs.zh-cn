---
title: "ASP.NET Core 中的日志记录"
author: ardalis
description: "了解 ASP.NET Core 中的记录框架。 发现内置日志记录提供程序，并详细了解常见第三方提供程序。"
keywords: "ASP.NET Core,日志记录,日志记录提供程序,Microsoft.Extensions.Logging,ILogger,ILoggerFactory,LogLevel,WithFilter,TraceSource,EventLog,EventSource,scopes"
ms.author: tdykstra
manager: wpickett
ms.date: 12/15/2017
ms.topic: article
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/logging/index
ms.openlocfilehash: 737de614625ce560df1c3d7cfd9810f9433c153d
ms.sourcegitcommit: f1436107b4c022b26f5235dddef103cec5aa6bff
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/15/2017
---
# <a name="introduction-to-logging-in-aspnet-core"></a><span data-ttu-id="224be-105">ASP.NET Core 中的日志记录简介</span><span class="sxs-lookup"><span data-stu-id="224be-105">Introduction to logging in ASP.NET Core</span></span>

<span data-ttu-id="224be-106">作者：[Steve Smith](https://ardalis.com/) 和 [Tom Dykstra](https://github.com/tdykstra)</span><span class="sxs-lookup"><span data-stu-id="224be-106">By [Steve Smith](https://ardalis.com/) and [Tom Dykstra](https://github.com/tdykstra)</span></span>

<span data-ttu-id="224be-107">ASP.NET Core 支持适用于各种日志记录提供程序的日志记录 API。</span><span class="sxs-lookup"><span data-stu-id="224be-107">ASP.NET Core supports a logging API that works with a variety of logging providers.</span></span> <span data-ttu-id="224be-108">通过内置提供程序，可向一个或多个目标发送日志，还可插入第三方记录框架。</span><span class="sxs-lookup"><span data-stu-id="224be-108">Built-in providers let you send logs to one or more destinations, and you can plug in a third-party logging framework.</span></span> <span data-ttu-id="224be-109">本文介绍如何在代码中使用内置日志记录 API 和提供程序。</span><span class="sxs-lookup"><span data-stu-id="224be-109">This article shows how to use the built-in logging API and providers in your code.</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="224be-110">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="224be-110">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="224be-111">[查看或下载示例代码](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/logging/index/sample2)（[如何下载](xref:tutorials/index#how-to-download-a-sample)）</span><span class="sxs-lookup"><span data-stu-id="224be-111">[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/logging/index/sample2) ([how to download](xref:tutorials/index#how-to-download-a-sample))</span></span>

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="224be-112">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="224be-112">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

<span data-ttu-id="224be-113">[查看或下载示例代码](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/logging/index/sample)（[如何下载](xref:tutorials/index#how-to-download-a-sample)）</span><span class="sxs-lookup"><span data-stu-id="224be-113">[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/logging/index/sample) ([how to download](xref:tutorials/index#how-to-download-a-sample))</span></span>

---

## <a name="how-to-create-logs"></a><span data-ttu-id="224be-114">如何创建日志</span><span class="sxs-lookup"><span data-stu-id="224be-114">How to create logs</span></span>

<span data-ttu-id="224be-115">要创建日志，请先从[依赖关系注入](xref:fundamentals/dependency-injection)容器获取 `ILogger` 对象：</span><span class="sxs-lookup"><span data-stu-id="224be-115">To create logs, get an `ILogger` object from the [dependency injection](xref:fundamentals/dependency-injection) container:</span></span>

[!code-csharp[](index/sample/Controllers/TodoController.cs?name=snippet_LoggerDI&highlight=7)]

<span data-ttu-id="224be-116">然后在该记录器对象上调用日志记录方法：</span><span class="sxs-lookup"><span data-stu-id="224be-116">Then call logging methods on that logger object:</span></span>

[!code-csharp[](index/sample/Controllers/TodoController.cs?name=snippet_CallLogMethods&highlight=3,7)]

<span data-ttu-id="224be-117">此示例使用 `TodoController` 类创建日志作为类别。</span><span class="sxs-lookup"><span data-stu-id="224be-117">This example creates logs with the `TodoController` class as the *category*.</span></span> <span data-ttu-id="224be-118">[本文的稍后部分](#log-category)对这些类别进行了介绍。</span><span class="sxs-lookup"><span data-stu-id="224be-118">Categories are explained [later in this article](#log-category).</span></span>

<span data-ttu-id="224be-119">ASP.NET Core 不提供异步记录器方法，因为日志记录的速度应快到无需使用异步。</span><span class="sxs-lookup"><span data-stu-id="224be-119">ASP.NET Core does not provide async logger methods because logging should be so fast that it isn't worth the cost of using async.</span></span> <span data-ttu-id="224be-120">如果发现自己的实际情况与上述不同，请考虑更改记录方式。</span><span class="sxs-lookup"><span data-stu-id="224be-120">If you're in a situation where that's not true, consider changing the way you log.</span></span> <span data-ttu-id="224be-121">如果数据存储速度较慢，请先将日志消息写入快速存储，稍后再将其转移至低速存储。</span><span class="sxs-lookup"><span data-stu-id="224be-121">If your data store is slow, write the log messages to a fast store first, then move them to a slow store later.</span></span> <span data-ttu-id="224be-122">例如，记录到由另一进程读取和保留以减缓存储的消息队列。</span><span class="sxs-lookup"><span data-stu-id="224be-122">For example, log to a message queue that is read and persisted to slow storage by another process.</span></span>

## <a name="how-to-add-providers"></a><span data-ttu-id="224be-123">如何添加提供程序</span><span class="sxs-lookup"><span data-stu-id="224be-123">How to add providers</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="224be-124">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="224be-124">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="224be-125">日志记录提供程序通过 `ILogger` 对象获取所创建的消息，并显示或存储它们。</span><span class="sxs-lookup"><span data-stu-id="224be-125">A logging provider takes the messages that you create with an `ILogger` object and displays or stores them.</span></span> <span data-ttu-id="224be-126">例如，控制台提供程序在控制台上显示消息，Azure App Service 提供程序可将消息存储在 Azure blob 存储中。</span><span class="sxs-lookup"><span data-stu-id="224be-126">For example, the Console provider displays messages on the console, and the Azure App Service provider can store them in Azure blob storage.</span></span>

<span data-ttu-id="224be-127">要使用提供程序，请在 Program.cs 中调用提供程序的 `Add<ProviderName>` 扩展方法：</span><span class="sxs-lookup"><span data-stu-id="224be-127">To use a provider, call the provider's `Add<ProviderName>` extension method in *Program.cs*:</span></span>

[!code-csharp[](index/sample2/Program.cs?name=snippet_ExpandDefault&highlight=16,17)]

<span data-ttu-id="224be-128">默认项目模板设置日志记录的方式与先前代码所示一致，但要由 `CreateDefaultBuilder` 方法来执行 `ConfigureLogging` 调用。</span><span class="sxs-lookup"><span data-stu-id="224be-128">The default project template sets up logging the way you see it in the preceding code, but the `ConfigureLogging` call is done by the `CreateDefaultBuilder` method.</span></span> <span data-ttu-id="224be-129">以下是 Program.cs 中由项目模板创建的代码：</span><span class="sxs-lookup"><span data-stu-id="224be-129">Here's the code in *Program.cs* that is created by project templates:</span></span>

[!code-csharp[](index/sample2/Program.cs?name=snippet_TemplateCode&highlight=7)]

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="224be-130">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="224be-130">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

<span data-ttu-id="224be-131">日志记录提供程序通过 `ILogger` 对象获取所创建的消息，并显示或存储它们。</span><span class="sxs-lookup"><span data-stu-id="224be-131">A logging provider takes the messages that you create with an `ILogger` object and displays or stores them.</span></span> <span data-ttu-id="224be-132">例如，控制台提供程序在控制台上显示消息，Azure App Service 提供程序可将消息存储在 Azure blob 存储中。</span><span class="sxs-lookup"><span data-stu-id="224be-132">For example, the Console provider displays messages on the console, and the Azure App Service provider can store them in Azure blob storage.</span></span>

<span data-ttu-id="224be-133">要使用提供程序，请安装其 NuGet 包，并在 `ILoggerFactory` 的实例上调用提供程序的扩展方法，如下方示例所示。</span><span class="sxs-lookup"><span data-stu-id="224be-133">To use a provider, install its NuGet package and call the provider's extension method on an instance of `ILoggerFactory`, as shown in the following example.</span></span>

[!code-csharp[](index/sample//Startup.cs?name=snippet_AddConsoleAndDebug&highlight=3,5-7)]

<span data-ttu-id="224be-134">ASP.NET Core [依赖关系注入](xref:fundamentals/dependency-injection) (DI) 将提供 `ILoggerFactory` 实例。</span><span class="sxs-lookup"><span data-stu-id="224be-134">ASP.NET Core [dependency injection](xref:fundamentals/dependency-injection) (DI) provides the `ILoggerFactory` instance.</span></span> <span data-ttu-id="224be-135">在 [Microsoft.Extensions.Logging.Console](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Console/) 和 [Microsoft.Extensions.Logging.Debug](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Debug/) 包中定义了 `AddConsole` 和 `AddDebug` 扩展方法。</span><span class="sxs-lookup"><span data-stu-id="224be-135">The `AddConsole` and `AddDebug` extension methods are defined in the [Microsoft.Extensions.Logging.Console](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Console/) and [Microsoft.Extensions.Logging.Debug](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Debug/) packages.</span></span> <span data-ttu-id="224be-136">每个扩展方法都调用 `ILoggerFactory.AddProvider` 方法，传入提供程序的一个实例。</span><span class="sxs-lookup"><span data-stu-id="224be-136">Each extension method calls the `ILoggerFactory.AddProvider` method, passing in an instance of the provider.</span></span> 

> [!NOTE]
> <span data-ttu-id="224be-137">本文的示例应用程序将在 `Startup` 类的 `Configure` 方法中添加日志记录提供程序。</span><span class="sxs-lookup"><span data-stu-id="224be-137">The sample application for this article adds logging providers in the `Configure` method of the `Startup` class.</span></span> <span data-ttu-id="224be-138">要从先前执行的代码获取日志输出，请改为在 `Startup` 类构造函数中添加日志记录提供程序。</span><span class="sxs-lookup"><span data-stu-id="224be-138">If you want to get log output from code that executes earlier, add logging providers in the `Startup` class constructor instead.</span></span> 

---

<span data-ttu-id="224be-139">本文稍后部分介绍了每个[内置日志记录提供程序](#built-in-logging-providers)，并提供了[第三方日志记录提供程序](#third-party-logging-providers)的链接。</span><span class="sxs-lookup"><span data-stu-id="224be-139">You'll find information about each [built-in logging provider](#built-in-logging-providers) and links to [third-party logging providers](#third-party-logging-providers) later in the article.</span></span>

## <a name="sample-logging-output"></a><span data-ttu-id="224be-140">日志记录输出示例</span><span class="sxs-lookup"><span data-stu-id="224be-140">Sample logging output</span></span>

<span data-ttu-id="224be-141">从命令行运行上一部分所示的示例代码时，将在控制台中看到日志。</span><span class="sxs-lookup"><span data-stu-id="224be-141">With the sample code shown in the preceding section, you'll see logs in the console when you run from the command line.</span></span> <span data-ttu-id="224be-142">以下是控制台输出示例：</span><span class="sxs-lookup"><span data-stu-id="224be-142">Here's an example of console output:</span></span>

```console
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[1]
      Request starting HTTP/1.1 GET http://localhost:5000/api/todo/0
info: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[1]
      Executing action method TodoApi.Controllers.TodoController.GetById (TodoApi) with arguments (0) - ModelState is Valid
info: TodoApi.Controllers.TodoController[1002]
      Getting item 0
warn: TodoApi.Controllers.TodoController[4000]
      GetById(0) NOT FOUND
info: Microsoft.AspNetCore.Mvc.StatusCodeResult[1]
      Executing HttpStatusCodeResult, setting HTTP status code 404
info: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[2]
      Executed action TodoApi.Controllers.TodoController.GetById (TodoApi) in 42.9286ms
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[2]
      Request finished in 148.889ms 404
```
 
<span data-ttu-id="224be-143">转到 `http://localhost:5000/api/todo/0`，触发上一部分所示的两个 `ILogger` 调用的执行，创建了以上日志。</span><span class="sxs-lookup"><span data-stu-id="224be-143">These logs were created by going to `http://localhost:5000/api/todo/0`, which triggers execution of both `ILogger` calls shown in the preceding section.</span></span>

<span data-ttu-id="224be-144">在 Visual Studio 中运行示例应用程序时，“调试”窗口中将显示如下日志：</span><span class="sxs-lookup"><span data-stu-id="224be-144">Here's an example of the same logs as they appear in the Debug window when you run the sample application in Visual Studio:</span></span>

```console
Microsoft.AspNetCore.Hosting.Internal.WebHost:Information: Request starting HTTP/1.1 GET http://localhost:53104/api/todo/0  
Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker:Information: Executing action method TodoApi.Controllers.TodoController.GetById (TodoApi) with arguments (0) - ModelState is Valid
TodoApi.Controllers.TodoController:Information: Getting item 0
TodoApi.Controllers.TodoController:Warning: GetById(0) NOT FOUND
Microsoft.AspNetCore.Mvc.StatusCodeResult:Information: Executing HttpStatusCodeResult, setting HTTP status code 404
Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker:Information: Executed action TodoApi.Controllers.TodoController.GetById (TodoApi) in 152.5657ms
Microsoft.AspNetCore.Hosting.Internal.WebHost:Information: Request finished in 316.3195ms 404 
```

<span data-ttu-id="224be-145">由上一部分所示的 `ILogger` 调用创建的日志以“TodoApi.Controllers.TodoController”开头的。</span><span class="sxs-lookup"><span data-stu-id="224be-145">The logs that were created by the `ILogger` calls shown in the preceding section begin with "TodoApi.Controllers.TodoController".</span></span> <span data-ttu-id="224be-146">以“Microsoft”类别开头的日志来自 ASP.NET Core。</span><span class="sxs-lookup"><span data-stu-id="224be-146">The logs that begin with "Microsoft" categories are from ASP.NET Core.</span></span> <span data-ttu-id="224be-147">ASP.NET Core 本身和应用程序代码使用相同的日志记录 API 和日志记录提供程序。</span><span class="sxs-lookup"><span data-stu-id="224be-147">ASP.NET Core itself and your application code are using the same logging API and the same logging providers.</span></span>

<span data-ttu-id="224be-148">本文余下部分将介绍有关日志记录的某些详细信息及选项。</span><span class="sxs-lookup"><span data-stu-id="224be-148">The remainder of this article explains some details and options for logging.</span></span>

## <a name="nuget-packages"></a><span data-ttu-id="224be-149">NuGet 包</span><span class="sxs-lookup"><span data-stu-id="224be-149">NuGet packages</span></span>

<span data-ttu-id="224be-150">`ILogger` 和 `ILoggerFactory` 接口位于 [Microsoft.Extensions.Logging.Abstractions](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Abstractions/) 中，其默认实现位于 [Microsoft.Extensions.Logging](https://www.nuget.org/packages/Microsoft.Extensions.Logging/) 中。</span><span class="sxs-lookup"><span data-stu-id="224be-150">The `ILogger` and `ILoggerFactory` interfaces are in [Microsoft.Extensions.Logging.Abstractions](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Abstractions/), and default implementations for them are in [Microsoft.Extensions.Logging](https://www.nuget.org/packages/Microsoft.Extensions.Logging/).</span></span>

## <a name="log-category"></a><span data-ttu-id="224be-151">日志类别</span><span class="sxs-lookup"><span data-stu-id="224be-151">Log category</span></span>

<span data-ttu-id="224be-152">所创建的每个日志都包含一个类别。</span><span class="sxs-lookup"><span data-stu-id="224be-152">A *category* is included with each log that you create.</span></span> <span data-ttu-id="224be-153">在创建 `ILogger` 对象时指定类别。</span><span class="sxs-lookup"><span data-stu-id="224be-153">You specify the category when you create an `ILogger` object.</span></span> <span data-ttu-id="224be-154">类别可以是任意字符串，但约定使用写入日志的类的完全限定名称。</span><span class="sxs-lookup"><span data-stu-id="224be-154">The category may be any string, but a convention is to use the fully qualified name of the class from which the logs are written.</span></span> <span data-ttu-id="224be-155">例如“TodoApi.Controllers.TodoController”。</span><span class="sxs-lookup"><span data-stu-id="224be-155">For example: "TodoApi.Controllers.TodoController".</span></span>

<span data-ttu-id="224be-156">可以将类别指定为字符串，或使用从类型派生类别的扩展方法。</span><span class="sxs-lookup"><span data-stu-id="224be-156">You can specify the category as a string or use an extension method that derives the category from the type.</span></span> <span data-ttu-id="224be-157">要将类别指定为字符串，请在 `ILoggerFactory` 实例上调用 `CreateLogger`，如下所示。</span><span class="sxs-lookup"><span data-stu-id="224be-157">To specify the category as a string, call `CreateLogger` on an `ILoggerFactory` instance, as shown below.</span></span>

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_CreateLogger&highlight=7,10)]

<span data-ttu-id="224be-158">如下方示例所示，在大多数情况下使用 `ILogger<T>` 更简单。</span><span class="sxs-lookup"><span data-stu-id="224be-158">Most of the time, it will be easier to use `ILogger<T>`, as in the following example.</span></span>

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_LoggerDI&highlight=7)]

<span data-ttu-id="224be-159">这相当于使用 `T` 的完全限定类型名称来调用 `CreateLogger`。</span><span class="sxs-lookup"><span data-stu-id="224be-159">This is equivalent to calling `CreateLogger` with the fully qualified type name of `T`.</span></span>

## <a name="log-level"></a><span data-ttu-id="224be-160">日志级别</span><span class="sxs-lookup"><span data-stu-id="224be-160">Log level</span></span>

<span data-ttu-id="224be-161">每次写入日志时都需指定其 [LogLevel](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.loglevel)。</span><span class="sxs-lookup"><span data-stu-id="224be-161">Each time you write a log, you specify its [LogLevel](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.loglevel).</span></span> <span data-ttu-id="224be-162">日志级别指示严重性或重要程度。</span><span class="sxs-lookup"><span data-stu-id="224be-162">The log level indicates the degree of severity or importance.</span></span> <span data-ttu-id="224be-163">例如，如果方法正常结束则写入 `Information` 日志，如果方法返回 404 返回代码则写入 `Warning` 日志，如果捕获未知异常则写入 `Error` 日志。</span><span class="sxs-lookup"><span data-stu-id="224be-163">For example, you might write an `Information` log when a method ends normally, a `Warning` log when a method returns a 404 return code, and an `Error` log when you catch an unexpected exception.</span></span>

<span data-ttu-id="224be-164">在下列代码示例中，由方法的名称（如 `LogWarning`）指定日志级别。</span><span class="sxs-lookup"><span data-stu-id="224be-164">In the following code example, the names of the methods (for example, `LogWarning`) specify the log level.</span></span> <span data-ttu-id="224be-165">第一个参数是[日志事件 ID](#log-event-id)。</span><span class="sxs-lookup"><span data-stu-id="224be-165">The first parameter is the [Log event ID](#log-event-id).</span></span> <span data-ttu-id="224be-166">第二个形参是[消息模板](#log-message-template)，包含由剩余方法形参提供的实参值的占位符。</span><span class="sxs-lookup"><span data-stu-id="224be-166">The second parameter is a [message template](#log-message-template) with placeholders for argument values provided by the remaining method parameters.</span></span> <span data-ttu-id="224be-167">本文稍后部分更详细地介绍了方法参数。</span><span class="sxs-lookup"><span data-stu-id="224be-167">The method parameters are explained in more detail later in this article.</span></span>

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_CallLogMethods&highlight=3,7)]

<span data-ttu-id="224be-168">在方法名称中包含级别的日志方法是 [ILogger 的扩展方法](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.loggerextensions)。</span><span class="sxs-lookup"><span data-stu-id="224be-168">Log methods that include the level in the method name are [extension methods for ILogger](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.loggerextensions).</span></span> <span data-ttu-id="224be-169">在后台，这些方法调用可接受 `LogLevel` 参数的 `Log` 方法。</span><span class="sxs-lookup"><span data-stu-id="224be-169">Behind the scenes, these methods call a `Log` method that takes a `LogLevel` parameter.</span></span> <span data-ttu-id="224be-170">可直接调用 `Log` 方法而不调用其中某个扩展方法，但语法相对复杂。</span><span class="sxs-lookup"><span data-stu-id="224be-170">You can call the `Log` method directly rather than one of these extension methods, but the syntax is relatively complicated.</span></span> <span data-ttu-id="224be-171">有关详细信息，请参阅 [ILogger 接口](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.ilogger)和[记录器扩展源代码](https://github.com/aspnet/Logging/blob/master/src/Microsoft.Extensions.Logging.Abstractions/LoggerExtensions.cs)。</span><span class="sxs-lookup"><span data-stu-id="224be-171">For more information, see the [ILogger interface](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.ilogger) and the [logger extensions source code](https://github.com/aspnet/Logging/blob/master/src/Microsoft.Extensions.Logging.Abstractions/LoggerExtensions.cs).</span></span>

<span data-ttu-id="224be-172">ASP.NET Core 定义了以下[日志级别](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.loglevel)（按严重性从低到高排列）。</span><span class="sxs-lookup"><span data-stu-id="224be-172">ASP.NET Core defines the following [log levels](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.loglevel), ordered here from least to highest severity.</span></span>

* <span data-ttu-id="224be-173">跟踪 = 0</span><span class="sxs-lookup"><span data-stu-id="224be-173">Trace = 0</span></span>

  <span data-ttu-id="224be-174">表示仅对于开发人员调试问题有价值的信息。</span><span class="sxs-lookup"><span data-stu-id="224be-174">For information that is valuable only to a developer debugging an issue.</span></span> <span data-ttu-id="224be-175">这些消息可能包含敏感应用程序数据，因此请勿在生产环境中启用它们。</span><span class="sxs-lookup"><span data-stu-id="224be-175">These messages may contain sensitive application data and so should not be enabled in a production environment.</span></span> <span data-ttu-id="224be-176">默认情况下禁用。</span><span class="sxs-lookup"><span data-stu-id="224be-176">*Disabled by default.*</span></span> <span data-ttu-id="224be-177">示例：`Credentials: {"User":"someuser", "Password":"P@ssword"}`</span><span class="sxs-lookup"><span data-stu-id="224be-177">Example: `Credentials: {"User":"someuser", "Password":"P@ssword"}`</span></span>

* <span data-ttu-id="224be-178">调试 = 1</span><span class="sxs-lookup"><span data-stu-id="224be-178">Debug = 1</span></span>

  <span data-ttu-id="224be-179">表示在开发和调试过程中短期有用的信息。</span><span class="sxs-lookup"><span data-stu-id="224be-179">For information that has short-term usefulness during development and debugging.</span></span> <span data-ttu-id="224be-180">示例：`Entering method Configure with flag set to true.` 除非要进行故障排除，否则由于日志的数量过多，通常不在生产中启用 `Debug` 级别日志。</span><span class="sxs-lookup"><span data-stu-id="224be-180">Example: `Entering method Configure with flag set to true.` You typically would not enable `Debug` level logs in production unless you are troubleshooting, due to the high volume of logs.</span></span>

* <span data-ttu-id="224be-181">信息 = 2</span><span class="sxs-lookup"><span data-stu-id="224be-181">Information = 2</span></span>

  <span data-ttu-id="224be-182">用于跟踪应用程序的常规流。</span><span class="sxs-lookup"><span data-stu-id="224be-182">For tracking the general flow of the application.</span></span> <span data-ttu-id="224be-183">这些日志通常有长期价值。</span><span class="sxs-lookup"><span data-stu-id="224be-183">These logs typically have some long-term value.</span></span> <span data-ttu-id="224be-184">示例：`Request received for path /api/todo`</span><span class="sxs-lookup"><span data-stu-id="224be-184">Example: `Request received for path /api/todo`</span></span>

* <span data-ttu-id="224be-185">警告 = 3</span><span class="sxs-lookup"><span data-stu-id="224be-185">Warning = 3</span></span>

  <span data-ttu-id="224be-186">表示应用程序流中的异常或意外事件。</span><span class="sxs-lookup"><span data-stu-id="224be-186">For abnormal or unexpected events in the application flow.</span></span> <span data-ttu-id="224be-187">可包括不会中断应用程序运行但仍需调查的错误或其他条件。</span><span class="sxs-lookup"><span data-stu-id="224be-187">These may include errors or other conditions that do not cause the application to stop, but which may need to be investigated.</span></span> <span data-ttu-id="224be-188">`Warning` 日志级别常用于已处理的异常。</span><span class="sxs-lookup"><span data-stu-id="224be-188">Handled exceptions are a common place to use the `Warning` log level.</span></span> <span data-ttu-id="224be-189">示例：`FileNotFoundException for file quotes.txt.`</span><span class="sxs-lookup"><span data-stu-id="224be-189">Example: `FileNotFoundException for file quotes.txt.`</span></span>

* <span data-ttu-id="224be-190">错误 = 4</span><span class="sxs-lookup"><span data-stu-id="224be-190">Error = 4</span></span>

  <span data-ttu-id="224be-191">表示无法处理的错误和异常。</span><span class="sxs-lookup"><span data-stu-id="224be-191">For errors and exceptions that cannot be handled.</span></span> <span data-ttu-id="224be-192">这些消息指示的是当前活动或操作（如当前 HTTP 请求）中的失败，而不是应用程序范围的失败。</span><span class="sxs-lookup"><span data-stu-id="224be-192">These messages indicate a failure in the current activity or operation (such as the current HTTP request), not an application-wide failure.</span></span> <span data-ttu-id="224be-193">日志消息示例：`Cannot insert record due to duplicate key violation.`</span><span class="sxs-lookup"><span data-stu-id="224be-193">Example log message: `Cannot insert record due to duplicate key violation.`</span></span>

* <span data-ttu-id="224be-194">严重 = 5</span><span class="sxs-lookup"><span data-stu-id="224be-194">Critical = 5</span></span>

  <span data-ttu-id="224be-195">需要立即关注的失败。</span><span class="sxs-lookup"><span data-stu-id="224be-195">For failures that require immediate attention.</span></span> <span data-ttu-id="224be-196">例如数据丢失、磁盘空间不足。</span><span class="sxs-lookup"><span data-stu-id="224be-196">Examples: data loss scenarios, out of disk space.</span></span>

<span data-ttu-id="224be-197">可以使用日志级别控制写入到特定存储介质或显示窗口的日志输出量。</span><span class="sxs-lookup"><span data-stu-id="224be-197">You can use the log level to control how much log output is written to a particular storage medium or display window.</span></span> <span data-ttu-id="224be-198">例如在生产中，可将所有 `Information` 及以下级别的日志放在卷数据存储，将所有 `Warning` 及以上级别的日志放在值数据存储。</span><span class="sxs-lookup"><span data-stu-id="224be-198">For example, in production you might want all logs of `Information` level and lower to go to a volume data store, and all logs of `Warning` level and higher to go to a value data store.</span></span> <span data-ttu-id="224be-199">在开发期间，通常要将严重性为 `Warning` 或以上级别的日志发送到控制台。</span><span class="sxs-lookup"><span data-stu-id="224be-199">During development, you might normally send logs of `Warning` or higher severity to the console.</span></span> <span data-ttu-id="224be-200">需要进行故障排除时，可添加 `Debug` 级别。</span><span class="sxs-lookup"><span data-stu-id="224be-200">Then when you need to troubleshoot, you can add `Debug` level.</span></span> <span data-ttu-id="224be-201">本文稍后的[日志筛选](#log-filtering)部分介绍如何控制提供程序处理的日志级别。</span><span class="sxs-lookup"><span data-stu-id="224be-201">The [Log filtering](#log-filtering) section later in this article explains how to control which log levels a provider handles.</span></span>

<span data-ttu-id="224be-202">ASP.NET Core 框架为框架事件写入 `Debug` 级别日志。</span><span class="sxs-lookup"><span data-stu-id="224be-202">The ASP.NET Core framework writes `Debug` level logs for framework events.</span></span> <span data-ttu-id="224be-203">本文前面部分提供的日志示例排除了低于 `Information` 级别的日志，因此未显示 `Debug` 级别日志。</span><span class="sxs-lookup"><span data-stu-id="224be-203">The log examples earlier in this article excluded logs below `Information` level, so no `Debug` level logs were shown.</span></span> <span data-ttu-id="224be-204">如果运行配置为显示控制台提供程序 `Debug` 及以上级别的日志的示例应用程序，则控制台日志示例如下所示。</span><span class="sxs-lookup"><span data-stu-id="224be-204">Here's an example of console logs if you run the sample application configured to show `Debug` and higher logs for the console provider.</span></span>

```console
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[1]
      Request starting HTTP/1.1 GET http://localhost:62555/api/todo/0
dbug: Microsoft.AspNetCore.Routing.Tree.TreeRouter[1]
      Request successfully matched the route with name 'GetTodo' and template 'api/Todo/{id}'.
dbug: Microsoft.AspNetCore.Mvc.Internal.ActionSelector[2]
      Action 'TodoApi.Controllers.TodoController.Update (TodoApi)' with id '089d59b6-92ec-472d-b552-cc613dfd625d' did not match the constraint 'Microsoft.AspNetCore.Mvc.Internal.HttpMethodActionConstraint'
dbug: Microsoft.AspNetCore.Mvc.Internal.ActionSelector[2]
      Action 'TodoApi.Controllers.TodoController.Delete (TodoApi)' with id 'f3476abe-4bd9-4ad3-9261-3ead09607366' did not match the constraint 'Microsoft.AspNetCore.Mvc.Internal.HttpMethodActionConstraint'
dbug: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[1]
      Executing action TodoApi.Controllers.TodoController.GetById (TodoApi)
info: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[1]
      Executing action method TodoApi.Controllers.TodoController.GetById (TodoApi) with arguments (0) - ModelState is Valid
info: TodoApi.Controllers.TodoController[1002]
      Getting item 0
warn: TodoApi.Controllers.TodoController[4000]
      GetById(0) NOT FOUND
dbug: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[2]
      Executed action method TodoApi.Controllers.TodoController.GetById (TodoApi), returned result Microsoft.AspNetCore.Mvc.NotFoundResult.
info: Microsoft.AspNetCore.Mvc.StatusCodeResult[1]
      Executing HttpStatusCodeResult, setting HTTP status code 404
info: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[2]
      Executed action TodoApi.Controllers.TodoController.GetById (TodoApi) in 0.8788ms
dbug: Microsoft.AspNetCore.Server.Kestrel[9]
      Connection id "0HL6L7NEFF2QD" completed keep alive response.
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[2]
      Request finished in 2.7286ms 404
```

## <a name="log-event-id"></a><span data-ttu-id="224be-205">日志事件 ID</span><span class="sxs-lookup"><span data-stu-id="224be-205">Log event ID</span></span>

<span data-ttu-id="224be-206">每次写入日志时都可指定一个事件 ID。</span><span class="sxs-lookup"><span data-stu-id="224be-206">Each time you write a log, you can specify an *event ID*.</span></span> <span data-ttu-id="224be-207">该示例应用通过使用本地定义的 `LoggingEvents` 类来执行此操作：</span><span class="sxs-lookup"><span data-stu-id="224be-207">The sample app does this by using a locally-defined `LoggingEvents` class:</span></span>

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_CallLogMethods&highlight=3,7)]

[!code-csharp[](index/sample//Core/LoggingEvents.cs?name=snippet_LoggingEvents)]

<span data-ttu-id="224be-208">事件 ID 是一个整数值，可用于关联多组已记录事件。</span><span class="sxs-lookup"><span data-stu-id="224be-208">An event ID is an integer value that you can use to associate a set of logged events with one another.</span></span> <span data-ttu-id="224be-209">例如，向购物车添加项的日志可以是事件 ID 1000，完成购买的日志可以是事件 ID 1001。</span><span class="sxs-lookup"><span data-stu-id="224be-209">For instance, a log for adding an item to a shopping cart could be event ID 1000 and a log for completing a purchase could be event ID 1001.</span></span>

<span data-ttu-id="224be-210">在日志记录输出中，事件 ID 既可存储在字段里，也可包含在文本消息内，这由提供程序来决定。</span><span class="sxs-lookup"><span data-stu-id="224be-210">In logging output, the event ID may be stored in a field or included in the text message, depending on the provider.</span></span> <span data-ttu-id="224be-211">调试提供程序不显示事件 ID，但控制台提供程序会将其显示在类别后的括号里：</span><span class="sxs-lookup"><span data-stu-id="224be-211">The Debug provider doesn't show event IDs, but the console provider shows them in brackets after the category:</span></span>

```console
info: TodoApi.Controllers.TodoController[1002]
      Getting item invalidid
warn: TodoApi.Controllers.TodoController[4000]
      GetById(invalidid) NOT FOUND
```

## <a name="log-message-template"></a><span data-ttu-id="224be-212">日志消息模板</span><span class="sxs-lookup"><span data-stu-id="224be-212">Log message template</span></span>

<span data-ttu-id="224be-213">每次写入日志消息都会提供一个消息模板。</span><span class="sxs-lookup"><span data-stu-id="224be-213">Each time you write a log message, you provide a message template.</span></span> <span data-ttu-id="224be-214">消息模板可以是字符串，也可以包含放置参数值的命名占位符。</span><span class="sxs-lookup"><span data-stu-id="224be-214">The message template can be a string or it can contain named placeholders into which argument values are placed.</span></span> <span data-ttu-id="224be-215">该模板不是格式字符串，应对占位符进行命名而不是编号。</span><span class="sxs-lookup"><span data-stu-id="224be-215">The template isn't a format string, and placeholders should be named, not numbered.</span></span>

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_CallLogMethods&highlight=3,7)]

<span data-ttu-id="224be-216">占位符的顺序（而非其名称）决定了为其提供值的参数。</span><span class="sxs-lookup"><span data-stu-id="224be-216">The order of placeholders, not their names, determines which parameters are used to provide their values.</span></span> <span data-ttu-id="224be-217">如果你有以下代码：</span><span class="sxs-lookup"><span data-stu-id="224be-217">If you have the following code:</span></span>

```csharp
string p1 = "parm1";
string p2 = "parm2";
_logger.LogInformation("Parameter values: {p2}, {p1}", p1, p2);
```

<span data-ttu-id="224be-218">生成的日志消息如下所示：</span><span class="sxs-lookup"><span data-stu-id="224be-218">The resulting log message looks like this:</span></span>

```
Parameter values: parm1, parm2
```

<span data-ttu-id="224be-219">记录框架以这种方式对消息进行格式化，以便日志记录提供程序能实现[语义日志记录（又称结构化日志记录）](https://softwareengineering.stackexchange.com/questions/312197/benefits-of-structured-logging-vs-basic-logging)。</span><span class="sxs-lookup"><span data-stu-id="224be-219">The logging framework does message formatting in this way to make it possible for logging providers to implement [semantic logging, also known as structured logging](https://softwareengineering.stackexchange.com/questions/312197/benefits-of-structured-logging-vs-basic-logging).</span></span> <span data-ttu-id="224be-220">由于传递到日志记录系统的是参数本身（而不仅仅是格式化的消息模板），因此除消息模板外，日志记录提供程序还可将参数值存储为字段。</span><span class="sxs-lookup"><span data-stu-id="224be-220">Because the arguments themselves are passed to the logging system, not just the formatted message template, logging providers can store the parameter values as fields in addition to the message template.</span></span> <span data-ttu-id="224be-221">如果将日志输出定向到 Azure 表存储，则记录器方法调用将如下所示：</span><span class="sxs-lookup"><span data-stu-id="224be-221">If you're directing your log output to Azure Table Storage and your logger method call looks like this:</span></span>

```csharp
_logger.LogInformation("Getting item {ID} at {RequestTime}", id, DateTime.Now);
```

<span data-ttu-id="224be-222">每个 Azure 表实体都具有 `ID` 和 `RequestTime` 属性，简化了对日志数据的查询。</span><span class="sxs-lookup"><span data-stu-id="224be-222">Each Azure Table entity can have `ID` and `RequestTime` properties, which simplifies queries on log data.</span></span> <span data-ttu-id="224be-223">无需分析文本消息的超时，即可找到特定 `RequestTime` 范围内的全部日志。</span><span class="sxs-lookup"><span data-stu-id="224be-223">You can find all logs within a particular `RequestTime` range without the need to parse the time out of the text message.</span></span>

## <a name="logging-exceptions"></a><span data-ttu-id="224be-224">日志记录异常</span><span class="sxs-lookup"><span data-stu-id="224be-224">Logging exceptions</span></span>

<span data-ttu-id="224be-225">记录器方法有可传入异常的重载，如下方示例所示：</span><span class="sxs-lookup"><span data-stu-id="224be-225">The logger methods have overloads that let you pass in an exception, as in the following example:</span></span>

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_LogException&highlight=3)]

<span data-ttu-id="224be-226">不同的提供程序处理异常信息的方式不同。</span><span class="sxs-lookup"><span data-stu-id="224be-226">Different providers handle the exception information in different ways.</span></span> <span data-ttu-id="224be-227">以下是上示代码的调试提供程序输出示例。</span><span class="sxs-lookup"><span data-stu-id="224be-227">Here's an example of Debug provider output from the code shown above.</span></span>

```
TodoApi.Controllers.TodoController:Warning: GetById(036dd898-fb01-47e8-9a65-f92eb73cf924) NOT FOUND

System.Exception: Item not found exception.
 at TodoApi.Controllers.TodoController.GetById(String id) in C:\logging\sample\src\TodoApi\Controllers\TodoController.cs:line 226
```

## <a name="log-filtering"></a><span data-ttu-id="224be-228">日志筛选</span><span class="sxs-lookup"><span data-stu-id="224be-228">Log filtering</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="224be-229">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="224be-229">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="224be-230">可为特定或所有提供程序和类别指定最低日志级别。</span><span class="sxs-lookup"><span data-stu-id="224be-230">You can specify a minimum log level for a specific provider and category or for all providers or all categories.</span></span> <span data-ttu-id="224be-231">最低级别以下的日志不会传递给该提供程序，因此不会显示或存储它们。</span><span class="sxs-lookup"><span data-stu-id="224be-231">Any logs below the minimum level aren't passed to that provider, so they don't get displayed or stored.</span></span> 

<span data-ttu-id="224be-232">要禁止显示所有日志，可将 `LogLevel.None` 指定为最低日志级别。</span><span class="sxs-lookup"><span data-stu-id="224be-232">If you want to suppress all logs, you can specify `LogLevel.None` as the minimum log level.</span></span> <span data-ttu-id="224be-233">`LogLevel.None` 的整数值为 6，它大于 `LogLevel.Critical` (5)。</span><span class="sxs-lookup"><span data-stu-id="224be-233">The integer value of `LogLevel.None` is 6, which is higher than `LogLevel.Critical` (5).</span></span>

<span data-ttu-id="224be-234">**在配置中创建筛选规则**</span><span class="sxs-lookup"><span data-stu-id="224be-234">**Create filter rules in configuration**</span></span>

<span data-ttu-id="224be-235">项目模板创建调用 `CreateDefaultBuilder` 的代码，以便为控制台和调试提供程序设置日志记录。</span><span class="sxs-lookup"><span data-stu-id="224be-235">The project templates create code that calls `CreateDefaultBuilder` to set up logging for the Console and Debug providers.</span></span> <span data-ttu-id="224be-236">`CreateDefaultBuilder` 方法还使用如下所示的代码，设置日志记录以查找 `Logging` 部分的配置：</span><span class="sxs-lookup"><span data-stu-id="224be-236">The `CreateDefaultBuilder` method also sets up logging to look for configuration in a `Logging` section, using code like the following:</span></span>

[!code-csharp[](index/sample2/Program.cs?name=snippet_ExpandDefault&highlight=15)]

<span data-ttu-id="224be-237">配置数据按提供程序和类别指定最低日志级别，如下方示例所示：</span><span class="sxs-lookup"><span data-stu-id="224be-237">The configuration data specifies minimum log levels by provider and category, as in the following example:</span></span>

[!code-json[](index/sample2/appsettings.json)]

<span data-ttu-id="224be-238">此 JSON 将创建六条筛选规则，一条用于调试提供程序，四条用于控制台提供程序，一条用于所有提供程序。</span><span class="sxs-lookup"><span data-stu-id="224be-238">This JSON creates six filter rules, one for the Debug provider, four for the Console provider, and one that applies to all providers.</span></span> <span data-ttu-id="224be-239">稍后你将了解在创建 `ILogger` 对象后，如何为每个提供程序从上述规则中选择其一。</span><span class="sxs-lookup"><span data-stu-id="224be-239">You'll see later how just one of these rules is chosen for each provider when an `ILogger` object is created.</span></span>

<span data-ttu-id="224be-240">**代码中的筛选规则**</span><span class="sxs-lookup"><span data-stu-id="224be-240">**Filter rules in code**</span></span>

<span data-ttu-id="224be-241">可在代码中注册筛选规则，如下方示例所示：</span><span class="sxs-lookup"><span data-stu-id="224be-241">You can register filter rules in code, as shown in the following example:</span></span>

[!code-csharp[](index/sample2/Program.cs?name=snippet_FilterInCode&highlight=4-5)]

<span data-ttu-id="224be-242">第二个 `AddFilter` 使用类型名称来指定调试提供程序。</span><span class="sxs-lookup"><span data-stu-id="224be-242">The second `AddFilter` specifies the Debug provider by using its type name.</span></span> <span data-ttu-id="224be-243">第一个 `AddFilter` 应用于全部提供程序，因为它未指定提供程序类型。</span><span class="sxs-lookup"><span data-stu-id="224be-243">The first `AddFilter` applies to all providers because it doesn't specify a provider type.</span></span>

<span data-ttu-id="224be-244">**如何筛选已应用的规则**</span><span class="sxs-lookup"><span data-stu-id="224be-244">**How filtering rules are applied**</span></span>

<span data-ttu-id="224be-245">先前示例中显示的配置数据和 `AddFilter` 代码会创建下表所示的规则。</span><span class="sxs-lookup"><span data-stu-id="224be-245">The configuration data and the `AddFilter` code shown in the preceding examples create the rules shown in the following table.</span></span> <span data-ttu-id="224be-246">前六条由配置示例创建，后两条由代码示例创建。</span><span class="sxs-lookup"><span data-stu-id="224be-246">The first six come from the configuration example and the last two come from the code example.</span></span>

| <span data-ttu-id="224be-247">数字</span><span class="sxs-lookup"><span data-stu-id="224be-247">Number</span></span> | <span data-ttu-id="224be-248">提供程序</span><span class="sxs-lookup"><span data-stu-id="224be-248">Provider</span></span>      | <span data-ttu-id="224be-249">类别的开头为...</span><span class="sxs-lookup"><span data-stu-id="224be-249">Categories that begin with ...</span></span>          | <span data-ttu-id="224be-250">最低日志级别</span><span class="sxs-lookup"><span data-stu-id="224be-250">Minimum log level</span></span> |
| :----: | ------------- | --------------------------------------- | ----------------- |
| <span data-ttu-id="224be-251">1</span><span class="sxs-lookup"><span data-stu-id="224be-251">1</span></span>      | <span data-ttu-id="224be-252">调试</span><span class="sxs-lookup"><span data-stu-id="224be-252">Debug</span></span>         | <span data-ttu-id="224be-253">全部类别</span><span class="sxs-lookup"><span data-stu-id="224be-253">All categories</span></span>                          | <span data-ttu-id="224be-254">信息</span><span class="sxs-lookup"><span data-stu-id="224be-254">Information</span></span>       |
| <span data-ttu-id="224be-255">2</span><span class="sxs-lookup"><span data-stu-id="224be-255">2</span></span>      | <span data-ttu-id="224be-256">控制台</span><span class="sxs-lookup"><span data-stu-id="224be-256">Console</span></span>       | <span data-ttu-id="224be-257">Microsoft.AspNetCore.Mvc.Razor.Internal</span><span class="sxs-lookup"><span data-stu-id="224be-257">Microsoft.AspNetCore.Mvc.Razor.Internal</span></span> | <span data-ttu-id="224be-258">警告</span><span class="sxs-lookup"><span data-stu-id="224be-258">Warning</span></span>           |
| <span data-ttu-id="224be-259">3</span><span class="sxs-lookup"><span data-stu-id="224be-259">3</span></span>      | <span data-ttu-id="224be-260">控制台</span><span class="sxs-lookup"><span data-stu-id="224be-260">Console</span></span>       | <span data-ttu-id="224be-261">Microsoft.AspNetCore.Mvc.Razor.Razor</span><span class="sxs-lookup"><span data-stu-id="224be-261">Microsoft.AspNetCore.Mvc.Razor.Razor</span></span>    | <span data-ttu-id="224be-262">调试</span><span class="sxs-lookup"><span data-stu-id="224be-262">Debug</span></span>             |
| <span data-ttu-id="224be-263">4</span><span class="sxs-lookup"><span data-stu-id="224be-263">4</span></span>      | <span data-ttu-id="224be-264">控制台</span><span class="sxs-lookup"><span data-stu-id="224be-264">Console</span></span>       | <span data-ttu-id="224be-265">Microsoft.AspNetCore.Mvc.Razor</span><span class="sxs-lookup"><span data-stu-id="224be-265">Microsoft.AspNetCore.Mvc.Razor</span></span>          | <span data-ttu-id="224be-266">错误</span><span class="sxs-lookup"><span data-stu-id="224be-266">Error</span></span>             |
| <span data-ttu-id="224be-267">5</span><span class="sxs-lookup"><span data-stu-id="224be-267">5</span></span>      | <span data-ttu-id="224be-268">控制台</span><span class="sxs-lookup"><span data-stu-id="224be-268">Console</span></span>       | <span data-ttu-id="224be-269">全部类别</span><span class="sxs-lookup"><span data-stu-id="224be-269">All categories</span></span>                          | <span data-ttu-id="224be-270">信息</span><span class="sxs-lookup"><span data-stu-id="224be-270">Information</span></span>       |
| <span data-ttu-id="224be-271">6</span><span class="sxs-lookup"><span data-stu-id="224be-271">6</span></span>      | <span data-ttu-id="224be-272">全部提供程序</span><span class="sxs-lookup"><span data-stu-id="224be-272">All providers</span></span> | <span data-ttu-id="224be-273">全部类别</span><span class="sxs-lookup"><span data-stu-id="224be-273">All categories</span></span>                          | <span data-ttu-id="224be-274">调试</span><span class="sxs-lookup"><span data-stu-id="224be-274">Debug</span></span>             |
| <span data-ttu-id="224be-275">7</span><span class="sxs-lookup"><span data-stu-id="224be-275">7</span></span>      | <span data-ttu-id="224be-276">全部提供程序</span><span class="sxs-lookup"><span data-stu-id="224be-276">All providers</span></span> | <span data-ttu-id="224be-277">系统</span><span class="sxs-lookup"><span data-stu-id="224be-277">System</span></span>                                  | <span data-ttu-id="224be-278">调试</span><span class="sxs-lookup"><span data-stu-id="224be-278">Debug</span></span>             |
| <span data-ttu-id="224be-279">8</span><span class="sxs-lookup"><span data-stu-id="224be-279">8</span></span>      | <span data-ttu-id="224be-280">调试</span><span class="sxs-lookup"><span data-stu-id="224be-280">Debug</span></span>         | <span data-ttu-id="224be-281">Microsoft</span><span class="sxs-lookup"><span data-stu-id="224be-281">Microsoft</span></span>                               | <span data-ttu-id="224be-282">跟踪</span><span class="sxs-lookup"><span data-stu-id="224be-282">Trace</span></span>             |

<span data-ttu-id="224be-283">创建 `ILogger` 对象来写入日志时，`ILoggerFactory` 对象将从根据提供程序选择一条规则，将其应用于该记录器。</span><span class="sxs-lookup"><span data-stu-id="224be-283">When you create an `ILogger` object to write logs with, the `ILoggerFactory` object selects a single rule per provider to apply to that logger.</span></span> <span data-ttu-id="224be-284">将按所选规则筛选该 `ILogger` 对象写入的所有消息。</span><span class="sxs-lookup"><span data-stu-id="224be-284">All messages written by that `ILogger` object are filtered based on the selected rules.</span></span> <span data-ttu-id="224be-285">从可用规则中为每个提供程序和类别对选择最具体的规则。</span><span class="sxs-lookup"><span data-stu-id="224be-285">The most specific rule possible for each provider and category pair is selected from the available rules.</span></span>

<span data-ttu-id="224be-286">在为给定的类别创建 `ILogger` 时，以下算法将用于每个提供程序：</span><span class="sxs-lookup"><span data-stu-id="224be-286">The following algorithm is used for each provider when an `ILogger` is created for a given category:</span></span>

* <span data-ttu-id="224be-287">选择匹配提供程序或其别名的所有规则。</span><span class="sxs-lookup"><span data-stu-id="224be-287">Select all rules that match the provider or its alias.</span></span> <span data-ttu-id="224be-288">如果找不到，则选择具有空提供程序的所有规则。</span><span class="sxs-lookup"><span data-stu-id="224be-288">If none are found, select all rules with an empty provider.</span></span>
* <span data-ttu-id="224be-289">根据上一步的结果，选择具有最长匹配类别前缀的规则。</span><span class="sxs-lookup"><span data-stu-id="224be-289">From the result of the preceding step, select rules with longest matching category prefix.</span></span> <span data-ttu-id="224be-290">如果找不到，则选择未指定类别的所有规则。</span><span class="sxs-lookup"><span data-stu-id="224be-290">If none are found, select all rules that don't specify a category.</span></span>
* <span data-ttu-id="224be-291">如果选择了多条规则，则采用最后一条。</span><span class="sxs-lookup"><span data-stu-id="224be-291">If multiple rules are selected take the **last** one.</span></span>
* <span data-ttu-id="224be-292">如果未选择任何规则，则使用 `MinimumLevel`。</span><span class="sxs-lookup"><span data-stu-id="224be-292">If no rules are selected, use `MinimumLevel`.</span></span>
 
<span data-ttu-id="224be-293">例如，假设你拥有上述规则列表，并为类别“Microsoft.AspNetCore.Mvc.Razor.RazorViewEngine”创建了 `ILogger` 对象：</span><span class="sxs-lookup"><span data-stu-id="224be-293">For example, suppose you have the preceding list of rules and you create an `ILogger` object for category "Microsoft.AspNetCore.Mvc.Razor.RazorViewEngine":</span></span>

* <span data-ttu-id="224be-294">对于调试提供程序，规则 1、6 和 8 适用。</span><span class="sxs-lookup"><span data-stu-id="224be-294">For the Debug provider, rules 1, 6, and 8 apply.</span></span> <span data-ttu-id="224be-295">规则 8 最为具体，因此选择了它。</span><span class="sxs-lookup"><span data-stu-id="224be-295">Rule 8 is most specific, so that's the one selected.</span></span>
* <span data-ttu-id="224be-296">对于控制台提供程序，符合的有规则 3、规则 4、规则 5 和规则 6。</span><span class="sxs-lookup"><span data-stu-id="224be-296">For the Console provider, rules 3, 4, 5, and 6 apply.</span></span> <span data-ttu-id="224be-297">规则 3 最为具体。</span><span class="sxs-lookup"><span data-stu-id="224be-297">Rule 3 is most specific.</span></span>

<span data-ttu-id="224be-298">在使用 `ILogger` 为类别“Microsoft.AspNetCore.Mvc.Razor.RazorViewEngine”创建日志时，`Trace` 及以上级别的日志将发送到调试提供程序，`Debug` 及以上级别的日志将发送到控制台提供程序。</span><span class="sxs-lookup"><span data-stu-id="224be-298">When you create logs with an `ILogger` for category "Microsoft.AspNetCore.Mvc.Razor.RazorViewEngine", logs of `Trace` level and above will go to the Debug provider, and logs of `Debug` level and above will go to the Console provider.</span></span>

<span data-ttu-id="224be-299">**提供程序别名**</span><span class="sxs-lookup"><span data-stu-id="224be-299">**Provider aliases**</span></span>

<span data-ttu-id="224be-300">可在配置中使用类型名称指定提供程序，但每个提供程序都定义了一个更短的别名，它更易于使用。</span><span class="sxs-lookup"><span data-stu-id="224be-300">You can use the type name to specify a provider in configuration, but each provider defines a shorter *alias* that is easier to use.</span></span> <span data-ttu-id="224be-301">对于内置提供程序，请使用以下别名：</span><span class="sxs-lookup"><span data-stu-id="224be-301">For the built-in providers, use the following aliases:</span></span>

- <span data-ttu-id="224be-302">控制台</span><span class="sxs-lookup"><span data-stu-id="224be-302">Console</span></span>
- <span data-ttu-id="224be-303">调试</span><span class="sxs-lookup"><span data-stu-id="224be-303">Debug</span></span>
- <span data-ttu-id="224be-304">EventLog</span><span class="sxs-lookup"><span data-stu-id="224be-304">EventLog</span></span>
- <span data-ttu-id="224be-305">AzureAppServices</span><span class="sxs-lookup"><span data-stu-id="224be-305">AzureAppServices</span></span>
- <span data-ttu-id="224be-306">TraceSource</span><span class="sxs-lookup"><span data-stu-id="224be-306">TraceSource</span></span>
- <span data-ttu-id="224be-307">EventSource</span><span class="sxs-lookup"><span data-stu-id="224be-307">EventSource</span></span>

<span data-ttu-id="224be-308">**默认最低级别**</span><span class="sxs-lookup"><span data-stu-id="224be-308">**Default minimum level**</span></span>

<span data-ttu-id="224be-309">仅当配置或代码中的规则对于给定提供程序或类别均不适用时，最低级别设置才会生效。</span><span class="sxs-lookup"><span data-stu-id="224be-309">There is a minimum level setting that takes effect only if no rules from configuration or code apply for a given provider and category.</span></span> <span data-ttu-id="224be-310">下面的示例演示如何设置最低级别：</span><span class="sxs-lookup"><span data-stu-id="224be-310">The following example shows how to set the minimum level:</span></span>

[!code-csharp[](index/sample2/Program.cs?name=snippet_MinLevel&highlight=3)]

<span data-ttu-id="224be-311">如果没有明确设置最低级别，则默认值为 `Information`，它表示 `Trace` 和 `Debug` 日志将被忽略。</span><span class="sxs-lookup"><span data-stu-id="224be-311">If you don't explicitly set the minimum level, the default value is `Information`, which means that `Trace` and `Debug` logs are ignored.</span></span>

<span data-ttu-id="224be-312">**筛选器函数**</span><span class="sxs-lookup"><span data-stu-id="224be-312">**Filter functions**</span></span>

<span data-ttu-id="224be-313">可向筛选器函数写入代码以应用筛选规则。</span><span class="sxs-lookup"><span data-stu-id="224be-313">You can write code in a filter function to apply filtering rules.</span></span> <span data-ttu-id="224be-314">对于配置或代码未将规则分配到的所有提供程序和类别，都将调用筛选器函数。</span><span class="sxs-lookup"><span data-stu-id="224be-314">A filter function is invoked for all providers and categories that do not have rules assigned to them by configuration or code.</span></span> <span data-ttu-id="224be-315">函数中的代码有权访问提供程序类型、类别和日志级别，以决定是否记录某条消息。</span><span class="sxs-lookup"><span data-stu-id="224be-315">Code in the function has access to the provider type, category, and log level to decide whether or not a message should be logged.</span></span> <span data-ttu-id="224be-316">例如: </span><span class="sxs-lookup"><span data-stu-id="224be-316">For example:</span></span>

[!code-csharp[](index/sample2/Program.cs?name=snippet_FilterFunction&highlight=5-13)]

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="224be-317">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="224be-317">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

<span data-ttu-id="224be-318">通过某些日志记录提供程序，可根据日志级别和类别指定何时向存储介质写入日志、何时忽略日志。</span><span class="sxs-lookup"><span data-stu-id="224be-318">Some logging providers let you specify when logs should be written to a storage medium or ignored based on log level and category.</span></span>

<span data-ttu-id="224be-319">`AddConsole` 和 `AddDebug` 扩展方法提供允许传入筛选条件的重载。</span><span class="sxs-lookup"><span data-stu-id="224be-319">The `AddConsole` and `AddDebug` extension methods provide overloads that let you pass in filtering criteria.</span></span> <span data-ttu-id="224be-320">下列示例代码将使控制台提供程序忽略低于 `Warning` 级别的日志，而使调试提供程序忽略由框架创建的日志。</span><span class="sxs-lookup"><span data-stu-id="224be-320">The following sample code causes the console provider to ignore logs below `Warning` level, while the Debug provider ignores logs that the framework creates.</span></span>

[!code-csharp[](index/sample/Startup.cs?name=snippet_AddConsoleAndDebugWithFilter&highlight=6-7)]

<span data-ttu-id="224be-321">`AddEventLog` 方法拥有接受 `EventLogSettings` 实例的重载，该实例的 `Filter` 属性中可能包含筛选函数。</span><span class="sxs-lookup"><span data-stu-id="224be-321">The `AddEventLog` method has an overload that takes an `EventLogSettings` instance, which may contain a filtering function in its `Filter` property.</span></span> <span data-ttu-id="224be-322">TraceSource 提供程序不提供以上任何重载，因为其日志记录级别和其他参数均基于它使用的 `SourceSwitch` 和 `TraceListener`。</span><span class="sxs-lookup"><span data-stu-id="224be-322">The TraceSource provider does not provide any of those overloads, since its logging level and other parameters are based on the `SourceSwitch` and `TraceListener` it uses.</span></span>

<span data-ttu-id="224be-323">可以使用 `WithFilter` 扩展方法为所有通过 `ILoggerFactory` 实例注册的提供程序设置筛选规则。</span><span class="sxs-lookup"><span data-stu-id="224be-323">You can set filtering rules for all providers that are registered with an `ILoggerFactory` instance by using the `WithFilter` extension method.</span></span> <span data-ttu-id="224be-324">以下示例将（类别以“Microsoft”或“System”开头的）框架日志限制为警告，并在调试级别记录应用日志。</span><span class="sxs-lookup"><span data-stu-id="224be-324">The example below limits framework logs (category begins with "Microsoft" or "System") to warnings while letting the app log at debug level.</span></span>

[!code-csharp[](index/sample/Startup.cs?name=snippet_FactoryFilter&highlight=6-11)]

<span data-ttu-id="224be-325">要通过筛选来防止写入某个特定类别的所有日志，可将 `LogLevel.None` 指定为该类别的最低日志级别。</span><span class="sxs-lookup"><span data-stu-id="224be-325">If you want to use filtering to prevent all logs from being written for a particular category, you can specify `LogLevel.None` as the minimum log level for that category.</span></span> <span data-ttu-id="224be-326">`LogLevel.None` 的整数值为 6，它大于 `LogLevel.Critical` (5)。</span><span class="sxs-lookup"><span data-stu-id="224be-326">The integer value of `LogLevel.None` is 6, which is higher than `LogLevel.Critical` (5).</span></span>

<span data-ttu-id="224be-327">`WithFilter` 扩展方法由 [Microsoft.Extensions.Logging.Filter](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Filter) NuGet 包提供。</span><span class="sxs-lookup"><span data-stu-id="224be-327">The `WithFilter` extension method is provided by the [Microsoft.Extensions.Logging.Filter](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Filter) NuGet package.</span></span> <span data-ttu-id="224be-328">该方法返回一个新的 `ILoggerFactory` 实例，该实例将筛选传递给注册的所有记录器提供程序的日志消息。</span><span class="sxs-lookup"><span data-stu-id="224be-328">The method returns a new `ILoggerFactory` instance that will filter the log messages passed to all logger providers registered with it.</span></span> <span data-ttu-id="224be-329">这不会影响其他任何 `ILoggerFactory` 实例（包括原始 `ILoggerFactory` 实例）。</span><span class="sxs-lookup"><span data-stu-id="224be-329">It does not affect any other `ILoggerFactory` instances, including the original `ILoggerFactory` instance.</span></span>

---

## <a name="log-scopes"></a><span data-ttu-id="224be-330">日志作用域</span><span class="sxs-lookup"><span data-stu-id="224be-330">Log scopes</span></span>

<span data-ttu-id="224be-331">可将逻辑操作集组合到作用域内，以便将相同的数据附加到在该集中创建的每个日志。</span><span class="sxs-lookup"><span data-stu-id="224be-331">You can group a set of logical operations within a *scope* in order to attach the same data to each log that is created as part of that set.</span></span> <span data-ttu-id="224be-332">例如，可让处理事务时创建的每个日志都包含事务 ID。</span><span class="sxs-lookup"><span data-stu-id="224be-332">For example, you might want every log created as part of processing a transaction to include the transaction ID.</span></span>

<span data-ttu-id="224be-333">作用域是由 `ILogger.BeginScope<TState>` 方法返回的 `IDisposable` 类型，在释放前将持续存在。</span><span class="sxs-lookup"><span data-stu-id="224be-333">A scope is an `IDisposable` type that is returned by the `ILogger.BeginScope<TState>` method and lasts until it is disposed.</span></span> <span data-ttu-id="224be-334">要使用作用域，请在 `using` 块中包装记录器调用，如下所示：</span><span class="sxs-lookup"><span data-stu-id="224be-334">You use a scope by wrapping your logger calls in a `using` block, as shown here:</span></span>

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_Scopes&highlight=4-5,13)]

<span data-ttu-id="224be-335">下列代码为控制台提供程序启用作用域：</span><span class="sxs-lookup"><span data-stu-id="224be-335">The following code enables scopes for the console provider:</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="224be-336">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="224be-336">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="224be-337">在 Program.cs 中：</span><span class="sxs-lookup"><span data-stu-id="224be-337">In *Program.cs*:</span></span>

[!code-csharp[](index/sample2/Program.cs?name=snippet_Scopes&highlight=4)]

> [!NOTE]
> <span data-ttu-id="224be-338">要启用基于作用域的日志记录，必须先配置 `IncludeScopes` 控制台记录器选项。</span><span class="sxs-lookup"><span data-stu-id="224be-338">Configuring the `IncludeScopes` console logger option is required to enable scope-based logging.</span></span> <span data-ttu-id="224be-339">ASP.NET Core 2.1 发布后，将支持使用 appsettings 配置文件来配置 `IncludeScopes`。</span><span class="sxs-lookup"><span data-stu-id="224be-339">Configuration of `IncludeScopes` using *appsettings* configuration files will be available with the release of ASP.NET Core 2.1.</span></span>

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="224be-340">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="224be-340">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

<span data-ttu-id="224be-341">在 Startup.cs 中：</span><span class="sxs-lookup"><span data-stu-id="224be-341">In *Startup.cs*:</span></span>

[!code-csharp[](index/sample/Startup.cs?name=snippet_Scopes&highlight=6)]

---

<span data-ttu-id="224be-342">每条日志消息都包含作用域内的信息：</span><span class="sxs-lookup"><span data-stu-id="224be-342">Each log message includes the scoped information:</span></span>

```
info: TodoApi.Controllers.TodoController[1002]
      => RequestId:0HKV9C49II9CK RequestPath:/api/todo/0 => TodoApi.Controllers.TodoController.GetById (TodoApi) => Message attached to logs created in the using block
      Getting item 0
warn: TodoApi.Controllers.TodoController[4000]
      => RequestId:0HKV9C49II9CK RequestPath:/api/todo/0 => TodoApi.Controllers.TodoController.GetById (TodoApi) => Message attached to logs created in the using block
      GetById(0) NOT FOUND
```

## <a name="built-in-logging-providers"></a><span data-ttu-id="224be-343">内置日志记录提供程序</span><span class="sxs-lookup"><span data-stu-id="224be-343">Built-in logging providers</span></span>

<span data-ttu-id="224be-344">ASP.NET Core 提供以下提供程序：</span><span class="sxs-lookup"><span data-stu-id="224be-344">ASP.NET Core ships the following providers:</span></span>

* [<span data-ttu-id="224be-345">控制台</span><span class="sxs-lookup"><span data-stu-id="224be-345">Console</span></span>](#console)
* [<span data-ttu-id="224be-346">调试</span><span class="sxs-lookup"><span data-stu-id="224be-346">Debug</span></span>](#debug)
* [<span data-ttu-id="224be-347">EventSource</span><span class="sxs-lookup"><span data-stu-id="224be-347">EventSource</span></span>](#eventsource)
* [<span data-ttu-id="224be-348">EventLog</span><span class="sxs-lookup"><span data-stu-id="224be-348">EventLog</span></span>](#eventlog)
* [<span data-ttu-id="224be-349">TraceSource</span><span class="sxs-lookup"><span data-stu-id="224be-349">TraceSource</span></span>](#tracesource)
* [<span data-ttu-id="224be-350">Azure 应用服务</span><span class="sxs-lookup"><span data-stu-id="224be-350">Azure App Service</span></span>](#appservice)

<a id="console"></a>
### <a name="the-console-provider"></a><span data-ttu-id="224be-351">控制台提供程序</span><span class="sxs-lookup"><span data-stu-id="224be-351">The console provider</span></span>

<span data-ttu-id="224be-352">[Microsoft.Extensions.Logging.Console](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Console) 提供程序包向控制台发送日志输出。</span><span class="sxs-lookup"><span data-stu-id="224be-352">The [Microsoft.Extensions.Logging.Console](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Console) provider package sends log output to the console.</span></span> 

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="224be-353">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="224be-353">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

```csharp
logging.AddConsole()
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="224be-354">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="224be-354">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

```csharp
loggerFactory.AddConsole()
```

<span data-ttu-id="224be-355">[AddConsole 重载](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.consoleloggerextensions)允许传入一个最低日志级别、一个筛选器函数，以及一个用于指示作用域是否受支持的布尔值。</span><span class="sxs-lookup"><span data-stu-id="224be-355">[AddConsole overloads](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.consoleloggerextensions) let you pass in an a minimum log level, a filter function, and a boolean that indicates whether scopes are supported.</span></span> <span data-ttu-id="224be-356">另一个选项是传递 `IConfiguration` 对象，可通过它来指定支持的作用域及日志记录级别。</span><span class="sxs-lookup"><span data-stu-id="224be-356">Another option is to pass in an `IConfiguration` object, which can specify scopes support and logging levels.</span></span> 

<span data-ttu-id="224be-357">在为生产环境选择控制台提供程序时，请考虑到它将对性能产生重大影响。</span><span class="sxs-lookup"><span data-stu-id="224be-357">If you are considering the console provider for use in production, be aware that it has a significant impact on performance.</span></span>

<span data-ttu-id="224be-358">在 Visual Studio 中创建新项目时，`AddConsole` 方法如下所示：</span><span class="sxs-lookup"><span data-stu-id="224be-358">When you create a new project in Visual Studio, the `AddConsole` method looks like this:</span></span>

```csharp
loggerFactory.AddConsole(Configuration.GetSection("Logging"));
```

<span data-ttu-id="224be-359">此代码引用 appSettings.json 文件的 `Logging` 部分：</span><span class="sxs-lookup"><span data-stu-id="224be-359">This code refers to the `Logging` section of the *appSettings.json* file:</span></span>

[!code-json[](index/sample//appsettings.json)]

<span data-ttu-id="224be-360">所显示的设置将框架日志限制为警告，并允许在调试级别记录应用日志，如[日志筛选](#log-filtering)部分所述。</span><span class="sxs-lookup"><span data-stu-id="224be-360">The settings shown limit framework logs to warnings while allowing the app to log at debug level, as explained in the [Log filtering](#log-filtering) section.</span></span> <span data-ttu-id="224be-361">有关详细信息，请参阅[配置](xref:fundamentals/configuration/index)。</span><span class="sxs-lookup"><span data-stu-id="224be-361">For more information, see [Configuration](xref:fundamentals/configuration/index).</span></span>

---

<a id="debug"></a>
### <a name="the-debug-provider"></a><span data-ttu-id="224be-362">调试提供程序</span><span class="sxs-lookup"><span data-stu-id="224be-362">The Debug provider</span></span>

<span data-ttu-id="224be-363">[Microsoft.Extensions.Logging.Debug](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Debug) 提供程序包使用 [System.Diagnostics.Debug](https://docs.microsoft.com/dotnet/core/api/system.diagnostics.debug#System_Diagnostics_Debug) 类（`Debug.WriteLine` 方法调用）来写入日志输出。</span><span class="sxs-lookup"><span data-stu-id="224be-363">The [Microsoft.Extensions.Logging.Debug](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Debug) provider package writes log output by using the [System.Diagnostics.Debug](https://docs.microsoft.com/dotnet/core/api/system.diagnostics.debug#System_Diagnostics_Debug) class (`Debug.WriteLine` method calls).</span></span>

<span data-ttu-id="224be-364">在 Linux 中，此提供程序将日志写入 /var/log/message。</span><span class="sxs-lookup"><span data-stu-id="224be-364">On Linux, this provider writes logs to */var/log/message*.</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="224be-365">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="224be-365">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

```csharp
logging.AddDebug()
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="224be-366">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="224be-366">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

```csharp
loggerFactory.AddDebug()
```

<span data-ttu-id="224be-367">[AddDebug 重载](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.debugloggerfactoryextensions)允许传入最低日志级别或筛选器函数。</span><span class="sxs-lookup"><span data-stu-id="224be-367">[AddDebug overloads](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.debugloggerfactoryextensions) let you pass in a minimum log level or a filter function.</span></span>

---

<a id="eventsource"></a>
### <a name="the-eventsource-provider"></a><span data-ttu-id="224be-368">EventSource 提供程序</span><span class="sxs-lookup"><span data-stu-id="224be-368">The EventSource provider</span></span>

<span data-ttu-id="224be-369">对于面向 ASP.NET Core 1.1.0 或更高版本的应用，[Microsoft.Extensions.Logging.EventSource](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventSource) 提供程序包可实现事件跟踪。</span><span class="sxs-lookup"><span data-stu-id="224be-369">For apps that target ASP.NET Core 1.1.0 or higher, the [Microsoft.Extensions.Logging.EventSource](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventSource) provider package can implement event tracing.</span></span> <span data-ttu-id="224be-370">在 Windows 中，它使用 [ETW](https://msdn.microsoft.com/library/windows/desktop/bb968803)。</span><span class="sxs-lookup"><span data-stu-id="224be-370">On Windows, it uses [ETW](https://msdn.microsoft.com/library/windows/desktop/bb968803).</span></span> <span data-ttu-id="224be-371">提供程序可跨平台使用，但尚无支持 Linux 或 macOS 的事件集合和显示工具。</span><span class="sxs-lookup"><span data-stu-id="224be-371">The provider is cross-platform, but there are no event collection and display tools yet for Linux or macOS.</span></span> 

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="224be-372">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="224be-372">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

```csharp
logging.AddEventSourceLogger()
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="224be-373">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="224be-373">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

```csharp
loggerFactory.AddEventSourceLogger()
```

---

<span data-ttu-id="224be-374">可使用 [PerfView 实用工具](https://www.microsoft.com/download/details.aspx?id=28567)收集和查看日志。</span><span class="sxs-lookup"><span data-stu-id="224be-374">A good way to collect and view logs is to use the [PerfView utility](https://www.microsoft.com/download/details.aspx?id=28567).</span></span> <span data-ttu-id="224be-375">虽然其他工具也可以查看 ETW 日志，但在处理由 ASP.NET 发出的 ETW 事件时，使用 PerfView 能获得最佳体验。</span><span class="sxs-lookup"><span data-stu-id="224be-375">There are other tools for viewing ETW logs, but PerfView provides the best experience for working with the ETW events emitted by ASP.NET.</span></span> 

<span data-ttu-id="224be-376">要将 PerfView 配置为收集此提供程序记录的事件，请向 Additional Providers 列表添加字符串 `*Microsoft-Extensions-Logging`。</span><span class="sxs-lookup"><span data-stu-id="224be-376">To configure PerfView for collecting events logged by this provider, add the string `*Microsoft-Extensions-Logging` to the **Additional Providers** list.</span></span> <span data-ttu-id="224be-377">（请勿遗漏字符串起始处的星号。）</span><span class="sxs-lookup"><span data-stu-id="224be-377">(Don't miss the asterisk at the start of the string.)</span></span>

![其他 Perfview 提供程序](index/_static/perfview-additional-providers.png)

<span data-ttu-id="224be-379">要捕获 Nano Server 的事件，需要进行一些额外设置：</span><span class="sxs-lookup"><span data-stu-id="224be-379">Capturing events on Nano Server requires some additional setup:</span></span>

* <span data-ttu-id="224be-380">将 PowerShell 远程连接到 Nano Server：</span><span class="sxs-lookup"><span data-stu-id="224be-380">Connect PowerShell remoting to the Nano Server:</span></span>

  ```powershell
  Enter-PSSession [name]
  ```

* <span data-ttu-id="224be-381">创建 ETW 会话：</span><span class="sxs-lookup"><span data-stu-id="224be-381">Create an ETW session:</span></span>

  ```powershell
  New-EtwTraceSession -Name "MyAppTrace" -LocalFilePath C:\trace.etl
  ```

* <span data-ttu-id="224be-382">按需求为 [CLR](https://docs.microsoft.com/dotnet/framework/performance/clr-etw-providers) 和 ASP.NET Core 等添加 ETW 提供程序。</span><span class="sxs-lookup"><span data-stu-id="224be-382">Add ETW providers for [CLR](https://docs.microsoft.com/dotnet/framework/performance/clr-etw-providers), ASP.NET Core, and others as needed.</span></span> <span data-ttu-id="224be-383">ASP.NET Core 提供程序 GUID 为 `3ac73b97-af73-50e9-0822-5da4367920d0`。</span><span class="sxs-lookup"><span data-stu-id="224be-383">The ASP.NET Core provider GUID is `3ac73b97-af73-50e9-0822-5da4367920d0`.</span></span> 

  ```powershell
  Add-EtwTraceProvider -Guid "{e13c0d23-ccbc-4e12-931b-d9cc2eee27e4}" -SessionName MyAppTrace
  Add-EtwTraceProvider -Guid "{3ac73b97-af73-50e9-0822-5da4367920d0}" -SessionName MyAppTrace
  ```

* <span data-ttu-id="224be-384">运行站点，并执行要跟踪其信息的操作。</span><span class="sxs-lookup"><span data-stu-id="224be-384">Run the site and do whatever actions you want tracing information for.</span></span>

* <span data-ttu-id="224be-385">在结束后停止跟踪会话：</span><span class="sxs-lookup"><span data-stu-id="224be-385">Stop the tracing session when you're finished:</span></span>

  ```powershell
  Stop-EtwTraceSession -Name "MyAppTrace"
  ```

<span data-ttu-id="224be-386">与在其他 Windows 版本中类似，可使用 PerfView 分析所生成的 C:\trace.etl 文件。</span><span class="sxs-lookup"><span data-stu-id="224be-386">The resulting *C:\trace.etl* file can be analyzed with PerfView as on other editions of Windows.</span></span>

<a id="eventlog"></a>
### <a name="the-windows-eventlog-provider"></a><span data-ttu-id="224be-387">Windows EventLog 提供程序</span><span class="sxs-lookup"><span data-stu-id="224be-387">The Windows EventLog provider</span></span>

<span data-ttu-id="224be-388">[Microsoft.Extensions.Logging.EventLog](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventLog) 提供程序包向 Windows 事件日志发送日志输出。</span><span class="sxs-lookup"><span data-stu-id="224be-388">The [Microsoft.Extensions.Logging.EventLog](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventLog) provider package sends log output to the Windows Event Log.</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="224be-389">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="224be-389">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

```csharp
logging.AddEventLog()
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="224be-390">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="224be-390">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

```csharp
loggerFactory.AddEventLog()
```

<span data-ttu-id="224be-391">[AddEventLog 重载](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.eventloggerfactoryextensions)允许传入 `EventLogSettings` 或最低日志级别。</span><span class="sxs-lookup"><span data-stu-id="224be-391">[AddEventLog overloads](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.eventloggerfactoryextensions) let you pass in `EventLogSettings` or a minimum log level.</span></span>

---

<a id="tracesource"></a>
### <a name="the-tracesource-provider"></a><span data-ttu-id="224be-392">TraceSource 提供程序</span><span class="sxs-lookup"><span data-stu-id="224be-392">The TraceSource provider</span></span>

<span data-ttu-id="224be-393">[Microsoft.Extensions.Logging.TraceSource](https://www.nuget.org/packages/Microsoft.Extensions.Logging.TraceSource) 提供程序包使用 [System.Diagnostics.TraceSource](https://docs.microsoft.com/dotnet/api/system.diagnostics.tracesource) 库和提供程序。</span><span class="sxs-lookup"><span data-stu-id="224be-393">The [Microsoft.Extensions.Logging.TraceSource](https://www.nuget.org/packages/Microsoft.Extensions.Logging.TraceSource) provider package uses the [System.Diagnostics.TraceSource](https://docs.microsoft.com/dotnet/api/system.diagnostics.tracesource) libraries and providers.</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="224be-394">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="224be-394">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

```csharp
logging.AddTraceSource(sourceSwitchName);
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="224be-395">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="224be-395">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

```csharp
loggerFactory.AddTraceSource(sourceSwitchName);
```

---

<span data-ttu-id="224be-396">[AddTraceSource 重载](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.tracesourcefactoryextensions) 允许传入资源开关和跟踪侦听器。</span><span class="sxs-lookup"><span data-stu-id="224be-396">[AddTraceSource overloads](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.tracesourcefactoryextensions) let you pass in a source switch and a trace listener.</span></span>

<span data-ttu-id="224be-397">要使用此提供程序，应用程序必须在 .NET Framework（而非 .NET Core）上运行。</span><span class="sxs-lookup"><span data-stu-id="224be-397">To use this provider, an application has to run on the .NET Framework (rather than .NET Core).</span></span> <span data-ttu-id="224be-398">使用提供程序，可以将消息路由到多个[侦听器](https://docs.microsoft.com/dotnet/framework/debug-trace-profile/trace-listeners)，例如示例应用程序中使用的 [TextWriterTraceListener](https://docs.microsoft.com/dotnet/api/system.diagnostics.textwritertracelistenerr)。</span><span class="sxs-lookup"><span data-stu-id="224be-398">The provider lets you route messages to a variety of [listeners](https://docs.microsoft.com/dotnet/framework/debug-trace-profile/trace-listeners), such as the [TextWriterTraceListener](https://docs.microsoft.com/dotnet/api/system.diagnostics.textwritertracelistenerr) used in the sample application.</span></span>

<span data-ttu-id="224be-399">以下示例配置了 `TraceSource` 提供程序，用于向控制台窗口记录 `Warning` 和更高级别的消息。</span><span class="sxs-lookup"><span data-stu-id="224be-399">The following example configures a `TraceSource` provider that logs `Warning` and higher messages to the console window.</span></span>

[!code-csharp[](index/sample/Startup.cs?name=snippet_TraceSource&highlight=9-12)]

<a id="appservice"></a>
### <a name="the-azure-app-service-provider"></a><span data-ttu-id="224be-400">Azure App Service 提供程序</span><span class="sxs-lookup"><span data-stu-id="224be-400">The Azure App Service provider</span></span>

<span data-ttu-id="224be-401">[Microsoft.Extensions.Logging.AzureAppServices](https://www.nuget.org/packages/Microsoft.Extensions.Logging.AzureAppServices) 提供程序包将日志写入 Azure App Service 应用的文件系统，以及 Azure 存储帐户中的 [blob 存储](https://azure.microsoft.com/documentation/articles/storage-dotnet-how-to-use-blobs/#what-is-blob-storage)。</span><span class="sxs-lookup"><span data-stu-id="224be-401">The [Microsoft.Extensions.Logging.AzureAppServices](https://www.nuget.org/packages/Microsoft.Extensions.Logging.AzureAppServices) provider package writes logs to text files in an Azure App Service app's file system and to [blob storage](https://azure.microsoft.com/documentation/articles/storage-dotnet-how-to-use-blobs/#what-is-blob-storage) in an Azure Storage account.</span></span> <span data-ttu-id="224be-402">该提供程序仅适用于面向 ASP.NET Core 1.1.0 或更高版本的应用。</span><span class="sxs-lookup"><span data-stu-id="224be-402">The provider is available only for apps that target ASP.NET Core 1.1.0 or higher.</span></span> 

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="224be-403">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="224be-403">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="224be-404">如果面向 .NET Core，无需安装提供程序包或显式调用 `AddAzureWebAppDiagnostics`。</span><span class="sxs-lookup"><span data-stu-id="224be-404">If targeting .NET Core, you don't have to install the provider package or explicitly call `AddAzureWebAppDiagnostics`.</span></span> <span data-ttu-id="224be-405">将应用部署到 Azure App Service 时，提供程序对应用自动可用。</span><span class="sxs-lookup"><span data-stu-id="224be-405">The provider is automatically available to your app when you deploy the app to Azure App Service.</span></span>

<span data-ttu-id="224be-406">如果面向 .NET Framework，需要向项目添加提供程序包并调用 `AddAzureWebAppDiagnostics`：</span><span class="sxs-lookup"><span data-stu-id="224be-406">If targeting .NET Framework, add the provider package to your project and invoke `AddAzureWebAppDiagnostics`:</span></span>

```csharp
logging.AddAzureWebAppDiagnostics();
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="224be-407">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="224be-407">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

```csharp
loggerFactory.AddAzureWebAppDiagnostics();
```

<span data-ttu-id="224be-408">`AddAzureWebAppDiagnostics` 重载允许传入 [AzureAppServicesDiagnosticsSettings](https://github.com/aspnet/Logging/blob/c7d0b1b88668ff4ef8a86ea7d2ebb5ca7f88d3e0/src/Microsoft.Extensions.Logging.AzureAppServices/AzureAppServicesDiagnosticsSettings.cs)，可用它替代默认设置，例如日志记录输出模板、blob 名称和文件大小限制等。</span><span class="sxs-lookup"><span data-stu-id="224be-408">An `AddAzureWebAppDiagnostics` overload lets you pass in [AzureAppServicesDiagnosticsSettings](https://github.com/aspnet/Logging/blob/c7d0b1b88668ff4ef8a86ea7d2ebb5ca7f88d3e0/src/Microsoft.Extensions.Logging.AzureAppServices/AzureAppServicesDiagnosticsSettings.cs) with which you can override default settings such as the logging output template, blob name, and file size limit.</span></span> <span data-ttu-id="224be-409">（输出模板是应用于所有日志的消息模板，其优先级高于调用 `ILogger` 方法时提供的模板。）</span><span class="sxs-lookup"><span data-stu-id="224be-409">(*Output template* is a message template that's applied to all logs on top of the one that you provide when you call an `ILogger` method.)</span></span>

---

<span data-ttu-id="224be-410">在部署 App Service 应用时，应用程序将遵循 Azure 门户中 App Service 页下[诊断日志](https://azure.microsoft.com/documentation/articles/web-sites-enable-diagnostic-log/#enablediag)部分的设置。</span><span class="sxs-lookup"><span data-stu-id="224be-410">When you deploy to an App Service app, your application honors the settings in the [Diagnostic Logs](https://azure.microsoft.com/documentation/articles/web-sites-enable-diagnostic-log/#enablediag) section of the **App Service** page of the Azure portal.</span></span> <span data-ttu-id="224be-411">这些设置将在更改后立即生效，无需重启应用或重新向其部署代码。</span><span class="sxs-lookup"><span data-stu-id="224be-411">When you change those settings, the changes take effect immediately without requiring that you restart the app or redeploy code to it.</span></span> 

![Azure 日志记录设置](index/_static/azure-logging-settings.png)

<span data-ttu-id="224be-413">日志文件的默认位置是 D:\\home\\LogFiles\\Application 文件夹，默认文件名为 diagnostics-yyyymmdd.txt。</span><span class="sxs-lookup"><span data-stu-id="224be-413">The default location for log files is in the *D:\\home\\LogFiles\\Application* folder, and the default file name is *diagnostics-yyyymmdd.txt*.</span></span> <span data-ttu-id="224be-414">默认文件大小上限为 10 MB，默认最大保留文件数为 2。</span><span class="sxs-lookup"><span data-stu-id="224be-414">The default file size limit is 10 MB, and the default maximum number of files retained is 2.</span></span> <span data-ttu-id="224be-415">默认 blob 名为 {app-name}{timestamp}/yyyy/mm/dd/hh/{guid}-applicationLog.txt。</span><span class="sxs-lookup"><span data-stu-id="224be-415">The default blob name is *{app-name}{timestamp}/yyyy/mm/dd/hh/{guid}-applicationLog.txt*.</span></span> <span data-ttu-id="224be-416">有关默认行为的详细信息，请参阅 [AzureAppServicesDiagnosticsSettings](https://github.com/aspnet/Logging/blob/c7d0b1b88668ff4ef8a86ea7d2ebb5ca7f88d3e0/src/Microsoft.Extensions.Logging.AzureAppServices/AzureAppServicesDiagnosticsSettings.cs)。</span><span class="sxs-lookup"><span data-stu-id="224be-416">For more information about default behavior, see [AzureAppServicesDiagnosticsSettings](https://github.com/aspnet/Logging/blob/c7d0b1b88668ff4ef8a86ea7d2ebb5ca7f88d3e0/src/Microsoft.Extensions.Logging.AzureAppServices/AzureAppServicesDiagnosticsSettings.cs).</span></span>

<span data-ttu-id="224be-417">该提供程序仅当项目在 Azure 环境中运行时有效。</span><span class="sxs-lookup"><span data-stu-id="224be-417">The provider only works when your project runs in the Azure environment.</span></span> <span data-ttu-id="224be-418">它在本地运行时无效，不会写入本地文件或 blob 的本地开发存储。</span><span class="sxs-lookup"><span data-stu-id="224be-418">It has no effect when you run locally &mdash; it does not write to local files or local development storage for blobs.</span></span>

## <a name="third-party-logging-providers"></a><span data-ttu-id="224be-419">第三方日志记录提供程序</span><span class="sxs-lookup"><span data-stu-id="224be-419">Third-party logging providers</span></span>

<span data-ttu-id="224be-420">以下第三方日志记录框架适用于 ASP.NET Core：</span><span class="sxs-lookup"><span data-stu-id="224be-420">Here are some third-party logging frameworks that work with ASP.NET Core:</span></span>

* <span data-ttu-id="224be-421">[elmah.io](https://github.com/elmahio/Elmah.Io.Extensions.Logging) - Elmah.Io 服务的提供程序</span><span class="sxs-lookup"><span data-stu-id="224be-421">[elmah.io](https://github.com/elmahio/Elmah.Io.Extensions.Logging) - provider for the Elmah.Io service</span></span>

* <span data-ttu-id="224be-422">[JSNLog](http://jsnlog.com) - 在服务器端日志中记录 JavaScript 异常和其他客户端事件。</span><span class="sxs-lookup"><span data-stu-id="224be-422">[JSNLog](http://jsnlog.com) - logs JavaScript exceptions and other client-side events in your server-side log.</span></span>

* <span data-ttu-id="224be-423">[Loggr](https://github.com/imobile3/Loggr.Extensions.Logging) - Loggr 服务的提供程序</span><span class="sxs-lookup"><span data-stu-id="224be-423">[Loggr](https://github.com/imobile3/Loggr.Extensions.Logging) - provider for the Loggr service</span></span>

* <span data-ttu-id="224be-424">[NLog](https://github.com/NLog/NLog.Extensions.Logging) - NLog 库的提供程序</span><span class="sxs-lookup"><span data-stu-id="224be-424">[NLog](https://github.com/NLog/NLog.Extensions.Logging) - provider for the NLog library</span></span>

* <span data-ttu-id="224be-425">[Serilog](https://github.com/serilog/serilog-extensions-logging) - Serilog 库的提供程序</span><span class="sxs-lookup"><span data-stu-id="224be-425">[Serilog](https://github.com/serilog/serilog-extensions-logging) - provider for the Serilog library</span></span>

<span data-ttu-id="224be-426">某些第三方框架可以执行[语义日志记录（又称结构化日志记录）](https://softwareengineering.stackexchange.com/questions/312197/benefits-of-structured-logging-vs-basic-logging)。</span><span class="sxs-lookup"><span data-stu-id="224be-426">Some third-party frameworks can do [semantic logging, also known as structured logging](https://softwareengineering.stackexchange.com/questions/312197/benefits-of-structured-logging-vs-basic-logging).</span></span>

<span data-ttu-id="224be-427">使用第三方框架与使用内置提供程序类似：向项目添加 NuGet 包并在 `ILoggerFactory` 上调用扩展方法即可。</span><span class="sxs-lookup"><span data-stu-id="224be-427">Using a third-party framework is similar to using one of the built-in providers: add a NuGet package to your project and call an extension method on `ILoggerFactory`.</span></span> <span data-ttu-id="224be-428">有关详细信息，请参阅各框架的相关文档。</span><span class="sxs-lookup"><span data-stu-id="224be-428">For more information, see each framework's documentation.</span></span>

<span data-ttu-id="224be-429">还可创建自定义提供程序，以支持其他日志记录框架或满足自身日志记录需求。</span><span class="sxs-lookup"><span data-stu-id="224be-429">You can create your own custom providers as well, to support other logging frameworks or your own logging requirements.</span></span>

## <a name="azure-log-streaming"></a><span data-ttu-id="224be-430">Azure 日志流式处理</span><span class="sxs-lookup"><span data-stu-id="224be-430">Azure log streaming</span></span>

<span data-ttu-id="224be-431">通过 Azure 日志流式处理，可从以下位置实时查看日志活动：</span><span class="sxs-lookup"><span data-stu-id="224be-431">Azure log streaming enables you to view log activity in real time from:</span></span> 

* <span data-ttu-id="224be-432">应用程序服务器</span><span class="sxs-lookup"><span data-stu-id="224be-432">The application server</span></span> 
* <span data-ttu-id="224be-433">Web 服务器</span><span class="sxs-lookup"><span data-stu-id="224be-433">The web server</span></span>
* <span data-ttu-id="224be-434">请求跟踪失败</span><span class="sxs-lookup"><span data-stu-id="224be-434">Failed request tracing</span></span> 

<span data-ttu-id="224be-435">要配置 Azure 日志流式处理，请执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="224be-435">To configure Azure log streaming:</span></span> 

* <span data-ttu-id="224be-436">从应用程序的门户页导航到“诊断日志”页</span><span class="sxs-lookup"><span data-stu-id="224be-436">Navigate to the **Diagnostics Logs** page from your application's portal page</span></span>
* <span data-ttu-id="224be-437">将“应用程序日志记录 (Filesystem)”设置为启用。</span><span class="sxs-lookup"><span data-stu-id="224be-437">Set **Application Logging (Filesystem)** to on.</span></span> 

![Azure 门户诊断日志页](index/_static/azure-diagnostic-logs.png)

<span data-ttu-id="224be-439">导航到“日志流式处理”页，查看应用程序消息。</span><span class="sxs-lookup"><span data-stu-id="224be-439">Navigate to the **Log Streaming** page to view application messages.</span></span> <span data-ttu-id="224be-440">它们由应用程序通过 `ILogger` 接口记录。</span><span class="sxs-lookup"><span data-stu-id="224be-440">They are logged by application through the `ILogger` interface.</span></span> 

![Azure 门户应用程序日志流式处理](index/_static/azure-log-streaming.png)


## <a name="see-also"></a><span data-ttu-id="224be-442">请参阅</span><span class="sxs-lookup"><span data-stu-id="224be-442">See also</span></span>

[<span data-ttu-id="224be-443">使用 LoggerMessage 的高性能日志记录</span><span class="sxs-lookup"><span data-stu-id="224be-443">High-performance logging with LoggerMessage</span></span>](xref:fundamentals/logging/loggermessage)