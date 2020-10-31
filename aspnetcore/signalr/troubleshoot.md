---
title: 'ASP.NET Core :::no-loc(SignalR)::: 연결 문제 해결'
author: bradygaster
description: :::no-loc(SignalR):::연결 문제 해결을 ASP.NET Core 합니다.
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 04/08/2020
no-loc:
- ':::no-loc(appsettings.json):::'
- ':::no-loc(ASP.NET Core Identity):::'
- ':::no-loc(cookie):::'
- ':::no-loc(Cookie):::'
- ':::no-loc(Blazor):::'
- ':::no-loc(Blazor Server):::'
- ':::no-loc(Blazor WebAssembly):::'
- ':::no-loc(Identity):::'
- ":::no-loc(Let's Encrypt):::"
- ':::no-loc(Razor):::'
- ':::no-loc(SignalR):::'
uid: signalr/troubleshoot
ms.openlocfilehash: f1d9761267d7c6af76c0be6abb238742f40fb016
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93059613"
---
# <a name="troubleshoot-connection-errors"></a><span data-ttu-id="4ef0b-103">연결 오류 문제 해결</span><span class="sxs-lookup"><span data-stu-id="4ef0b-103">Troubleshoot connection errors</span></span>

<span data-ttu-id="4ef0b-104">이 섹션에서는 ASP.NET Core 허브에 대 한 연결을 설정 하려고 할 때 발생할 수 있는 오류에 대 한 도움말을 제공 합니다 :::no-loc(SignalR)::: .</span><span class="sxs-lookup"><span data-stu-id="4ef0b-104">This section provides help with errors that can occur when trying to establish a connection to a ASP.NET Core :::no-loc(SignalR)::: hub.</span></span>

### <a name="response-code-404"></a><span data-ttu-id="4ef0b-105">응답 코드 404</span><span class="sxs-lookup"><span data-stu-id="4ef0b-105">Response code 404</span></span>

<span data-ttu-id="4ef0b-106">Websocket을 사용 하는 경우 `skipNegotiation = true`</span><span class="sxs-lookup"><span data-stu-id="4ef0b-106">When using WebSockets and `skipNegotiation = true`</span></span>
```log
WebSocket connection to 'wss://xxx/HubName' failed: Error during WebSocket handshake: Unexpected response code: 404
```

* <span data-ttu-id="4ef0b-107">고정 세션이 없는 여러 서버를 사용 하는 경우 한 서버에서 연결을 시작한 후 다른 서버로 전환할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-107">When using multiple servers without sticky sessions, the connection can start on one server and then switch to another server.</span></span> <span data-ttu-id="4ef0b-108">다른 서버는 이전 연결을 인식 하지 못합니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-108">The other server is not aware of the previous connection.</span></span>
* <span data-ttu-id="4ef0b-109">클라이언트가 올바른 끝점에 연결 하 고 있는지 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-109">Verify the client is connecting to the correct endpoint.</span></span> <span data-ttu-id="4ef0b-110">예를 들어 서버가에서 호스트 되 `http://127.0.0.1:5000/hub/myHub` 고 클라이언트가에 연결 하려고 `http://127.0.0.1:5000/myHub` 합니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-110">For example, the server is hosted at `http://127.0.0.1:5000/hub/myHub` and client is trying to connect to `http://127.0.0.1:5000/myHub`.</span></span>
* <span data-ttu-id="4ef0b-111">연결에서 ID를 사용 하 고 협상 후 서버에 요청을 보내는 데 시간이 너무 오래 걸리는 경우 서버는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-111">If the connection uses the ID and takes too long to send a request to the server after the negotiate, the server:</span></span>

  * <span data-ttu-id="4ef0b-112">ID를 삭제 합니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-112">Deletes the ID.</span></span>
  * <span data-ttu-id="4ef0b-113">404을 반환 합니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-113">Returns a 404.</span></span>

### <a name="response-code-400-or-503"></a><span data-ttu-id="4ef0b-114">응답 코드 400 또는 503</span><span class="sxs-lookup"><span data-stu-id="4ef0b-114">Response code 400 or 503</span></span>

<span data-ttu-id="4ef0b-115">다음 오류의 경우:</span><span class="sxs-lookup"><span data-stu-id="4ef0b-115">For the following error:</span></span>

```log
WebSocket connection to 'wss://xxx/HubName' failed: Error during WebSocket handshake: Unexpected response code: 400

Error: Failed to start the connection: Error: There was an error with the transport.
```

<span data-ttu-id="4ef0b-116">이 오류는 대개 Websocket 전송만 사용 하는 클라이언트에 의해 발생 하지만 서버에서 Websocket 프로토콜이 사용 하도록 설정 되어 있지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-116">This error is usually caused by a client using only the WebSockets transport but the WebSockets protocol is not enabled on the server.</span></span>

### <a name="response-code-307"></a><span data-ttu-id="4ef0b-117">응답 코드 307</span><span class="sxs-lookup"><span data-stu-id="4ef0b-117">Response code 307</span></span>

<span data-ttu-id="4ef0b-118">Websocket을 사용 하는 경우 `skipNegotiation = true`</span><span class="sxs-lookup"><span data-stu-id="4ef0b-118">When using WebSockets and `skipNegotiation = true`</span></span>
```log
WebSocket connection to 'ws://xxx/HubName' failed: Error during WebSocket handshake: Unexpected response code: 307
```

<span data-ttu-id="4ef0b-119">이 오류는 negotiate 요청 중에도 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-119">This error can also happen during the negotiate request.</span></span>

