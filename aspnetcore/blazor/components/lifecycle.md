---
title: ASP.NET Core Blazor 수명 주기
author: guardrex
description: ASP.NET Core Blazor 앱에서 Razor 구성 요소 수명 주기 메서드를 사용하는 방법을 알아봅니다.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 11/06/2020
no-loc:
- appsettings.json
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
uid: blazor/components/lifecycle
ms.openlocfilehash: 03a49c827a1f70e6b721adf293857bb33475ed36
ms.sourcegitcommit: 04ad9cd26fcaa8bd11e261d3661f375f5f343cdc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/10/2021
ms.locfileid: "100107079"
---
# <a name="aspnet-core-blazor-lifecycle"></a>ASP.NET Core Blazor 수명 주기

Blazor 프레임워크는 동기 및 비동기 수명 주기 메서드를 포함합니다. 구성 요소 초기화 및 렌더링 중에 구성 요소에서 추가 작업을 수행하려면 수명 주기 메서드를 재정의합니다.

다음 다이어그램은 Blazor 수명 주기를 보여 줍니다. 수명 주기 메서드는 이 문서의 다음 섹션에 있는 예제를 사용하여 정의됩니다.

구성 요소 수명 주기 이벤트:

1. 구성 요소가 요청 시 처음 렌더링되는 경우:
   * 구성 요소의 인스턴스를 만듭니다.
   * 속성 삽입을 수행합니다. [`SetParametersAsync`](#before-parameters-are-set) 를 실행합니다.
   * [`OnInitialized{Async}`](#component-initialization-methods)를 호출합니다. <xref:System.Threading.Tasks.Task>가 반환되면 <xref:System.Threading.Tasks.Task>가 대기한 다음, 구성 요소가 렌더링됩니다. <xref:System.Threading.Tasks.Task>가 반환되지 않으면 구성 요소가 렌더링됩니다.
1. [`OnParametersSet{Async}`](#after-parameters-are-set)를 호출하고 구성 요소를 렌더링합니다. <xref:System.Threading.Tasks.Task>가 `OnParametersSetAsync`에서 반환되면 <xref:System.Threading.Tasks.Task>가 대기한 다음 구성 요소가 렌더링됩니다.

![Blazor 내 Razor 구성 요소의 구성 요소 수명 주기 이벤트](lifecycle/_static/lifecycle1.png)

DOM(문서 개체 모델) 이벤트 처리:

1. 이벤트 처리기가 실행됩니다.
1. <xref:System.Threading.Tasks.Task>가 반환되면 <xref:System.Threading.Tasks.Task>가 대기한 다음, 구성 요소가 렌더링됩니다. <xref:System.Threading.Tasks.Task>가 반환되지 않으면 구성 요소가 렌더링됩니다.

![DOM(문서 개체 모델) 이벤트 처리](lifecycle/_static/lifecycle2.png)

`Render` 수명 주기:

1. 다음의 경우에 구성 요소에서 추가 렌더링 작업을 방지합니다.
   * 첫 번째 렌더링 이후
   * [`ShouldRender`](#suppress-ui-refreshing)가 `false`인 경우
1. 렌더링 트리 diff(차이)를 빌드하고 구성 요소를 렌더링합니다.
1. DOM이 업데이트될 때까지 기다립니다.
1. [`OnAfterRender{Async}`](#after-component-render)를 호출합니다.

![렌더링 수명 주기](lifecycle/_static/lifecycle3.png)

개발자가 [`StateHasChanged`](#state-changes)를 호출하면 렌더러가 생성됩니다. 자세한 내용은 <xref:blazor/components/rendering>를 참조하세요.

## <a name="lifecycle-methods"></a>수명 주기 메서드

### <a name="before-parameters-are-set"></a>매개 변수를 설정하기 전

<xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A>는 렌더링 트리 또는 경로 매개 변수에서 구성 요소의 부모가 제공하는 매개 변수를 설정합니다. 메서드를 재정의하면 개발자 코드가 <xref:Microsoft.AspNetCore.Components.ParameterView>의 매개 변수와 직접 상호 작용할 수 있습니다.

다음 예제에서 `Param`의 경로 매개 변수 구문 분석이 성공하는 경우 <xref:Microsoft.AspNetCore.Components.ParameterView.TryGetValue%2A?displayProperty=nameWithType>는 `Param` 매개 변수의 값을 `value`에 할당합니다. `value`가 `null`이 아니면 값이 구성 요소에 의해 표시됩니다.

[경로 매개 변수 일치는 대/소문자를 구분하지 않지만](xref:blazor/fundamentals/routing#route-parameters) <xref:Microsoft.AspNetCore.Components.ParameterView.TryGetValue%2A>는 경로 템플릿에서 대/소문자를 구분하는 매개 변수 이름만 일치시킵니다. 다음 예제에서 값을 가져오려면 `/{param?}`가 아닌 `/{Param?}`를 사용해야 합니다. 이 시나리오에서 `/{param?}`를 사용하면 <xref:Microsoft.AspNetCore.Components.ParameterView.TryGetValue%2A>는 `false`를 반환하고 `message`는 문자열 중 하나로 설정되지 않습니다.

`Pages/SetParametersAsyncExample.razor`:

```razor
@page "/setparametersasync-example/{Param?}"

<h1>SetParametersAsync Example</h1>

<p>@message</p>

@code {
    private string message;

    [Parameter]
    public string Param { get; set; }

    public override async Task SetParametersAsync(ParameterView parameters)
    {
        if (parameters.TryGetValue<string>(nameof(Param), out var value))
        {
            if (value is null)
            {
                message = "The value of 'Param' is null.";
            }
            else
            {
                message = $"The value of 'Param' is {value}.";
            }
        }

        await base.SetParametersAsync(parameters);
    }
}
```

<xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A>를 호출할 때마다 <xref:Microsoft.AspNetCore.Components.ParameterView>에 구성 요소의 매개 변수 값 세트가 포함됩니다.

<xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A>의 기본 구현에서는 <xref:Microsoft.AspNetCore.Components.ParameterView>에 해당 값이 있는 [`[Parameter]`](xref:Microsoft.AspNetCore.Components.ParameterAttribute) 또는 [`[CascadingParameter]` 특성](xref:Microsoft.AspNetCore.Components.CascadingParameterAttribute)으로 각 속성의 값을 설정합니다. <xref:Microsoft.AspNetCore.Components.ParameterView>에 해당 값이 없는 매개 변수는 변경되지 않고 그대로 유지됩니다.

[`base.SetParametersAsync`](xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A)를 호출하지 않은 경우, 사용자 지정 코드는 들어오는 매개 변수 값을 필요한 방식으로 해석할 수 있습니다. 예를 들어 들어오는 매개 변수를 클래스의 속성에 할당해야 하는 요구 사항이 없습니다.

이벤트 처리기가 설정된 경우 삭제 시 해당 처리기를 언후크합니다. 자세한 내용은 [`IDisposable`을 사용한 구성 요소 삭제](#component-disposal-with-idisposable) 섹션을 참조하세요.

### <a name="component-initialization-methods"></a>구성 요소 초기화 메서드

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> 및 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A>는 <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A>에서 구성 요소가 부모 구성 요소에서 초기 매개 변수를 받은 후 초기화할 때 호출됩니다. 

구성 요소가 비동기 작업을 수행하며, 작업이 완료될 때 새로 고쳐야 하는 경우 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>를 사용합니다.

동기 작업의 경우 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A>를 재정의합니다.

```csharp
protected override void OnInitialized()
{
    ...
}
```

비동기 작업을 수행하려면 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>를 재정의하고 작업에 [`await`](/dotnet/csharp/language-reference/operators/await) 연산자를 사용합니다.

```csharp
protected override async Task OnInitializedAsync()
{
    await ...
}
```

[콘텐츠를 미리 렌더링](xref:blazor/fundamentals/signalr#render-mode)하는 Blazor Server 앱이 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>를 ‘두 번’ 호출합니다.

* 첫 번째 호출: 구성 요소가 처음에 페이지 일부로 정적 렌더링될 때
* 두 번째 호출: 브라우저가 서버에 다시 연결할 때

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>의 개발자 코드가 두 번 실행되는 것을 방지하려면 [미리 렌더링한 후의 상태 저장 다시 연결](#stateful-reconnection-after-prerendering) 섹션을 참조하세요.

Blazor Server 앱을 미리 렌더링 중이면 브라우저에 연결되어 있지 않으므로 JavaScript 호출 등의 특정 작업을 수행할 수 없습니다. 미리 렌더링된 경우 구성 요소를 다르게 렌더링해야 할 수 있습니다. 자세한 내용은 [앱을 미리 렌더링 중인 경우 검색](#detect-when-the-app-is-prerendering) 섹션을 참조하세요.

이벤트 처리기가 설정된 경우 삭제 시 해당 처리기를 언후크합니다. 자세한 내용은 [`IDisposable`을 사용한 구성 요소 삭제](#component-disposal-with-idisposable) 섹션을 참조하세요.

### <a name="after-parameters-are-set"></a>매개 변수를 설정한 후

다음 두 가지 경우에 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> 또는 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSet%2A>가 호출됩니다.

* 구성 요소가 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A> 또는 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>에서 초기화된 후
* 부모 구성 요소가 다시 렌더링되고 다음을 제공하는 경우
  * 하나 이상의 매개 변수가 변경된, 알려진 변경 불가능 기본 형식만
  * 복합 형식 매개 변수. 프레임워크는 복합 형식 매개 변수의 값이 내부적으로 변경되었는지 아닌지를 알 수 없으므로 매개 변수 집합을 변경된 것으로 처리합니다.

```csharp
protected override async Task OnParametersSetAsync()
{
    await ...
}
```

> [!NOTE]
> 매개 변수와 속성 값을 적용하는 경우 비동기 작업은 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> 수명 주기 이벤트 중에 수행해야 합니다.

```csharp
protected override void OnParametersSet()
{
    ...
}
```

이벤트 처리기가 설정된 경우 삭제 시 해당 처리기를 언후크합니다. 자세한 내용은 [`IDisposable`을 사용한 구성 요소 삭제](#component-disposal-with-idisposable) 섹션을 참조하세요.

### <a name="after-component-render"></a>구성 요소 렌더링 후

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> 및 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A>는 구성 요소 렌더링을 완료한 후에 호출됩니다. 이때 요소 및 구성 요소 참조가 채워집니다. 렌더링된 DOM 요소에서 작동하는 타사 JavaScript 라이브러리 활성화와 같은 추가 초기화 단계를 렌더링된 콘텐츠로 수행하려면 이 단계를 사용합니다.

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> 및 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A>의 `firstRender` 매개 변수와 관련해서 다음 사항을 확인합니다.

* 구성 요소 인스턴스를 처음 렌더링할 때는 `true`로 설정됩니다.
* 초기화 작업이 한 번만 수행되도록 하는 데 사용할 수 있습니다.

```csharp
protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (firstRender)
    {
        await ...
    }
}
```

> [!NOTE]
> 렌더링 직후의 비동기 작업은 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> 수명 주기 이벤트 중에 수행해야 합니다.
>
> <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A>에서 <xref:System.Threading.Tasks.Task>를 반환하는 경우에도, 해당 작업이 완료된 후에는 프레임워크에서 구성 요소에 대해 추가 렌더링 주기를 예약하지 않습니다. 이 동작은 무한 렌더링 루프를 방지하기 위한 것입니다. 반환된 작업이 완료된 후 추가 렌더링 주기를 예약하는 기타 수명 주기 메서드와는 다릅니다.

```csharp
protected override void OnAfterRender(bool firstRender)
{
    if (firstRender)
    {
        ...
    }
}
```

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> 및 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A>는 ‘서버에서 미리 렌더링 프로세스 중에는 호출되지 않습니다’. 이 메서드는 미리 렌더링이 완료된 후 구성 요소가 대화형으로 렌더링될 때 호출됩니다. 앱이 미리 렌더링되는 경우:

1. 구성 요소가 서버에서 실행되어 HTTP 응답에 몇 가지 정적 HTML 태그를 생성합니다. 이 단계에서는 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> 및 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A>가 호출되지 않습니다.
1. `blazor.server.js` 또는 `blazor.webassembly.js`가 브라우저에서 시작되면 구성 요소가 대화형 렌더링 모드로 다시 시작됩니다. 구성 요소가 다시 시작되면 앱이 더 이상 미리 렌더링 단계에 포함되지 않기 때문에 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> 및 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A>가 **호출됩니다**.

이벤트 처리기가 설정된 경우 삭제 시 해당 처리기를 언후크합니다. 자세한 내용은 [`IDisposable`을 사용한 구성 요소 삭제](#component-disposal-with-idisposable) 섹션을 참조하세요.

### <a name="suppress-ui-refreshing"></a>UI 새로 고침 표시 안 함

UI 새로 고침을 표시하지 않으려면 <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A>를 재정의합니다. 구현에서 `true`를 반환하는 경우 UI가 새로 고쳐집니다.

```csharp
protected override bool ShouldRender()
{
    var renderUI = true;

    return renderUI;
}
```

구성 요소를 렌더링할 때마다 <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A>가 호출됩니다.

<xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A>를 재정의한 경우에도 처음에는 구성 요소가 항상 렌더링됩니다.

자세한 내용은 <xref:blazor/webassembly-performance-best-practices#avoid-unnecessary-rendering-of-component-subtrees>를 참조하세요.

## <a name="state-changes"></a>상태 변경

<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A>는 구성 요소에 상태가 변경되었음을 알립니다. 해당하는 경우, <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A>를 호출하면 구성 요소가 다시 렌더링됩니다.

<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A>는 <xref:Microsoft.AspNetCore.Components.EventCallback> 메서드에 대해 자동으로 호출됩니다. 자세한 내용은 <xref:blazor/components/event-handling#eventcallback>를 참조하세요.

자세한 내용은 <xref:blazor/components/rendering>를 참조하세요.

## <a name="handle-incomplete-async-actions-at-render"></a>렌더링 시 불완전한 비동기 작업 처리

수명 주기 이벤트에서 수행한 비동기 작업이 구성 요소를 렌더링하기 전에 완료되지 않았을 수 있습니다. 수명 주기 메서드를 실행하는 동안 개체가 `null`이거나 불완전하게 데이터로 채워졌을 수 있습니다. 개체가 초기화되었는지 확인하는 렌더링 논리를 제공합니다. 개체가 `null`인 동안 자리 표시자 UI 요소(예: 로드 메시지)를 렌더링합니다.

Blazor 템플릿의 `FetchData` 구성 요소에서 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>는 예측 데이터(`forecasts`)를 비동기적으로 수신하도록 재정의되었습니다. `forecasts`가 `null`인 경우 사용자에게 로드 메시지가 표시됩니다. <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>에서 반환된 `Task`가 완료되면 구성 요소가 업데이트된 상태로 다시 렌더링됩니다.

Blazor Server 템플릿의 `Pages/FetchData.razor`:

[!code-razor[](lifecycle/samples_snapshot/FetchData.razor?highlight=9,21,25)]

## <a name="handle-errors"></a>오류 처리

수명 주기 메서드 실행 중의 오류 처리 방법에 대한 자세한 내용은 <xref:blazor/fundamentals/handle-errors#lifecycle-methods>를 참조하세요.

## <a name="stateful-reconnection-after-prerendering"></a>미리 렌더링 후의 상태 저장 다시 연결

Blazor Server 앱에서 <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper.RenderMode>가 <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.ServerPrerendered>인 경우 구성 요소는 처음에 페이지 일부로 정적 렌더링됩니다. 브라우저가 서버에 다시 연결하면 구성 요소가 ‘다시’ 렌더링되고, 이제 대화형 구성 요소가 됩니다. 구성 요소를 초기화하기 위한 [`OnInitialized{Async}`](#component-initialization-methods) 수명 주기 메서드가 있는 경우 메서드가 다음과 같이 ‘두 번’ 실행됩니다.

* 구성 요소를 정적으로 미리 렌더링할 때
* 서버 연결이 설정된 후

이 동작 때문에 구성 요소를 최종적으로 렌더링할 때는 UI에 표시되는 데이터가 상당히 변경될 수 있습니다.

Blazor Server 앱에서 이중 렌더링 시나리오를 방지하려면 다음을 수행합니다.

* 미리 렌더링 중에 상태를 캐시하고 앱이 다시 시작된 후 상태를 검색하는 데 사용할 수 있는 식별자를 전달합니다.
* 미리 렌더링 중에 식별자를 사용하여 구성 요소 상태를 저장합니다.
* 미리 렌더링 후에 식별자를 사용하여 캐시된 상태를 검색합니다.

다음 코드는 템플릿 기반 Blazor Server 앱에서 이중 렌더링을 방지하도록 업데이트된 `WeatherForecastService`를 보여 줍니다.

```csharp
public class WeatherForecastService
{
    private static readonly string[] summaries = new[]
    {
        "Freezing", "Bracing", "Chilly", "Cool", "Mild",
        "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
    };
    
    public WeatherForecastService(IMemoryCache memoryCache)
    {
        MemoryCache = memoryCache;
    }
    
    public IMemoryCache MemoryCache { get; }

    public Task<WeatherForecast[]> GetForecastAsync(DateTime startDate)
    {
        return MemoryCache.GetOrCreateAsync(startDate, async e =>
        {
            e.SetOptions(new MemoryCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = 
                    TimeSpan.FromSeconds(30)
            });

            var rng = new Random();

            await Task.Delay(TimeSpan.FromSeconds(10));

            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
            {
                Date = startDate.AddDays(index),
                TemperatureC = rng.Next(-20, 55),
                Summary = summaries[rng.Next(summaries.Length)]
            }).ToArray();
        });
    }
}
```

<xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper.RenderMode>에 대한 자세한 내용은 <xref:blazor/fundamentals/signalr#render-mode>를 참조하세요.

## <a name="detect-when-the-app-is-prerendering"></a>앱을 미리 렌더링 중인 경우 검색

[!INCLUDE[](~/blazor/includes/prerendering.md)]

## <a name="component-disposal-with-idisposable"></a>`IDisposable`로 구성 요소 삭제

구성 요소가 <xref:System.IDisposable>을 구현하는 경우 프레임워크는 관리되지 않는 리소스를 해제할 수 있는 UI에서 해당 구성 요소가 제거될 때 [삭제 메서드](/dotnet/standard/garbage-collection/implementing-dispose)를 호출합니다. [구성 요소 초기화](#component-initialization-methods) 도중을 포함하여 언제든지 삭제가 수행될 수 있습니다. 다음 구성 요소는 [`@implements`](xref:mvc/views/razor#implements) Razor 지시문을 사용하여 <xref:System.IDisposable>을 구현합니다.

```razor
@using System
@implements IDisposable

...

@code {
    public void Dispose()
    {
        ...
    }
}
```

개체를 삭제해야 하는 경우 <xref:System.IDisposable.Dispose%2A?displayProperty=nameWithType>를 호출할 때 람다를 사용하여 개체를 삭제할 수 있습니다.

`Pages/CounterWithTimerDisposal.razor`:

```razor
@page "/counter-with-timer-disposal"
@using System.Timers
@implements IDisposable

<h1>Counter with <code>Timer</code> disposal</h1>

<p>Current count: @currentCount</p>

@code {
    private int currentCount = 0;
    private Timer timer = new Timer(1000);

    protected override void OnInitialized()
    {
        timer.Elapsed += (sender, eventArgs) => OnTimerCallback();
        timer.Start();
    }

    private void OnTimerCallback()
    {
        _ = InvokeAsync(() =>
        {
            currentCount++;
            StateHasChanged();
        });
    }

    public void IDisposable.Dispose() => timer.Dispose();
}
```

위 예제는 <xref:blazor/components/rendering#receiving-a-call-from-something-external-to-the-blazor-rendering-and-event-handling-system>에 표시됩니다.

비동기 삭제 작업에는 <xref:System.IDisposable.Dispose> 대신 `DisposeAsync`를 사용합니다.

```csharp
public async ValueTask DisposeAsync()
{
    ...
}
```

> [!NOTE]
> `Dispose`에서 <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> 호출은 지원되지 않습니다. 렌더러를 삭제하는 과정에서 <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A>가 호출될 수 있으므로, 해당 시점에 UI 업데이트를 요청할 수는 없습니다.

.NET 이벤트에서 이벤트 처리기의 구독을 취소합니다. 다음 [Blazor 양식](xref:blazor/forms-validation) 예제에서는 `Dispose` 메서드에서 이벤트 처리기를 구독 취소하는 방법을 보여 줍니다.

* 프라이빗 필드 및 람다 접근 방식

  [!code-razor[](lifecycle/samples_snapshot/event-handler-disposal-1.razor?highlight=23,28)]

* 프라이빗 메서드 접근 방식

  [!code-razor[](lifecycle/samples_snapshot/event-handler-disposal-2.razor?highlight=16,26)]

[익명 함수](/dotnet/csharp/programming-guide/statements-expressions-operators/anonymous-functions), 메서드 또는 식이 사용되는 경우에는 <xref:System.IDisposable>을 구현하고 대리자를 구독 취소할 필요가 없습니다. 그러나 대리자를 구독 취소하지 못하면 **이벤트를 노출하는 개체가 대리자를 등록하는 구성 요소보다 수명이 긴 경우** 문제가 됩니다. 이 경우 등록된 대리자가 원래 개체를 활성 상태로 유지하기 때문에 메모리 누수가 발생합니다. 따라서 이벤트 대리자가 신속하게 삭제됨을 알고 있는 경우에만 다음 접근 방식을 사용합니다. 삭제해야 하는 개체의 수명이 확실하지 않으면 대리자 메서드를 구독하고 위 예제에 표시된 대로 대리자를 적절하게 삭제합니다.

* 익명 람다 메서드 접근 방식(명시적 삭제 필요하지 않음)

  ```csharp
  private void HandleFieldChanged(object sender, FieldChangedEventArgs e)
  {
      formInvalid = !editContext.Validate();
      StateHasChanged();
  }

  protected override void OnInitialized()
  {
      editContext = new EditContext(starship);
      editContext.OnFieldChanged += (s, e) => HandleFieldChanged((editContext)s, e);
  }
  ```

* 익명 람다 식 접근 방식(명시적 삭제 필요하지 않음)

  ```csharp
  private ValidationMessageStore messageStore;

  [CascadingParameter]
  private EditContext CurrentEditContext { get; set; }

  protected override void OnInitialized()
  {
      ...

      messageStore = new ValidationMessageStore(CurrentEditContext);

      CurrentEditContext.OnValidationRequested += (s, e) => messageStore.Clear();
      CurrentEditContext.OnFieldChanged += (s, e) => 
          messageStore.Clear(e.FieldIdentifier);
  }
  ```

  익명 람다 식이 포함된 위 코드의 전체 예제는 <xref:blazor/forms-validation#validator-components>에 표시됩니다.

자세한 내용은 [관리되지 않는 리소스 정리](/dotnet/standard/garbage-collection/unmanaged)와 `Dispose` 및 `DisposeAsync` 메서드 구현에 이어지는 토픽을 참조하세요.

## <a name="cancelable-background-work"></a>취소할 수 있는 백그라운드 작업

구성 요소는 네트워크 호출(<xref:System.Net.Http.HttpClient>) 및 데이터베이스와의 상호 작용 등과 같은 장기 실행 백그라운드 작업을 수행하는 경우가 많습니다. 여러 상황에서 시스템 리소스를 절약하기 위해 백그라운드 작업을 중지하는 것이 좋습니다. 예를 들어 사용자가 구성 요소에서 이동할 때 백그라운드 비동기 작업이 자동으로 중지되지 않습니다.

백그라운드 작업 항목에 취소가 필요할 수 있는 다른 이유는 다음과 같습니다.

* 잘못된 입력 데이터 또는 처리 매개 변수를 사용하여 실행 중인 백그라운드 작업이 시작되었습니다.
* 현재 실행 중인 백그라운드 작업 항목 집합을 새 작업 항목 집합으로 바꾸어야 합니다.
* 현재 실행 중인 작업의 우선 순위를 변경해야 합니다.
* 서버에 다시 배포하기 위해 앱을 종료해야 합니다.
* 서버 리소스가 제한되므로 백그라운드 작업 항목을 다시 예약해야 합니다.

구성 요소에서 취소할 수 있는 백그라운드 작업 패턴을 구현하는 방법은 다음과 같습니다.

* <xref:System.Threading.CancellationTokenSource> 및 <xref:System.Threading.CancellationToken>을 사용합니다.
* [구성 요소 삭제](#component-disposal-with-idisposable) 시 및 토큰을 수동으로 취소하여 언제든지 취소가 필요할 때 [`CancellationTokenSource.Cancel`](xref:System.Threading.CancellationTokenSource.Cancel%2A)을 호출하여 백그라운드 작업을 취소해야 한다고 알립니다.
* 비동기 호출이 반환된 후에는 토큰에서 <xref:System.Threading.CancellationToken.ThrowIfCancellationRequested%2A>를 호출합니다.

다음 예제에서는

* `await Task.Delay(5000, cts.Token);`는 장기 실행 비동기 백그라운드 작업을 나타냅니다.
* `BackgroundResourceMethod`는 호출되기 전에 `Resource`가 삭제된 경우 시작하지 않아야 하는 장기 실행 백그라운드 메서드를 나타냅니다.

```razor
@implements IDisposable
@using System.Threading

<button @onclick="LongRunningWork">Trigger long running work</button>

@code {
    private Resource resource = new Resource();
    private CancellationTokenSource cts = new CancellationTokenSource();

    protected async Task LongRunningWork()
    {
        await Task.Delay(5000, cts.Token);

        cts.Token.ThrowIfCancellationRequested();
        resource.BackgroundResourceMethod();
    }

    public void Dispose()
    {
        cts.Cancel();
        cts.Dispose();
        resource.Dispose();
    }

    private class Resource : IDisposable
    {
        private bool disposed;

        public void BackgroundResourceMethod()
        {
            if (disposed)
            {
                throw new ObjectDisposedException(nameof(Resource));
            }
            
            ...
        }
        
        public void Dispose()
        {
            disposed = true;
        }
    }
}
```

## <a name="blazor-server-reconnection-events"></a>Blazor Server 재연결 이벤트

이 문서에서 설명하는 구성 요소 수명 주기 이벤트는 [Blazor Server의 재연결 이벤트 처리기](xref:blazor/fundamentals/signalr#reflect-the-connection-state-in-the-ui)와 별도로 작동합니다. Blazor Server 앱의 클라이언트에 대한 SignalR 연결이 끊어지는 경우 UI 업데이트만 중단됩니다. UI 업데이트는 연결이 다시 설정될 때 다시 시작됩니다. 회로 처리기 이벤트 및 구성에 대한 자세한 내용은 <xref:blazor/fundamentals/signalr> 문서를 참조하세요.
