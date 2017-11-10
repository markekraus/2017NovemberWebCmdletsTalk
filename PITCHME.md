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

@title[Consequences of HttpClient]

### Consequences of `HttpClient`
* Strict Request Headers parsing default
* Use `-SkipHeaderValidation` to bypass for `-Headers` and `-UserAgent`

---
@title[Consequences of HttpClient (cont.)]

### Consequences of `HttpClient`
* `BasicHtmlWebResponseObject.BaseResponse` changed from `HttpWebResponse` to `HttpResponseMessage`
* Response Headers changed from `WebHeaderCollection` to `HttpHeaders`
* `BasicHtmlWebResponseObject.Headers` values are now `String[]` instead of `String`
* Content related Response Headers separated
---
@title[Consequences of HttpClient (cont.)]
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



@title[Missing Features]
## Missing Features
---

@title[Basic Parsing Only]

### Basic Parsing Only
```powershell
$url = 'https://www.google.com'
$Result = Invoke-WebRequest $url
$Result.GetType().Name
```
```none
BasicHtmlWebResponseObject
```
---

@title[HttpWebRequest to HttpClient]

### Basic Parsing Only
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

@title[Same Name Response Headers]
### Same Name Response Headers
```none
X-Header: Value1
X-Header: Value2
```
* `BasicHtmlWebResponseObject.Headers` now an array
* `BasicHtmlWebResponseObject.RawContent` now properly displays