<span data-ttu-id="4ef0b-120">일반적인 원인:</span><span class="sxs-lookup"><span data-stu-id="4ef0b-120">Common cause:</span></span>
* <span data-ttu-id="4ef0b-121">앱은에서를 호출 하 여 HTTPS를 적용 `UseHttpsRedirection` `Startup` 하거나 URL 재작성 규칙을 통해 https를 적용 하도록 구성 됩니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-121">App is configured to enforce HTTPS by calling `UseHttpsRedirection` in `Startup`, or enforces HTTPS via URL rewrite rule.</span></span>

<span data-ttu-id="4ef0b-122">가능한 해결 방법:</span><span class="sxs-lookup"><span data-stu-id="4ef0b-122">Possible solution:</span></span>
* <span data-ttu-id="4ef0b-123">클라이언트 쪽의 URL을 "http"에서 "https"로 변경 합니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-123">Change the URL on the client side from "http" to "https".</span></span> `.withUrl("https://xxx/HubName")`

### <a name="response-code-405"></a><span data-ttu-id="4ef0b-124">응답 코드 405</span><span class="sxs-lookup"><span data-stu-id="4ef0b-124">Response code 405</span></span>

<span data-ttu-id="4ef0b-125">Http 상태 코드 405-메서드를 사용할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-125">Http status code 405 - Method Not Allowed</span></span>

* <span data-ttu-id="4ef0b-126">앱에서 [CORS](xref:signalr/security#cross-origin-resource-sharing) 를 사용 하도록 설정 하지 않았습니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-126">The app doesn't have [CORS](xref:signalr/security#cross-origin-resource-sharing) enabled</span></span>

### <a name="response-code-0"></a><span data-ttu-id="4ef0b-127">응답 코드 0</span><span class="sxs-lookup"><span data-stu-id="4ef0b-127">Response code 0</span></span>

<span data-ttu-id="4ef0b-128">Http 상태 코드 0-일반적으로 [CORS](xref:signalr/security#cross-origin-resource-sharing) 문제 이며 상태 코드는 제공 되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-128">Http status code 0 - Usually a [CORS](xref:signalr/security#cross-origin-resource-sharing) issue, no status code is given</span></span>

```log
Cross-Origin Request Blocked: The Same Origin Policy disallows reading the remote resource at http://localhost:5000/default/negotiate?negotiateVersion=1. (Reason: CORS header 'Access-Control-Allow-Origin' missing).
```

* <span data-ttu-id="4ef0b-129">예상 원본을에 추가 합니다. `.WithOrigins(...)`</span><span class="sxs-lookup"><span data-stu-id="4ef0b-129">Add the expected origins to `.WithOrigins(...)`</span></span>

```log
Cross-Origin Request Blocked: The Same Origin Policy disallows reading the remote resource at http://localhost:5000/default/negotiate?negotiateVersion=1. (Reason: expected 'true' in CORS header 'Access-Control-Allow-Credentials').
```

* <span data-ttu-id="4ef0b-130">`.AllowCredentials()`CORS 정책에를 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-130">Add `.AllowCredentials()` to your CORS policy.</span></span> <span data-ttu-id="4ef0b-131">`.AllowAnyOrigin()` `.WithOrigins("*")` 이 옵션과 함께 또는를 사용할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-131">Cannot use `.AllowAnyOrigin()` or `.WithOrigins("*")` with this option</span></span>

### <a name="response-code-413"></a><span data-ttu-id="4ef0b-132">응답 코드 413</span><span class="sxs-lookup"><span data-stu-id="4ef0b-132">Response code 413</span></span>

<span data-ttu-id="4ef0b-133">Http 상태 코드 413-페이로드가 너무 큼</span><span class="sxs-lookup"><span data-stu-id="4ef0b-133">Http status code 413 - Payload Too Large</span></span>

<span data-ttu-id="4ef0b-134">이는 종종 4k를 초과 하는 액세스 토큰이 있는 경우에 발생 합니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-134">This is often caused by having an access token that is over 4k.</span></span>

* <span data-ttu-id="4ef0b-135">Azure 서비스를 사용 하는 경우 다음을 사용 하 여 :::no-loc(SignalR)::: 서비스를 통해 전송 되는 클레임을 사용자 지정 하 여 토큰 크기를 줄입니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-135">If using the Azure :::no-loc(SignalR)::: Service, reduce the token size by customizing the claims being sent through the Service with:</span></span>
```csharp
.AddAzure:::no-loc(SignalR):::(options =>
{
    options.ClaimsProvider = context => context.User.Claims;
});
```

### <a name="transient-network-failures"></a><span data-ttu-id="4ef0b-136">일시적인 네트워크 오류</span><span class="sxs-lookup"><span data-stu-id="4ef0b-136">Transient network failures</span></span>

<span data-ttu-id="4ef0b-137">일시적인 네트워크 오류로 인해 연결이 닫힐 수 있습니다 :::no-loc(SignalR)::: .</span><span class="sxs-lookup"><span data-stu-id="4ef0b-137">Transient network failures may close the :::no-loc(SignalR)::: connection.</span></span> <span data-ttu-id="4ef0b-138">서버는 닫힌 연결을 정상적인 클라이언트 연결 끊기로 해석할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-138">The server may interpret the closed connection as a graceful client disconnect.</span></span> <span data-ttu-id="4ef0b-139">이러한 경우 클라이언트에서 연결을 끊은 이유에 대 한 자세한 정보를 보려면 [클라이언트 및 서버에서 로그를 수집](xref:signalr/diagnostics)합니다.</span><span class="sxs-lookup"><span data-stu-id="4ef0b-139">To get more info on why a client disconnected in those cases [gather logs from the client and server](xref:signalr/diagnostics).</span></span>