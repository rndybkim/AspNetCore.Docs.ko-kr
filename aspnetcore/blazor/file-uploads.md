---
title: ASP.NET Core Blazor 파일 업로드
author: guardrex
description: InputFile 구성 요소를 사용하여 Blazor에서 파일을 업로드하는 방법을 알아봅니다.
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 09/29/2020
no-loc:
- ASP.NET Core Identity
- cookie
- Cookie
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: blazor/file-uploads
ms.openlocfilehash: 06d1464cb731a8008362fc911f463e4ff8a37b6b
ms.sourcegitcommit: d1a897ebd89daa05170ac448e4831d327f6b21a8
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/01/2020
ms.locfileid: "91606653"
---
# <a name="aspnet-core-no-locblazor-file-uploads"></a><span data-ttu-id="397e6-103">ASP.NET Core Blazor 파일 업로드</span><span class="sxs-lookup"><span data-stu-id="397e6-103">ASP.NET Core Blazor file uploads</span></span>

<span data-ttu-id="397e6-104">작성자: [Daniel Roth](https://github.com/danroth27), [Pranav Krishnamoorthy](https://github.com/pranavkm)</span><span class="sxs-lookup"><span data-stu-id="397e6-104">By [Daniel Roth](https://github.com/danroth27) and [Pranav Krishnamoorthy](https://github.com/pranavkm)</span></span>

<span data-ttu-id="397e6-105">[예제 코드 살펴보기 및 다운로드](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/file-uploads/samples/) ([다운로드 방법](xref:index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="397e6-105">[View or download sample code](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/file-uploads/samples/) ([how to download](xref:index#how-to-download-a-sample))</span></span>

<span data-ttu-id="397e6-106">`InputFile` 구성 요소를 사용하여 파일 업로드용을 비롯한 브라우저 파일 데이터를 .NET 코드로 읽습니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-106">Use the `InputFile` component to read browser file data into .NET code, including for file uploads.</span></span>

> [!WARNING]
> <span data-ttu-id="397e6-107">항상 파일 업로드 보안 모범 사례를 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-107">Always follow file upload security best practices.</span></span> <span data-ttu-id="397e6-108">자세한 내용은 <xref:mvc/models/file-uploads#security-considerations>를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="397e6-108">For more information, see <xref:mvc/models/file-uploads#security-considerations>.</span></span>

## <a name="inputfile-component"></a><span data-ttu-id="397e6-109">`InputFile` 구성 요소</span><span class="sxs-lookup"><span data-stu-id="397e6-109">`InputFile` component</span></span>

<span data-ttu-id="397e6-110">`InputFile` 구성 요소는 `file` 형식의 HTML 입력으로 렌더링됩니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-110">The `InputFile` component renders as an HTML input of type `file`.</span></span>

<span data-ttu-id="397e6-111">기본적으로 사용자는 단일 파일을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-111">By default, the user selects single files.</span></span> <span data-ttu-id="397e6-112">사용자가 한 번에 여러 파일을 업로드할 수 있도록 `multiple` 특성을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-112">Add the `multiple` attribute to permit the user to upload multiple files at once.</span></span> <span data-ttu-id="397e6-113">사용자가 하나 이상의 파일을 선택하면 `InputFile` 구성 요소는 `OnChange` 이벤트를 발생시키고 선택한 파일 목록 및 각 파일에 대한 세부 정보에 액세스할 수 있는 `InputFileChangeEventArgs`를 전달합니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-113">When one or more files is selected by the user, the `InputFile` component fires an `OnChange` event and passes in an `InputFileChangeEventArgs` that provides access to the selected file list and details about each file.</span></span>

<span data-ttu-id="397e6-114">사용자가 선택한 파일에서 데이터를 읽으려면</span><span class="sxs-lookup"><span data-stu-id="397e6-114">To read data from a user-selected file:</span></span>

* <span data-ttu-id="397e6-115">파일에 대한 `OpenReadStream`을 호출하고 반환된 스트림에서 읽습니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-115">Call `OpenReadStream` on the file and read from the returned stream.</span></span> <span data-ttu-id="397e6-116">자세한 내용은 [파일 스트림](#file-streams) 섹션을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="397e6-116">For more information, see the [File streams](#file-streams) section.</span></span>
* <span data-ttu-id="397e6-117">`ReadAsync`을 사용하세요.</span><span class="sxs-lookup"><span data-stu-id="397e6-117">Use `ReadAsync`.</span></span> <span data-ttu-id="397e6-118">기본적으로 `ReadAsync`는 524,288KB(512KB)보다 작은 파일만 읽을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-118">By default, `ReadAsync` only allows reading a file smaller than 524,288 KB (512 KB) in size.</span></span> <span data-ttu-id="397e6-119">이 제한은 개발자가 실수로 메모리에 대규모 파일을 읽어 들이는 것을 방지하기 위한 것입니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-119">This limit is present to prevent developers from accidentally reading large files in to memory.</span></span> <span data-ttu-id="397e6-120">더 큰 파일을 지원해야 할 경우 최대 예상 파일 크기에 적절한 근사치를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-120">Specify a reasonable approximation for the maximum expected file size if larger files must be supported.</span></span> <span data-ttu-id="397e6-121">들어오는 파일 스트림을 메모리로 직접 읽지 않도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-121">Avoid reading the incoming file stream directly into memory.</span></span> <span data-ttu-id="397e6-122">예를 들어 파일 바이트를 <xref:System.IO.MemoryStream>에 복사하거나 바이트 배열로 읽지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-122">For example, don't copy file bytes into a <xref:System.IO.MemoryStream> or read as a byte array.</span></span> <span data-ttu-id="397e6-123">이러한 접근 방식은 특히 Blazor Server에서 성능 및 보안 문제를 초래할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-123">These approaches can result in performance and security problems, especially in Blazor Server.</span></span> <span data-ttu-id="397e6-124">대신 Blob 또는 디스크의 파일과 같은 외부 저장소에 파일 바이트를 복사하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-124">Instead, consider copying file bytes to an external store, such as a a blob or a file on disk.</span></span>

<span data-ttu-id="397e6-125">이미지 파일을 수신하는 구성 요소는 파일에 대한 `RequestImageFileAsync` 편의 메서드를 호출하여 이미지가 앱으로 스트리밍되기 전에 브라우저의 JavaScript 런타임 내에서 이미지 데이터의 크기를 조정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-125">A component that receives an image file can call the `RequestImageFileAsync` convenience method on the file to resize the image data within the browser's JavaScript runtime before the image is streamed into the app.</span></span>

<span data-ttu-id="397e6-126">다음 예제에서는 한 구성 요소에서 여러 이미지 파일을 업로드하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-126">The following example demonstrates multiple image file upload in a component.</span></span> <span data-ttu-id="397e6-127">`InputFileChangeEventArgs.GetMultipleFiles`는 여러 파일을 읽을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-127">`InputFileChangeEventArgs.GetMultipleFiles` allows reading multiple files.</span></span> <span data-ttu-id="397e6-128">악의적인 사용자가 앱에서 예상하는 것보다 많은 수의 파일을 업로드하지 못하도록 예상되는 최대 읽기 파일 수를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-128">Specify the maximum number of files you expect to read to prevent a malicious user from uploading a larger number of files than the app expects.</span></span> <span data-ttu-id="397e6-129">`InputFileChangeEventArgs.File`은 파일 업로드가 여러 파일을 지원하지 않을 경우 첫 번째 및 유일한 파일을 읽을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-129">`InputFileChangeEventArgs.File` allows reading the first and only file if the file upload does not support multiple files.</span></span>

```razor
<h3>Upload PNG images</h3>

<p>
    <InputFile OnChange="@OnInputFileChange" multiple />
</p>

@if (imageDataUrls.Count > 0)
{
    <h4>Images</h4>

    <div class="card" style="width:30rem;">
        <div class="card-body">
            @foreach (var imageDataUrl in imageDataUrls)
            {
                <img class="rounded m-1" src="@imageDataUrl" />
            }
        </div>
    </div>
}

@code {
    IList<string> imageDataUrls = new List<string>();

    private async Task OnInputFileChange(InputFileChangeEventArgs e)
    {
        var maxAllowedFiles = 3;
        var format = "image/png";

        foreach (var imageFile in e.GetMultipleFiles(maxAllowedFiles))
        {
            var resizedImageFile = await imageFile.RequestImageFileAsync(format, 
                100, 100);
            var buffer = new byte[resizedImageFile.Size];
            await resizedImageFile.OpenReadStream().ReadAsync(buffer);
            var imageDataUrl = 
                $"data:{format};base64,{Convert.ToBase64String(buffer)}";
            imageDataUrls.Add(imageDataUrl);
        }
    }
}
```

<span data-ttu-id="397e6-130">`IBrowserFile`은 [브라우저](https://developer.mozilla.org/docs/Web/API/File#Instance_properties)에서 속성으로 노출하는 메타데이터를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-130">`IBrowserFile` returns metadata [exposed by the browser](https://developer.mozilla.org/docs/Web/API/File#Instance_properties) as properties.</span></span> <span data-ttu-id="397e6-131">이 메타데이터는 예비 유효성 검사에 유용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-131">This metadata can be useful to preliminary validation.</span></span> <span data-ttu-id="397e6-132">예제는 [`FileUpload.razor` 및 `FilePreview.razor` 샘플 구성 요소](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/file-uploads/samples/)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="397e6-132">For example, see the [`FileUpload.razor` and `FilePreview.razor` sample components](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/file-uploads/samples/).</span></span>

## <a name="file-streams"></a><span data-ttu-id="397e6-133">파일 스트림</span><span class="sxs-lookup"><span data-stu-id="397e6-133">File streams</span></span>

<span data-ttu-id="397e6-134">Blazor WebAssembly 앱에서 데이터는 브라우저 내의 .NET 코드로 직접 스트리밍됩니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-134">In a Blazor WebAssembly app, the data is streamed directly into the .NET code within the browser.</span></span>

<span data-ttu-id="397e6-135">Blazor Server 앱에서는 스트림에서 파일을 읽을 때 파일 데이터가 SignalR 연결을 통해 서버의 .NET 코드로 스트리밍됩니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-135">In a Blazor Server app, the file data is streamed over the SignalR connection into .NET code on the server as the file is read from the stream.</span></span> <span data-ttu-id="397e6-136">[`Forms.RemoteBrowserFileStreamOptions`](https://github.com/dotnet/aspnetcore/blob/master/src/Components/Web/src/Forms/InputFile/RemoteBrowserFileStreamOptions.cs)를 통해 Blazor Server에 대한 파일 업로드 특성을 구성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="397e6-136">[`Forms.RemoteBrowserFileStreamOptions`](https://github.com/dotnet/aspnetcore/blob/master/src/Components/Web/src/Forms/InputFile/RemoteBrowserFileStreamOptions.cs) allows configuring file upload characteristics for Blazor Server.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="397e6-137">추가 리소스</span><span class="sxs-lookup"><span data-stu-id="397e6-137">Additional resources</span></span>

* <xref:mvc/models/file-uploads#security-considerations>
* <xref:blazor/forms-validation>