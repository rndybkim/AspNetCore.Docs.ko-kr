Azure Portal은 AAD 테넌트에 [확인되거나 확인되지 않은 게시자 도메인](/azure/active-directory/develop/howto-configure-publisher-domain)이 있는지 여부를 기준으로 다음 앱 ID URI 형식 중 하나를 제공합니다.

* `api://{SERVER API APP CLIENT ID OR CUSTOM VALUE}`
* `https://{TENANT}.onmicrosoft.com/{API CLIENT ID OR CUSTOM VALUE}`

위 앱 ID URI 중 두 번째를 클라이언트 앱에서 사용하여 범위 URI를 구성했는데 권한 부여에 실패한 경우에는 스키마 및 호스트 없이 범위 URI를 제공해 보세요.

```csharp
options.ProviderOptions.DefaultAccessTokenScopes.Add(
    "{SERVER API APP CLIENT ID OR CUSTOM VALUE}/{SCOPE NAME}");
```

예제:

```csharp
options.ProviderOptions.DefaultAccessTokenScopes.Add(
    "41451fa7-82d9-4673-8fa5-69eff5a761fd/API.Access");
```
