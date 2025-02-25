# 使用 cURL 下载文件指南

[![推广](https://github.com/luminati-io/LinkedIn-Scraper/blob/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/)

在本指南中，你将看到：

- [cURL 下载文件最基本的用法](#cURL-下载文件最基本的用法)
- [下载文件时处理更复杂的场景](#使用-cURL-下载文件-高级选项)
- [如何一次性下载多个文件](#使用-cURL-下载文件的最佳实践)
- [一些高效使用 cURL 的最佳实践](#一些高效使用-cURL-的最佳实践)
- [cURL 与 Wget 下载文件的简要对比](#cURL-与-Wget-下载文件的对比)

让我们开始吧！

---

## cURL 下载文件最基本的用法

下面是最基本的 cURL 下载文件语法：

```powershell
curl -O <file_url>
```

> 💡 **重要说明：**  
> 在 Windows 上，`curl` 是 Windows PowerShell 中 [`Invoke-WebRequest`](https://github.com/bright-cn/Invoke-web-request-proxy) 的别名。为了避免冲突，请将 `curl` 替换为 `curl.exe`。

[`-O`](https://curl.se/docs/manpage.html#-O) 和 `--remote-name` 参数会让 cURL 使用原始文件名保存下载的文件：

```powershell
curl --remote-name <file_url>
```  
例如，考虑以下 cURL 命令来下载文件：

```powershell
curl -O "https://i.imgur.com/CSRiAeN.jpg"
```

此时将会出现一个下载进度条，如下所示：

```
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 35354  100 35354    0     0   155k      0 --:--:-- --:--:-- --:--:--  158k
```

当进度条到达 100% 时，将在你运行 cURL 命令的文件夹中生成一个名为 `CSRiAeN.jpg` 的文件：

![下载完成后，文件夹中出现名为 CSRiAeN.jpg 的文件](https://github.com/bright-cn/curl-download-files/blob/main/image-37.png)

更多关于 cURL 及其选项的信息，请参阅 [我们的 cURL 指南](https://www.bright.cn/blog/web-data/what-is-curl)。

## 使用 cURL 下载文件：高级选项

让我们学习一些其他选项。

### 自定义下载后的文件名

默认情况下，`-O` 选项会使用远程文件的原始名称来保存。如果 URL 中的远程文件没有文件名，cURL 会创建一个没有扩展名、名为 `curl_response` 的文件：

![使用默认文件名的 curl_response 文件](https://github.com/bright-cn/curl-download-files/blob/main/image-38.png)

同时，cURL 会进行警告提示，说明这种情况：

```
Warning: No remote file name, uses "curl_response"
```

要为下载的文件指定其它名字，可以使用 [`-o`](https://curl.se/docs/manpage.html#-o)（或 `--output`）参数：

```powershell
curl "https://i.imgur.com/CSRiAeN.jpg" -o "logo.jpg"
```

cURL 会对指定的文件 URL 执行 GET 请求，并将下载的内容保存为 “`-o`” 后的名称。此时，保存下来的文件会是一个名为 `logo.jpg` 的文件：

![下载完成后，文件夹中出现名为 logo.jpg 的文件](https://github.com/bright-cn/curl-download-files/blob/main/image-39.png)

### 跟随重定向

有些 URL 不会直接指向目标文件，需要自动重定向才能到达最终的下载地址。

对此，可以使用 [`-L`](https://curl.se/docs/manpage.html#-L) 选项让 cURL 跟随重定向：

```powershell
curl -O -L "<file_url>"
```

否则，cURL 将会在输出中显示重定向的请求头，而不会访问 `Location` 头中指示的新地址。

### 在服务器上进行身份认证

有些服务器会限制访问，需要进行用户身份认证。要执行基本的 HTTP 或 FTP 认证，可以使用 [`-u`](https://curl.se/docs/manpage.html#-u)（或 `--user`）选项，并指定用户名和密码，格式如下：

```powershell
<username>:<password>
```

该格式无法在用户名中包含冒号，但密码可以包含冒号。

`<password>` 字符串是可选的。如果只指定用户名，则 cURL 会提示你输入密码。

以下是使用 cURL 进行服务器身份认证并下载文件的语法：

```powershell
curl -O -u <username>:<password> <file_url>
```

例如，可以像这样下载一个需要身份认证的文件：

```powershell
curl -O -u "myUser:myPassword" "https://example.com/secret.txt"
```

### 限制下载带宽

如果想避免占用全部可用带宽，可以使用 [`--limit-rate`](https://curl.se/docs/manpage.html#--limit-rate) 选项，并在后面指定你要设置的最大下载速度：

```powershell
curl -O --limit-rate 5k "https://i.imgur.com/CSRiAeN.jpg"
```

输出类似如下：

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 35354  100 35354    0     0   5166      0  0:00:06  0:00:06 --:--:--  5198
```

`--limit-rate` 选项可帮助控制带宽占用，避免占用网络资源或遵守带宽限制，或者模拟网络较慢的情况以进行测试。

### 通过代理服务器下载文件

若想保护隐私或规避[防机器人机制（如限速）](https://www.bright.cn/blog/web-data/anti-scraping-techniques)，隐藏你的 IP 并通过代理服务器路由请求，可以使用 [`-x`](https://curl.se/docs/manpage.html#-x)（或 `--proxy`）选项：

```powershell
curl -x <proxy_url> -O <file_url>
```

`<proxy_url>` 的格式如下：

```powershell
[protocol://]host[:port]
```

代理 URL 会因使用的是 HTTP、HTTPS 或 SOCKS 代理而有所不同。更多详情可参阅 [我们的 cURL 代理整合指南](https://www.bright.cn/blog/proxy-101/curl-with-proxies)。

例如使用 HTTP 代理时，命令可写为：

```powershell
curl -x "http://proxy.example.com:8080" -O "https://i.imgur.com/CSRiAeN.jpg"
```

### 后台下载

若想在下载过程中不显示进度条及错误信息，可以使用 [`-s`](https://curl.se/docs/manpage.html#-s)（或 `--silent`）选项启用“静默模式”：

```powershell
curl -O -s "https://i.imgur.com/CSRiAeN.jpg"
```

如果下载成功，文件会出现在当前目录中，而终端里不会有任何提示。

### 显示详细信息

若出现错误或者你想了解 cURL 在后台做了什么，可以添加 [`-v`](https://curl.se/docs/manpage.html#-v)（或 `--verbose`）选项来开启详细模式：

```powershell
curl -O -v "https://i.imgur.com/CSRiAeN.jpg"
```

这样会输出更多背景信息：

```
* IPv6: (none)
* IPv4: 146.75.52.193
*   Trying 146.75.52.193:443...
* schannel: disabled automatic use of client certificate
* ALPN: curl offers http/1.1
* ALPN: server accepted http/1.1
* Connected to i.imgur.com (146.75.52.193) port 443
* using HTTP/1.x
> GET /CSRiAeN.jpg HTTP/1.1
> Host: i.imgur.com
> User-Agent: curl/8.10.1
> Accept: */*
>
* Request completely sent off
* schannel: failed to decrypt data, need more data
< HTTP/1.1 200 OK
< Connection: keep-alive
< Content-Length: 35354
< Content-Type: image/jpeg
< Last-Modified: Wed, 08 Jan 2025 08:02:49 GMT
< ETag: "117b93e0521ba1313429bad28b3befc8"
< x-amz-server-side-encryption: AES256
< X-Amz-Cf-Pop: IAD89-P1
< X-Amz-Cf-Id: wTQ20stgw0Ffl1BRmhRhFqpCXY_2hnBLbPXn9D8LgPwdjL96xarRVQ==
< cache-control: public, max-age=31536000
< Accept-Ranges: bytes
< Age: 2903
< Date: Wed, 08 Jan 2025 08:51:12 GMT
< X-Served-By: cache-iad-kiad7000028-IAD, cache-lin1730072-LIN
< X-Cache: Miss from cloudfront, HIT, HIT
< X-Cache-Hits: 1, 0
< X-Timer: S1736326272.410959,VS0,VE1
< Strict-Transport-Security: max-age=300
< Access-Control-Allow-Methods: GET, OPTIONS
< Access-Control-Allow-Origin: *
< Server: cat factory 1.0
< X-Content-Type-Options: nosniff
<
{ [1371 bytes data]
100 35354  100 35354    0     0   212k      0 --:--:-- --:--:-- --:--:--  214k
* Connection #0 to host i.imgur.com left intact.
```

### 设置简化进度条

你可以使用 `-#`（或 `--progress-bar`）选项启用更简单的进度条：

```powershell
curl -O -# "https://i.imgur.com/CSRiAeN.jpg"
```

它会以 `#` 号的形式显示进度条并逐渐填充，直到文件下载完成：

```
########################################################### 100.0%
```

## 如何使用 cURL 下载多个文件

### 范围批量下载

对于同一个远程 URL 的多个文件，你可以使用大括号 `{}` 来指定：

```powershell
curl -O "https://example.com/images/{1.jpg,2.jpg,3.jpg}"
```

这会下载以下三个文件：

```
1.jpg
2.jpg
3.jpg
```

在这种场景下，你也可以使用方括号 `[]` 的正则表达式语法：

```powershell
curl -O "https://example.com/files/file[1-3].jpg"
```

> **注意：**  
> 所有自定义选项（如 `-s` 静默模式或 `--limit-rate` 带宽限制）都会应用到所有文件的下载中。

### 多个文件来源下载

如果想从不同的 URL 下载文件，需要多次使用 `-O` 选项：

```powershell
curl -O "https://i.imgur.com/CSRiAeN.jpg" -O "https://brightdata.com/wp-content/uploads/2020/12/upload_blog_20201220_153903.svg"
```

输出中会针对每个 URL 分别显示下载进度：

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 35354  100 35354    0     0   271k      0 --:--:-- --:--:-- --:--:--  276k
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 22467    0 22467    0     0  34657      0 --:--:-- --:--:-- --:--:-- 34724
```

同样，你可以多次使用 `-o` 来为不同 URL 的文件指定自定义名称：

```powershell
curl "https://i.imgur.com/CSRiAeN.jpg" -o "logo.jpg" "https://brightdata.com/wp-content/uploads/2020/12/upload_blog_20201220_153903.svg" -o "blog_post.svg"
```

也可以混合使用 `-O` 和 `-o`：

```powershell
curl "https://i.imgur.com/CSRiAeN.jpg" -o "logo.jpg" -O "https://brightdata.com/wp-content/uploads/2020/12/upload_blog_20201220_153903.svg"
```

> **注意：**  
> 像 `-v`、`-s` 或 `--limit-rate` 这类选项，都会同时指向所有的 URL，所以只需指定一次。

## 使用 cURL 下载文件的最佳实践

下面列出一些使用 cURL 下载文件时的重要最佳实践：

- **在 Windows 上使用 `curl.exe`** 以避免与 `Invoke-WebRequest` cmdlet 的冲突。  
- **忽略 HTTPS 和 SSL/TLS 报错** 可使用 `-k`（或 `--insecure`）选项。  
- **指定正确的 HTTP 方法**：使用 `-X` 指定 GET、POST 或 PUT。  
- **用引号包裹 URL 并对特殊字符转义**（如 `\`）以避免空格、& 等字符导致的问题。  
- **设置代理以保护身份**：使用 `-x`（或 `--proxy`）选项。  
- **跨不同请求保存并重用 cookies**：使用 `-c` 和 `-b` 选项。  
- **限制下载速度以更好地控制**：使用 `--limit-rate` 选项。  
- **启用详细输出以便调试**：使用 `-v` 选项。  
- **检查错误响应**：可使用 `-w` 选项输出 HTTP 响应码并进行检验。

## cURL 与 Wget 下载文件的对比

- **cURL：** 对数据传输有更细粒度的控制，支持自定义头、身份认证、多种协议。  
- **Wget：** 更加简单，适合批量下载、递归下载及断点续传等场景。

## 结论

每当你发起一次 HTTP 请求时，你都会在网络上留下痕迹。为了保护你的身份和隐私，并增强安全性，考虑将代理与 cURL 结合使用：

- [数据中心代理](https://www.bright.cn/proxy-types/datacenter-proxies) – 超过 770,000 个数据中心 IP。  
- [住宅代理](https://www.bright.cn/proxy-types/residential-proxies) – 超过 72,000,000 个住宅 IP，覆盖 195 多个国家。  
- [ISP 代理](https://www.bright.cn/proxy-types/isp-proxies) – 超过 700,000 个 ISP IP。  
- [移动代理](https://www.bright.cn/proxy-types/mobile-proxies) – 超过 7,000,000 个移动 IP。

[立即注册](https://www.bright.cn)，免费测试我们的代理和爬虫解决方案！
