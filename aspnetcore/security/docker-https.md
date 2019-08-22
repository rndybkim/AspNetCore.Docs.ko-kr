---
title: HTTPS를 통해 Docker를 사용 하 여 ASP.NET Core 이미지 호스팅
author: rick-anderson
description: HTTPS를 통해 Docker를 사용 하 여 ASP.NET Core 이미지를 호스트 하는 방법을 알아봅니다.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/05/2019
uid: security/docker-https
ms.openlocfilehash: f17a3abe1b00b39b7b6787be5b20ce65771190b8
ms.sourcegitcommit: 257cc3fe8c1d61341aa3b07e5bc0fa3d1c1c1d1c
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/19/2019
ms.locfileid: "69619698"
---
# <a name="hosting-aspnet-core-images-with-docker-over-https"></a><span data-ttu-id="7bed6-103">HTTPS를 통해 Docker를 사용 하 여 ASP.NET Core 이미지 호스팅</span><span class="sxs-lookup"><span data-stu-id="7bed6-103">Hosting ASP.NET Core images with Docker over HTTPS</span></span>

<span data-ttu-id="7bed6-104">작성자: [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="7bed6-104">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="7bed6-105">ASP.NET Core는 [기본적으로 HTTPS를](/aspnet/core/security/enforcing-ssl)사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-105">ASP.NET Core uses [HTTPS by default](/aspnet/core/security/enforcing-ssl).</span></span> <span data-ttu-id="7bed6-106">[HTTPS](https://en.wikipedia.org/wiki/HTTPS) 는 [인증서](https://en.wikipedia.org/wiki/Public_key_certificate) 를 신뢰, id 및 암호화에 의존 합니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-106">[HTTPS](https://en.wikipedia.org/wiki/HTTPS) relies on [certificates](https://en.wikipedia.org/wiki/Public_key_certificate) for trust, identity, and encryption.</span></span>

<span data-ttu-id="7bed6-107">이 문서에서는 HTTPS를 사용 하 여 미리 작성 된 컨테이너 이미지를 실행 하는 방법을 설명 합니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-107">This document explains how to run pre-built container images with HTTPS.</span></span>

<span data-ttu-id="7bed6-108">개발 시나리오는 [HTTPS를 통해 Docker를 사용 하 여 ASP.NET Core 응용 프로그램 개발](https://github.com/dotnet/dotnet-docker/blob/master/samples/aspnetapp/aspnetcore-docker-https-development.md) 을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="7bed6-108">See [Developing ASP.NET Core Applications with Docker over HTTPS](https://github.com/dotnet/dotnet-docker/blob/master/samples/aspnetapp/aspnetcore-docker-https-development.md) for development scenarios.</span></span>

<span data-ttu-id="7bed6-109">이 샘플에는 docker [클라이언트](https://www.docker.com/products/docker)의 [docker 17.06](https://docs.docker.com/release-notes/docker-ce) 이상이 필요 합니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-109">This sample requires [Docker 17.06](https://docs.docker.com/release-notes/docker-ce) or later of the [Docker client](https://www.docker.com/products/docker).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="7bed6-110">필수 구성 요소</span><span class="sxs-lookup"><span data-stu-id="7bed6-110">Prerequisites</span></span>

<span data-ttu-id="7bed6-111">이 문서의 지침 중 일부에는 [.Net Core 2.2 SDK](https://www.microsoft.com/net/download) 이상이 필요 합니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-111">The [.NET Core 2.2 SDK](https://www.microsoft.com/net/download) or later is required for some of the instructions in this document.</span></span>

## <a name="certificates"></a><span data-ttu-id="7bed6-112">인증서</span><span class="sxs-lookup"><span data-stu-id="7bed6-112">Certificates</span></span>

<span data-ttu-id="7bed6-113">도메인에 대 한 [프로덕션 호스팅을](https://blogs.msdn.microsoft.com/webdev/2017/11/29/configuring-https-in-asp-net-core-across-different-platforms/) 위해서는 [인증 기관의](https://en.wikipedia.org/wiki/Certificate_authority) 인증서가 필요 합니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-113">A certificate from a [certificate authority](https://en.wikipedia.org/wiki/Certificate_authority) is required for [production hosting](https://blogs.msdn.microsoft.com/webdev/2017/11/29/configuring-https-in-asp-net-core-across-different-platforms/) for a domain.</span></span>  <span data-ttu-id="7bed6-114">[암호화](https://letsencrypt.org/) 는 무료 인증서를 제공 하는 인증 기관입니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-114">[Let's Encrypt](https://letsencrypt.org/) is a certificate authority that offers free certificates.</span></span>

<span data-ttu-id="7bed6-115">이 문서에서는 미리 빌드된 이미지 `localhost`를 호스트 하기 위해 [자체 서명 된 개발 인증서](https://en.wikipedia.org/wiki/Self-signed_certificate) 를 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-115">This document uses [self-signed development certificates](https://en.wikipedia.org/wiki/Self-signed_certificate) for hosting pre-built images over `localhost`.</span></span> <span data-ttu-id="7bed6-116">지침은 프로덕션 인증서를 사용 하는 것과 유사 합니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-116">The instructions are similar to using production certificates.</span></span>

<span data-ttu-id="7bed6-117">프로덕션 인증서의 경우:</span><span class="sxs-lookup"><span data-stu-id="7bed6-117">For production certs:</span></span>

* <span data-ttu-id="7bed6-118">도구 `dotnet dev-certs` 는 필요 하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-118">The `dotnet dev-certs` tool is not required.</span></span>
* <span data-ttu-id="7bed6-119">지침에 사용 되는 위치에 인증서를 저장할 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-119">Certificates do not need to be stored in the location used in the instructions.</span></span> <span data-ttu-id="7bed6-120">사이트 디렉터리 내에 인증서를 저장 하지 않는 것이 좋지만 모든 위치는 작동 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-120">Any location should work, although storing certs within your site directory is not recommended.</span></span>

<span data-ttu-id="7bed6-121">명령 볼륨은 컨테이너에 인증서를 탑재 합니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-121">The instructions volume mount certificates into containers.</span></span> <span data-ttu-id="7bed6-122">Dockerfile의 `COPY` 명령을 사용 하 여 컨테이너 이미지에 인증서를 추가할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-122">You can add certificates into container images with a `COPY` command in a Dockerfile.</span></span> <span data-ttu-id="7bed6-123">인증서를 이미지로 복사 하지 않는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-123">Copying certificates into an image is not recommended:</span></span>

* <span data-ttu-id="7bed6-124">개발자 인증서를 사용 하 여 테스트 하는 데 동일한 이미지를 사용 하는 것은 어렵습니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-124">It makes difficult to use the same image for testing with developer certificates.</span></span>
* <span data-ttu-id="7bed6-125">프로덕션 인증서를 사용 하 여 호스트 하는 데 동일한 이미지를 사용 하는 것은 어렵습니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-125">It makes difficult to use the same image for Hosting with production certificates.</span></span>
* <span data-ttu-id="7bed6-126">인증서 공개에는 상당한 위험이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-126">There is significant risk of certificate disclosure.</span></span>

## <a name="running-pre-built-container-images-with-https"></a><span data-ttu-id="7bed6-127">HTTPS를 사용 하 여 미리 작성 된 컨테이너 이미지 실행</span><span class="sxs-lookup"><span data-stu-id="7bed6-127">Running pre-built container images with HTTPS</span></span>

<span data-ttu-id="7bed6-128">운영 체제 구성에 대해 다음 지침을 따르십시오.</span><span class="sxs-lookup"><span data-stu-id="7bed6-128">Use the following instructions for your operating system configuration.</span></span>

### <a name="windows-using-linux-containers"></a><span data-ttu-id="7bed6-129">Linux 컨테이너를 사용 하는 Windows</span><span class="sxs-lookup"><span data-stu-id="7bed6-129">Windows using Linux containers</span></span>

<span data-ttu-id="7bed6-130">인증서 생성 및 로컬 컴퓨터 구성:</span><span class="sxs-lookup"><span data-stu-id="7bed6-130">Generate certificate and configure local machine:</span></span>

```console
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

<span data-ttu-id="7bed6-131">위의 명령에서을 암호로 바꿉니다 `{ password here }` .</span><span class="sxs-lookup"><span data-stu-id="7bed6-131">In the preceding commands, replace `{ password here }` with a password.</span></span>

<span data-ttu-id="7bed6-132">HTTPS에 대해 구성 된 ASP.NET Core를 사용 하 여 컨테이너 이미지를 실행 합니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-132">Run the container image with ASP.NET Core configured for HTTPS:</span></span>

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v %USERPROFILE%\.aspnet\https:/https/ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

<span data-ttu-id="7bed6-133">암호는 인증서에 사용 된 암호와 일치 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-133">The password must match the password used for the certificate.</span></span>

### <a name="macos-or-linux"></a><span data-ttu-id="7bed6-134">macOS 또는 Linux</span><span class="sxs-lookup"><span data-stu-id="7bed6-134">macOS or Linux</span></span>

<span data-ttu-id="7bed6-135">인증서 생성 및 로컬 컴퓨터 구성:</span><span class="sxs-lookup"><span data-stu-id="7bed6-135">Generate certificate and configure local machine:</span></span>

```console
dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

<span data-ttu-id="7bed6-136">`dotnet dev-certs https --trust`는 macOS 및 Windows 에서만 지원 됩니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-136">`dotnet dev-certs https --trust` is only supported on macOS and Windows.</span></span> <span data-ttu-id="7bed6-137">배포판에서 지 원하는 방식으로 Linux에서 인증서를 신뢰 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-137">You need to trust certs on Linux in the way that is supported by your distro.</span></span> <span data-ttu-id="7bed6-138">브라우저에서 인증서를 신뢰 해야 할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-138">It is likely that you need to trust the certificate in your browser.</span></span>

<span data-ttu-id="7bed6-139">위의 명령에서을 암호로 바꿉니다 `{ password here }` .</span><span class="sxs-lookup"><span data-stu-id="7bed6-139">In the preceding commands, replace `{ password here }` with a password.</span></span>

<span data-ttu-id="7bed6-140">HTTPS에 대해 구성 된 ASP.NET Core를 사용 하 여 컨테이너 이미지를 실행 합니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-140">Run the container image with ASP.NET Core configured for HTTPS:</span></span>

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v ${HOME}/.aspnet/https:/https/ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

<span data-ttu-id="7bed6-141">암호는 인증서에 사용 된 암호와 일치 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-141">The password must match the password used for the certificate.</span></span>

### <a name="windows-using-windows-containers"></a><span data-ttu-id="7bed6-142">Windows 컨테이너를 사용 하는 windows</span><span class="sxs-lookup"><span data-stu-id="7bed6-142">Windows using Windows containers</span></span>

<span data-ttu-id="7bed6-143">인증서 생성 및 로컬 컴퓨터 구성:</span><span class="sxs-lookup"><span data-stu-id="7bed6-143">Generate certificate and configure local machine:</span></span>

```console
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

<span data-ttu-id="7bed6-144">위의 명령에서을 암호로 바꿉니다 `{ password here }` .</span><span class="sxs-lookup"><span data-stu-id="7bed6-144">In the preceding commands, replace `{ password here }` with a password.</span></span>

<span data-ttu-id="7bed6-145">HTTPS에 대해 구성 된 ASP.NET Core를 사용 하 여 컨테이너 이미지를 실행 합니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-145">Run the container image with ASP.NET Core configured for HTTPS:</span></span>

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=\https\aspnetapp.pfx -v %USERPROFILE%\.aspnet\https:C:\https\ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

<span data-ttu-id="7bed6-146">암호는 인증서에 사용 된 암호와 일치 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="7bed6-146">The password must match the password used for the certificate.</span></span>