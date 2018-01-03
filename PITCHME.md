@title[Introduction]

## PowerShell Core

## Web Cmdlets In Depth

January 2018

@ Jacksonville PowerShell User Group

PowerShell Core 6.0.0-rc.2

---

@title[Who Am I?]

### Who Am I?

![Rin Avatar](img/rin.jpg)

* Mark Kraus
* Lead IT Solutions Architect for Mitel
* Collaborator for [PowerShell/PowerShell](https://github.com/PowerShell/PowerShell)
* PowerShell Core Contributor for Web Cmdlets
* Author of [Get-PowerShellBlog](https://get-powershellblog.blogspot.com/)
* Microsoft MVP

---

@title[Topics]

### Topics

* PowerShell Core v6.0.0-rc.2 Released
* Move from `WebRequest` to `HttpClient`
* Deprecated/Missing Features and Issues
* New Features and Fixes

---

@title[PowerShell Core v6.0.0-rc.2 Released]

### PowerShell Core v6.0.0-rc.2 Released

* RC Released 12/14
* Go forth and [install](https://github.com/PowerShell/PowerShell/releases/tag/v6.0.0-rc.2)!!
* Test all the things!!
* Open new [issues](https://github.com/PowerShell/PowerShell/issues)
* GA Release Target: January 10, 2018

---

@title[Move WebRequest to HttpClient]

## Move from `WebRequest` to `HttpClient`

---

@title[WebRequest Vs HttpClient: WebRequest]

### `WebRequest`

* Older API
* Slightly less performant
* Less attractive async
* Multiple protocols supported

---

@title[WebRequest Vs HttpClient: HttpClient]

### `HttpClient`

* More like a Headless Web Browser
* Better fit for Web Cmdlets
* More attractive async
* HTTP and HTTPS only
* Allows better optimizations
  * Reuse

---

@title[Strict Request Headers Parsing]

### Strict Request Headers Parsing

* Strict Request Headers parsing default
* Use `-SkipHeaderValidation` to bypass for `-Headers` and `-UserAgent`
* [#4085](https://github.com/PowerShell/PowerShell/pull/4085) and [#4479](https://github.com/PowerShell/PowerShell/pull/4479)

---

@title[Strict Request Headers Parsing (cont.)]

```powershell
$Params = @{
    Headers = @{"if-match" ="12345"}
    Uri = "http://httpbin.org/headers"
}
Invoke-RestMethod @Params
```

```none
Invoke-RestMethod : The format of value '12345' is invalid.
```

```powershell
$Params = @{
    Headers = @{"if-match" ="12345"}
    Uri = "http://httpbin.org/headers"
    SkipHeaderValidation = $true
}
Invoke-RestMethod @Params
```

---

@title[Strict Request Headers Parsing (cont.)]

#### For backwards compatibility:

```powershell
$command = 'Invoke-WebRequest:SkipHeaderValidation'
$PSDefaultParameterValues[$command] = $true
$command = 'Invoke-RestMethod:SkipHeaderValidation'
$PSDefaultParameterValues[$command] = $true
```

---

@title[Move from HttpWebResponse to HttpResponseMessage]

### Move from HttpWebResponse to HttpResponseMessage

* `BasicHtmlWebResponseObject.BaseResponse` changed from `HttpWebResponse` to `HttpResponseMessage`
* Response Headers changed from `WebHeaderCollection` to `HttpHeaders`
* `BasicHtmlWebResponseObject.Headers` values are now `String[]` instead of `String`
* Content related Response Headers separated

---

@title[Move from HttpWebResponse to HttpResponseMessage (cont.)]

### Move from HttpWebResponse to HttpResponseMessage (cont.)

* Error handling changed
* `Exception.Response`

```powershell
Invoke-RestMethod -Uri https://httpbin.org/status/404
$Error[0].Exception.Response.GetType().FullName
```

Windows PowerShell 5.1

```none
System.Net.HttpWebResponse
```

PowerShell Core 6.0.0

```none
System.Net.Http.HttpResponseMessage
```

---

@title[Move from HttpWebResponse to HttpResponseMessage (cont.)]

```powershell
$url = 'https://www.google.com'
$Result = Invoke-WebRequest $url
$Result.Headers.'Date'.GetType().FullName
$Result.BaseResponse.GetType().FullName
$Result.BaseResponse.Headers.GetType().FullName
$null -eq $Result.BaseResponse.Content.Headers
```

Windows PowerShell 5.1:

```none
System.String
System.Net.HttpWebResponse
System.Net.WebHeaderCollection
True
```

PowerShell Core 6.0.0:

```none
System.String[]
System.Net.Http.HttpResponseMessage
System.Net.Http.Headers.HttpResponseHeaders
False
```

---

@title[Watchout for Headers as an array]

#### Watch out for Headers as an array:

```powershell
$url = 'https://www.google.com'
$Result = Invoke-WebRequest $url
[int]$Expires = $Result.Headers.'Expires'
$Expires
```

Windows PowerShell 5.1:

```none
-1
```

PowerShell Core 6.0.0:

```none
Cannot convert the "System.String[]" value of type
"System.String[]" to type "System.Int32".
```

---

@title[Watchout for Headers (cont.)]

#### For backwards compatibility:

Comma Join:

```powershell
[int]$Expires = $Result.Headers.'Expires' -join ','
```

`Select-Object`

```powershell
[int]$Expires = $Result.Headers.'Expires' |
    Select-Object -First 1
```

Avoid index access:

```powershell
[int]$Expires = $Result.Headers.'Expires'[0]
```

Windows PowerShell 5.1:

```none
45
```

---

@title[Missing Features and Issue]

## Missing Features and Issues

---

@title[No FTP: or FILE: Support]

### No FTP: or FILE: Support

* `HttpClient` supports only `HTTP:` and `HTTPS:`
* No `FTP:` or `FILE:`
* Possibly separate cmdlet in the future
* Issue [#5491](https://github.com/PowerShell/PowerShell/issues/5491)

---

@title[Basic Parsing Only]

### Basic Parsing Only

* Parsing depended on IE and COM not available in Core
* Would not have been cross-platform
* The future is `ConvertFrom-Html`
* [AngleSharp](https://github.com/AngleSharp/AngleSharp) engine
* Hopefully for 6.1.0

---

@title[Basic Parsing Only (cont.)]

### Basic Parsing Only (cont.)

```powershell
$url = 'https://www.google.com'
$Result = Invoke-WebRequest $url
$Result.GetType().Name
```

```none
BasicHtmlWebResponseObject
```

---

@title[Basic Parsing Only (cont.)]

### Basic Parsing Only (cont.)

* `Forms` and `ParsedHtml` Removed in [#5376](https://github.com/PowerShell/PowerShell/pull/5376)

```powershell
$Result.Links.Count
$Result.Images.Count
$null -eq $Result.Forms
$null -eq $Result.ParsedHtml
```

```none
34
1
true
true
```

---

@title[No New-WebServiceProxy]

### No `New-WebServiceProxy`

* depended on `System.Web.Services.dll` not available in CoreFX
* Low priority
* Up For Grabs!

---

@title[macOS SSL/TLS/Certificate Partial Feature Support]

### macOS SSL/TLS/Certificate Partial Feature Support

* CoreFX wraps `libcurl`
* macOS `libcurl` not all using OpenSSL
* Flaky support for various new security and encryption features
* May see the same on *nix distros with Non-OpenSSL `libcurl`

---

@title[No SSL 3.0 Support]

### No SSL 3.0 support

* SSL 3.0 Deprecated
* TLS 1.0, 1.1, 1.2 supported
* No TLS 1.3/2.0 on any platforms yet

---

@title[No ServicePointManager Support]

### No ServicePointManager Support

None of these have any affect:

```powershell
[Net.ServicePointManager]::SecurityProtocol
[Net.ServicePointManager]::ServerCertificateValidationCallback
[Net.ServicePointManager]::MaxServicePoints
[Net.ServicePointManager]::MaxServicePointIdleTime
```

---

@title[No Custom Certificate Validation Support]

### No Custom Certificate Validation Support

* Relied on `System.Net.ServicePointManager`
* `HttpClient` implements on `HttpClientHandler.ServerCertificateCustomValidationCallback`
* Targeting Support in 6.1.0
* `-SkipCertificateCheck` only option for now
* [#4899](https://github.com/PowerShell/PowerShell/issues/4899) & [#4970](https://github.com/PowerShell/PowerShell/pull/4970)

---

@title[New Features and Fixes]

## New Features and Fixes

---

@title[New Parameters in Both Cmdlets]

### New Parameters in Both Cmdlets

* AllowUnencryptedAuthentication
* Authentication
* CustomMethod
* NoProxy
* PreserveAuthorizationOnRedirect

---

@title[New Parameters in Both Cmdlets (cont.)]

### New Parameters in Both Cmdlets (cont.)

* SkipCertificateCheck
* SkipHeaderValidation
* SslProtocol
* Token

---

@title[New Parameters in Invoke-RestMethod]

### New Parameters in

### Invoke-RestMethod

* FollowRelLink
* MaximumFollowRelLink
* ResponseHeadersVariable

---

@title[User-Agent Change]

### User-Agent Changes

Windows PowerShell 5.1:

```none
Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US)
 WindowsPowerShell/5.1.15063.674
```

6.0.0 Windows:

```none
Mozilla/5.0 (Windows NT 10.0; Microsoft Windows
 10.0.15063; en-US) PowerShell/6.0.0
```

---

6.0.0 Linux:

```none
Mozilla/5.0 (Linux; Linux 4.4.0-96-generic
 #119-Ubuntu SMP Tue Sep 12 14:59:54 UTC 2017;
  en-US) PowerShell/6.0.0
```

6.0.0 macOS:

```none
Mozilla/5.0 (Macintosh; Darwin 17.0.0 Darwin
 Kernel Version 17.0.0: Thu Aug 24 21:48:19
  PDT 2017; root:xnu-4570.1.46~2/RELEASE_X86_64;
 ) PowerShell/6.0.0
```

[#4914](https://github.com/PowerShell/PowerShell/pull/4914), [#4937](https://github.com/PowerShell/PowerShell/pull/4937), & [#5256](https://github.com/PowerShell/PowerShell/pull/5256)

---

@title[Authentication]

### Authentication

* `-Authentication` or `-Auth`
  * Basic
  * Bearer / OAuth
  * None (default)
* Explicit Authentication
* Possibly more in the future
* [#5052](https://github.com/PowerShell/PowerShell/pull/5052)
* [https://get-powershellblog.blogspot.com/2017/10/new-powershell-core-feature-basic-and.html](https://get-powershellblog.blogspot.com/2017/10/new-powershell-core-feature-basic-and.html)

---

@title[Authentication: Basic]

### Authentication: Basic

* RFC-7617 `Authorization: Basic` Request Header
* Uses `-Auth Basic`
* Requires `-Credential`
* Requires a `PSCredential`
* `-UseDefaultCredentials` not supported

```powershell
$Credential = Get-Credential
$uri = 'https://httpbin.org/hidden-basic-auth/user/passwd'
Invoke-RestMethod -Auth Basic -Cred $Credential -Uri $uri
```

---

@title[Authentication: Bearer and OAuth]

### Authentication: Bearer and OAuth

* RFC-6750 `Authorization: Bearer` Request Header
* Uses `-Auth OAuth` or `-Auth Bearer`
* Requires `-Token`
* Requires a `SecureString`

```powershell
$Token = Read-Host -AsSecureString -Prompt "Token"
$uri = 'https://httpbin.org/headers'
Invoke-RestMethod -Auth OAuth -Token $Token -Uri $uri
```

---

@title[Auth Errors on Non-HTTPS]

### Authentication calls over non-HTTPS Result in Errors

```powershell
$Credential = Get-Credential
$uri = 'http://google.com'
Invoke-RestMethod -Auth Basic -Cred $Credential -Uri $uri
```

```none
Invoke-RestMethod : The cmdlet cannot protect plain text
secrets sent over unencrypted connections. To suppress this
warning and send plain text secrets over unencrypted networks,
reissue the command specifying the
AllowUnencryptedAuthentication parameter.
```

---

@title[Bypass Auth Errors with -AllowUnencryptedAuthentication]

### Bypass Auth Errors with -AllowUnencryptedAuthentication

* !!! USE HTTPS !!!
* But when you can't, use `-AllowUnencryptedAuthentication`
* [#5052](https://github.com/PowerShell/PowerShell/pull/5052) & [#5402](https://github.com/PowerShell/PowerShell/pull/5402)

```powershell
$Params = @{
    Authentication = 'Basic'
    Credential = Get-Credential
    uri = 'http://google.com'
    AllowUnencryptedAuthentication = $true
}
Invoke-RestMethod @Params
```

---

@title[Same Name Response Headers]

### Same Name Response Headers

```none
X-Header: Value1
X-Header: Value2
```

* `BasicHtmlWebResponseObject.Headers` now an array
* `BasicHtmlWebResponseObject.RawContent` now properly displays
* [#4494](https://github.com/PowerShell/PowerShell/pull/4494)

---

@title[BasicHtmlWebResponseObject.Headers Performance Enhancement]

### BasicHtmlWebResponseObject.

### Headers Performance Enhancement

* Headers dictionary builds only once on first access
* Windows PowerShell creates a new dictionary on every access
* [#4853](https://github.com/PowerShell/PowerShell/pull/4853)

---

@title[-ResponseHeadersVariable on Invoke-RestMethod]

### -ResponseHeadersVariable on Invoke-RestMethod

* APIs return useful Response Headers
* Windows PowerShell `Invoke-RestMethod` has no access to Response Headers
* `-ResponseHeadersVariable` (`-RHV`) works similar to `-SessionVariable`
* Same as `BasicHtmlWebResponseObject.Headers`
* [#4888](https://github.com/PowerShell/PowerShell/pull/4888)

---

@title[-ResponseHeadersVariable on Invoke-RestMethod (cont.)]

```powershell
$Params = @{
    Uri = 'https://httpbin.org/get'
    ResponseHeadersVariable = 'RHV'
}
$Res = Invoke-RestMethod @Params
$RHV
```

```none
Key                              Value
---                              -----
Connection                       {keep-alive}
Date                             {Sun, 12 Nov 2017 15:23:57 GMT}
Via                              {1.1 vegur}
Server                           {meinheld/0.6.1}
Access-Control-Allow-Origin      {*}
Access-Control-Allow-Credentials {true}
X-Powered-By                     {Flask}
X-Processed-Time                 {0.000648021697998}
Content-Length                   {266}
Content-Type                     {application/json}
```

---

@title[-SslProtocol Parameter]

### -SslProtocol Parameter

* Supports
  * Default (TLS 1.0, TLS 1.1, and TLS 1.2)
  * Tls - TLS 1.0
  * Tls11 - TLS 1.1
  * Tls12 - TLS 1.2
* `Flags` can support multiple
* [#5329](https://github.com/PowerShell/PowerShell/pull/5329)

---

@title[-SslProtocol Parameter (cont.)]

### -SslProtocol Parameter (cont.)

```powershell
$uri = 'https://google.com'
Invoke-WebRequest -uri $uri -SslProtocol 'Ts12'
Invoke-WebRequest -uri $uri -SslProtocol 'Ts12, Tls11'
```

---

@title[-CustomMethod Parameter]

### -CustomMethod Parameter

* For custom request methods not supported by `-Method`
* [#3142](https://github.com/PowerShell/PowerShell/pull/3142)

```powershell
$uri = 'http://http.lee.io/method'
Invoke-RestMethod -uri $uri -CustomMethod 'PURIFY' |
    Select-Object -Expand Output
```

```none
method
------
PURIFY
```

---

@title[-NoProxy Parameter]

### -NoProxy Parameter

* Bypass a default proxy if one is set on the system
* [#3447](https://github.com/PowerShell/PowerShell/pull/3447)

```powershell
$uri = 'http://http.lee.io/method'
Invoke-RestMethod -uri $uri -NoProxy
```

---

@title[-PreserveAuthorizationOnRedirect Parameter]

### -PreserveAuthorizationOnRedirect Parameter

* Allows `Authorization` header to be sent when request is redirected.
* [#3885](https://github.com/PowerShell/PowerShell/pull/3885)

```powershell
$Params = @{
    Uri = 'https://httpbin.org/redirect-to?url=/get'
    Headers = @{Authorization = 'Test'}
    PreserveAuthorizationOnRedirect = $true
}
$res = Invoke-RestMethod @Params
$res.headers.Authorization
```

```none
Test
```

---

@title[-SkipCertificateCheck Parameter]

### -SkipCertificateCheck Parameter

* By passes all certificate checks
* Unsafe, but currently the only option for untrusted certs.
* [#2006](https://github.com/PowerShell/PowerShell/pull/2006)

```powershell
$uri = 'https://expired.badssl.com/'
Invoke-RestMethod -uri $uri -SkipCertificateCheck
```

---

@title[Link Header Pagination]

### Link Header Pagination

* RFC-5988 Relation `Link` Response Header
* `Invoke-RestMethod -FollowRelLink`
* Follows to "next" links
* `Invoke-RestMethod -MaximumFollowRelLink <count>`
* `Invoke-WebRequest` Returns `RelationLink` property
* [#3828](https://github.com/PowerShell/PowerShell/pull/3828) & [#5265](https://github.com/PowerShell/PowerShell/pull/5265)

---

@title[Link Header Pagination (cont.)]

### Link Header Pagination (cont.)

Build the URL:

```powershell
$baseurl = 'https://httpbin.org/response-headers?Link='
$relLink = '<https://httpbin.org/response-headers?link=done>; rel="next"'
$relLink = [uri]::EscapeDataString($relLink)
$url = '{0}{1}' -f $baseurl, $relLink
```

---

@title[Link Header Pagination (cont.)]

### Link Header Pagination (cont.)

`Invoke-WebRequest` Example:

```powershell
$Res = Invoke-WebRequest $url
$Res.RelationLink
```

```none
Key  Value
---  -----
next https://httpbin.org/response-headers?link=done
```

---

@title[Link Header Pagination (cont.)]

### Link Header Pagination (cont.)

Invoke-RestMethod example:

```powershell
Invoke-RestMethod $url -FollowRelLink -verbose
```

```none
VERBOSE: GET https://httpbin.org/response-headers?Link=
 <https:%2F%2Fhttpbin.org%2Fresponse-headers%3Flink%3Ddone>%3B rel%3D"next"
 with 0-byte payload
VERBOSE: received 118-byte response of content type application/json
VERBOSE: Content encoding: iso-8859-1
VERBOSE: Following rel link https://httpbin.org/response-headers?link=done
VERBOSE: GET https://httpbin.org/response-headers?link=done with 0-byte payload
VERBOSE: received 60-byte response of content type application/json
VERBOSE: Content encoding: iso-8859-1
Content-Type     Link
------------     ----
application/json <https://httpbin.org/response-headers?link=done>; rel="next"
application/json done
```

---

@title[Multipart/form-data Support]

### Multipart/form-data Support

* `-Body`
* `System.Net.Http.MultipartFormDataContent`
* [#4782](https://github.com/PowerShell/PowerShell/pull/4782)
* [https://get-powershellblog.blogspot.com/2017/09/multipartform-data-support-for-invoke.html](https://get-powershellblog.blogspot.com/2017/09/multipartform-data-support-for-invoke.html)

---

@title[Multipart/form-data Support (cont.)]

### Multipart/form-data Support (cont.)

```powershell
$multipartContent =
  [System.Net.Http.MultipartFormDataContent]::new()
$stringHeader =
  [System.Net.Http.Headers.ContentDispositionHeaderValue]::new(
      "form-data")
$stringHeader.Name = "TestString"
$StringContent =
  [System.Net.Http.StringContent]::new("TestValue")
$StringContent.Headers.ContentDisposition = $stringHeader
$multipartContent.Add($stringContent)
$uri = 'https://httpbin.org/post'
Invoke-RestMethod $uri -Body $multipartContent -Method 'POST' |
    Select-Object -ExpandProperty Form
```

```none
TestString
----------
TestValue
```

---

@title[Invoke-RestMethod Null JSON Literal Handling]

### Invoke-RestMethod Null JSON Literal Handling

* Invoke-RestMethod now supports single value `null` JSON Literal
* Was previously returning string `'null'`
* [#5338](https://github.com/PowerShell/PowerShell/pull/5338)

---

@title[Invoke-RestMethod Null JSON Literal Handling (cont.)]

### Invoke-RestMethod Null JSON Literal Handling (cont.)

```powershell
$uri = 'http://urlecho.appspot.com/echo'
$uri += '?status=200&Content-Type=application%2Fjson&body=null'
$result = Invoke-RestMethod -uri $uri
$null -eq $result
'null' -eq $result
```

Windows PowerShell 5.1

```none
False
True
```

PowerShell Core 6.0.0

```none
True
False
```

---

### Future Plans

* Custom Cert Validation
* Session/Process level settings
* Better Multipart/form-data support
* FTP Support
* Stream Support

---

### Thank You!

![Rin Avatar](img/rin.jpg)

* [@markekraus on Twitter](https://twitter.com/markekraus)
* [markekraus on GitHub](https://github.com/markekraus)
* [/u/markekraus on reddit](https://www.reddit.com/user/markekraus/)
* [@markekraus on Poshcode Slack](http://slack.poshcode.org/)
* [http://get-powershellblog.blogspot.com/](http://get-powershellblog.blogspot.com/)

---

### More Info

* Detailed Blog Series
  * [https://get-powershellblog.blogspot.com/2017/11/powershell-core-web-cmdlets-in-depth.html](https://get-powershellblog.blogspot.com/2017/11/powershell-core-web-cmdlets-in-depth.html)
* Slides:
  * [https://github.com/markekraus/2017NovemberWebCmdletsTalk/tree/2018January](https://github.com/markekraus/2017NovemberWebCmdletsTalk/tree/2018January)

---

## Q & A