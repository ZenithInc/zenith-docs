# HTTP Request

我们可以使用 requests 库发起 HTTP 请求。

## 安装 {id="pip-install"}

```shell
python3 -m pip install requests
```

## 基本使用 {id="basic-example"}

然后引入该库:

```python
import request
```

接着，发起 HTTP GET 请求:

```python
r = requests.get('http://api.mock.end.wiki/test.json')
print(r.text)   # 输出接口数据，文本格式
print(r.json()) # 将接口数据转成 JSON 格式
```

## HTTPS 证书问题

编写了一个 Python的 脚步，使用 request 库请求一个 HTTPS 的页面，然后出现了如下错误：

```text
requests.exceptions.SSLError: HTTPSConnectionPool(host='httpbin.ceshiren.com', port=443): Max retries exceeded with url: /get (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate 
```

这个错误是由于SSL证书验证失败引起的，通常是因为请求的HTTPS站点的SSL证书未能验证通过或者Python环境中缺少必要的根证书文件。

要解决这个问题，你可以尝试以下方法。通常情况下，第一种方法是最安全和推荐的方法，因为它使用了经过验证的根证书。第二种方法只应在测试或开发环境中使用，而第三种方法可能会更加麻烦并不保证成功。

### 更新或安装根证书 {id="update-or-install-root-cert"}

确保你的Python环境中具有最新的根证书。你可以下载最新的根证书包，然后配置Python使用它。在Python 3.4+ 版本中，你可以使用 `certifi` 库来管理根证书。首先安装 `certifi`：

```
pip install certifi
```

然后在你的代码中导入并设置根证书文件路径：

```python
import certifi
import requests

# 设置根证书路径
requests.get('https://example.com', verify=certifi.where())
```

这将使用 `certifi` 提供的根证书文件来验证SSL证书。

###  禁用SSL证书验证 {id="disable-ssl-verify"}

你可以在请求中禁用SSL证书验证，但这不是安全的做法，不建议在生产环境中使用。你可以这样做：

```python
import requests

# 禁用SSL证书验证
requests.get('https://example.com', verify=False)
```

这将跳过SSL证书验证，但可能会导致安全风险，因此只用于测试或开发目的。

### 安装缺少的根证书 {id="install-root-cert"}

如果你在Python环境中缺少根证书，你可以尝试手动安装它。根证书文件通常位于操作系统的某个特定目录中，你可以查找相关文档来了解如何手动安装根证书。

### 仍然无法解决 {id="other-problems"}

如果你仍然遇到 SSL 证书验证失败的问题，即使使用了 `certifi` 库，可能存在其他问题。以下是一些可能的原因和解决方法：

1.  **证书链问题**：SSL 证书通常包含一个证书链，包括站点证书和中间证书颁发机构（CA）的证书。确保站点的证书链是完整的。有时站点的服务器可能没有正确配置，导致证书链不完整。你可以使用一些在线工具来检查站点的证书链是否正确。

2.  **证书更新问题**：SSL 证书有时会过期，或者站点的证书可能已经被吊销。确保站点的证书是有效的。你可以使用浏览器访问站点并检查证书的有效性。

3.  **CA 根证书问题**：如果站点使用了自定义的根证书，而不是公共的根证书颁发机构，你需要确保你的系统中包含了该根证书。这可能需要手动安装根证书。

4.  **检查代理设置**：如果你的请求经过代理服务器，确保代理服务器的设置正确，并且不会干扰SSL证书验证。

5.  **更新Python和requests库**：确保你的Python和requests库是最新版本，以确保可能存在的SSL问题得到修复。

经过检查，发现是因为第一点，也就是证书链的问题。可以通过 [WhatsMyChainCert](https://whatsmychaincert.com/)  这个网站声称包含 root 的证书链，然后下载到本地后:

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/L8CAmDf5bnowEEanhAdJ.png" alt="what's my chain cert"/>

写入代码进行验证:

```Python
import requests
import requests

def test_get():
    url = 'https://httpbin.ceshiren.com/get'
    cert_path = "/path/httpbin.ceshiren.com.chained+root.crt"
    r = requests.get(url, verify=cert_path)
    return r.json()

result = test_get()
print(result)
```

## 总结 {id="summary"}

这篇文档主要讲了如何安装并使用 `requests` 库，并着重讲述了当请求 HTTPS 资源的时候，可能会出现的问题，以及如何解决。