@title[Introduction]
## PowerShell Core Web Cmdlets In Depth
#### November 2017 
#### @ The next North Texas PC Users Group (NTPCUG) PowerShell SIG

PowerShell Core 6.0.0-Beta.9

---

@title[Who Am I?]
### Who Am I?
![Rin Avatar](img/rin.jpg)
* Mark Kraus
* Lead IT Solutions Architect for Mitel
* Collaborator for [PowerShell/PowerShell](https://github.com/PowerShell/PowerShell)
* PowerShell Core Contributor for Web Cmdlets
* Author of [Get-PowerShellBlog](https://get-powershellblog.blogspot.com/)

---

@title[Topics]

### Topics

* Move from `HttpWebRequest` to `HttpClient`
* Deprecated and/or Missing Features
* New Features and Fixes

---

@title[Move HttpWebRequest to HttpClient]

## Move from `HttpWebRequest` to `HttpClient`

---

@title[HttpWebRequest Vs HttpClient: HttpWebRequest]

### `HttpWebRequest`

* Older API
* Slightly less performant
* Less attractive async

---

@title[HttpWebRequest Vs HttpClient: HttpClient]

### `HttpClient`

* More like a Headless Web Browser
* Better fit for Web Cmdlets
* More attractive async
* Allows better optimizations
  * Reuse
  * Session level settings
---

@title[Strict Request Headers Parsing]

### Consequences of `HttpClient`
* Strict Request Headers parsing default
* Use `-SkipHeaderValidation` to bypass for `-Headers` and `-UserAgent`
* [#4085](https://github.com/PowerShell/PowerShell/pull/4085) and [#4479](https://github.com/PowerShell/PowerShell/pull/4479)

---

@title[Strict Request Headers Parsing (cont.)]
```powershell
$headers = @{}
$headers.Add("if-match","12345")
Invoke-WebRequest -Uri "http://httpbin.org/headers" -Headers $headers
```
```none
Invoke-WebRequest : The format of value '12345' is invalid.
At line:1 char:1
+ Invoke-WebRequest -Uri "http://httpbin.org/headers" -Headers $headers
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Invoke-WebRequest], FormatException
    + FullyQualifiedErrorId : System.FormatException,Microsoft.PowerShell.Commands.InvokeWebRequestCommand
```
```powershell
$headers = @{}
$headers.Add("if-match","12345")
Invoke-WebRequest -Uri "http://httpbin.org/headers" -Headers $headers -SkipHeaderValidation
```

---

@title[Strict Request Headers Parsing (cont.)]
#### For backwards compatibility:
```powershell
$PSDefaultParameterValues['Invoke-WebRequest:SkipHeaderValidation'] = $true
$PSDefaultParameterValues['Invoke-RestMethod:SkipHeaderValidation'] = $true
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
```powershell
$url = 'https://www.google.com'
$Result = Invoke-WebRequest $url
$Result.Headers.'Date'.GetType().FullName
$Result.BaseResponse.GetType().FullName
$Result.BaseResponse.Headers.GetType().FullName
$null -eq $Result.BaseResponse.Content.Headers
```
#### Windows PowerShell 5.1
```none
System.String
System.Net.HttpWebResponse
System.Net.WebHeaderCollection
True
```
#### PowerShell Core 6.0.0-Beta.9
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
#### Windows PowerShell 5.1
```none
-1
```
#### PowerShell Core 6.0.0-Beta.9
```none
Cannot convert the "System.String[]" value of type "System.String[]" to type "System.Int32".
At line:1 char:1
+ [int]$Expires = $Result.Headers.'Expires'
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : MetadataError: (:) [], ArgumentTransformationMetadataException
    + FullyQualifiedErrorId : RuntimeException
```

---

@title[Watchout for Headers (cont.)]
#### For backwards compatibility:
* Comma Join
```powershell
[int]$Expires = $Result.Headers.'Expires' -join ','
```
* `Select-Object`
```powershell
[int]$Expires = $Result.Headers.'Expires' | Select-Object -First 1
```
* Avoid index access:
```powershell
[int]$Expires = $Result.Headers.'Expires'[0]
$Expires
```
#### Windows PowerShell 5.1
```none
45
```

---

@title[Missing Features]
## Missing Features

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
### Basic Parsing Only
* Removed in [#5376](https://github.com/PowerShell/PowerShell/pull/5376)
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

---

@title[macOS SSL/TLS/Certificate Partial Feature Support]
### macOS SSL/TLS/Certificate Partial Feature Support
* CoreFX wraps `curl`
* macOS `curl` not all using OpenSSL
* Flaky support for various new security and encryption features
* May see the same on *nix distros with Non-OpenSSL `curl`

---

@title[New Features and Fixes]
## New Features and Fixes

---

@title[Authentication]

### Authentication
* `-Authentication` or `-Auth` Added with support for:
  * Basic
  * Bearer
  * None (default)
  * OAuth
* Explicit Authentication
* Possibly more in the future
* [5052](https://github.com/PowerShell/PowerShell/pull/5052)
* [https://get-powershellblog.blogspot.com/2017/10/new-powershell-core-feature-basic-and.html](https://get-powershellblog.blogspot.com/2017/10/new-powershell-core-feature-basic-and.html)

---

@title[Authentication: Basic]
### Authentication: Basic
* RFC-7617 `Authorization: Basic` Request Header
* Uses `-Auth Basic`
* Requires `-Credential` 
* Requires a `PSCredential`
```powershell
$Credential = Get-Credential
$uri = 'https://httpbin.org/hidden-basic-auth/user/passwd'
Invoke-RestMethod -Authentication Basic -Credential $Credential -Uri $uri
```

---

@title[Authentication: Bearer and OAuth]
### Authentication: Bearer and OAuth
* RFC-6750 `Authorization: Bearer` Request Header
* Uses `-Auth OAuth` or `-Auth Bearer`
* Requires `-Token`
* Requires a `SecureString`
```powershell
$Token = Read-Host -AsSecureString -Prompt "Enter OAuth Access Token"
$uri = 'https://httpbin.org/headers'
Invoke-RestMethod -Authentication Basic -Token $Token -Uri $uri
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
Invoke-RestMethod : The cmdlet cannot protect plain text secrets sent over unencrypted connections. To supress this
warning and send plain text secrets over unencrypted networks, reissue the command specifying the
AllowUnencryptedAuthentication parameter.
```

--- 

@title[Bypass Auth Errors with -AllowUnencryptedAuthentication]
### Bypass Auth Errors with -AllowUnencryptedAuthentication
* !!! USE HTTPS !!!
* But when you can't, use `-AllowUnencryptedAuthentication`
```powershell
$Credential = Get-Credential
$uri = 'http://google.com'
Invoke-RestMethod -Auth Basic -Cred $Credential -Uri $uri -AllowUnencryptedAuthentication
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

---



