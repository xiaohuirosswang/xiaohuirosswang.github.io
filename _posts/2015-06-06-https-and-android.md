---
layout: post
title: "HTTPS 证书链以及 Android 应用中的 HTTPS 实现问题"
description: ""
category: [Android, HTTPS]
---

## 背景

Google 在 [Android 6.0 Changes](http://developer.android.com/about/versions/marshmallow/android-6.0-changes.html) 中声称在 Android 6.0 系统中，移除了 Apache HttpClient 组件“

> Android 6.0 release removes support for the Apache HTTP client. If your app is using this client and targets Android 2.3 (API level 9) or higher, use the HttpURLConnection class instead. This API is more efficient because it reduces network use through transparent compression and response caching, and minimizes power consumption. To continue using the Apache HTTP APIs, you must first declare the following compile-time dependency in your build.gradle file: 
> 
	android {
    useLibrary 'org.apache.http.legacy'
	}

所以，为了适配最新的 Android 6.0 系统，作为开发者，如果之前使用了 Apache HttpClient 开发的网络通信模块，是时候使用 URLConnection 以及 Derived 类型来重新实现，毕竟调整后能得到上面 Google 提到的各种好处，而且不需要依赖一个额外的包。

在我们使用智能移动设备的时候，很多时候我们可能处于不安全的 Wifi 环境中，考虑到用户数据在传输过程中的安全性，`Https` 协议越来越成为网络通信的主流，但是使用 `Https` 的时候如果实现有误，还是避免不了 [MITMA](http://en.wikipedia.org/wiki/Man-in-the-middle_attack) （Man in the middle attack），国外有篇论文 [Detection of SSL-related security
vulnerabilities in Android
applications
](https://grahamedgecombe.com/work/diss.pdf) 对这种情况作了很好的总结，推荐大家看看。而且 [SSL Vulnerabilities at Large](https://www.fireeye.com/blog/threat-research/2014/08/ssl-vulnerabilities-who-listens-when-android-applications-talk.html) 文章经过统计发现 __73%__ 的 Android 应用程序没有正确地在 `Https` 通信过程中进行证书验证。

这篇文章就是想讨论一下如何正确地使用 `HttpsURLConnection` 实现 `Https` 通信。除了本文，[Android security - Implementation of Self-signed SSL certificate for your App.](http://www.codeproject.com/Articles/826045/Android-security-Implementation-of-Self-signed-SSL) 也非常推荐，但遗憾的是该文最后给出的完整实例使用了 Apache HttpClient 的实现方式，不过该文对 Android 应用开发中如何正确处理 `Https` 通信做了非常好的分析。

## Https 证书

### 证书链（Certificate Chains）

我们一般常见的证书链分为两种：

- 二级证书：直接由 `受信任的根证书颁发机构` 颁发的证书（CRT 文件），由于这种情况下一旦 Root CA 证书遭到破坏或者泄露，提供这个 Certificate Authority 的机构之前颁发的证书就全部失去安全性了，需要全部换掉，对这个 CA 也是毁灭性打击，现在主流的商业 CA 都提供三级证书。
- 三级证书：由 `受信任的根证书颁发机构` 下的 `中级证书颁发机构` 颁发的证书，这样 ROOT CA 就可以离线放在一个物理隔离的安全的地方，即使这个 CA 的中级证书被破坏或者泄露，虽然后果也很严重，但根证书还在，可以再生成一个中级证书重新颁发证书，而且这种情况对 HTTPS 的性能和证书安装过程也没有太大影响，这种方式也基本成为主流做法。

我们的互联网就是运行在这个基于信任关系的基础上，国际上的 `受信任的根证书颁发机构` 是有限的几个机构，具体信息可以参考 [维基百科的介绍][4]。

### 如何获得可用于 HTTPS 的证书

#### 生成 openssl 生成一个私钥：

	# 生成使用 RSA 非对称加密类型的私钥，使用 DES3 算法，输出 OpenSSL 格式，采用 2048 位强度。server.key是文件名，生成过程需要提供一个至少四位的密码。
	openssl genrsa -des3 -out server.key 2048

在部署支持 HTTPS 网站的时候 Web Server 每次启动都需要提供使用的私钥密码，可以使用下面的命令去除刚生成的私钥密码：

	openssl rsa -in server.key -out server_nopwd.key

#### 生成 CSR （Certificate Signing Request）文件：

	openssl req -new -key server_nopwd.key -out server.csr

在这个过程中需要提供像国家、地区、组织名称以及 E-mail 等信息，对于用于 HTTPS 的 CSR 来说，`Common Name` 必须和网站域名一致，以便之后进行 Host Name 校验，所以 `Common Name` 的选择比较重要，可以选择 `Single Name` 或者 `WildCard` 类型的，二者区别可见 [Choosing the SSL Certificate Common Name](https://support.dnsimple.com/articles/ssl-certificate-hostname/)。上面的命令执行时会需要输入一些信息：

> Country Name (2 letter code) [AU]:CN  
State or Province Name (full name) [Some-State]:BJ  
Locality Name (eg, city) []:BJ 
Organization Name (eg, company) [Internet Widgits Pty Ltd]:OrgName  
Organizational Unit Name (eg, section) []:OrgUnit  
Common Name (eg, YOUR name) []:www.DOMAIN.com  
Email Address []:Name@DOMAIN.com

#### 生成 CRT 证书文件

Self-Signed Certificate 自签名证书，如果只是用于比如应用 API 接口，不需要用于可通过浏览器访问的网页展示，为了节省开支或者仅仅是内部使用可以考虑。

	openssl x509 -req -days 365 -in server.csr -signkey server_nopwd.key -out server.crt

提交 CSR 文件并购买授信中级证书机构颁发的证书

	openssl x509 -md5 -days 3560 -req -CA ca.crt -CAkey ca.key -CAcreateserial -CAserial ca.srl -in server.csr -out server.crt

有的 CA 签名后会提供两个 CRT 文件，一个是对提交的 CSR 文件签名后的 CRT，一个是 CA 自己的 Intermediate CRT 文件，这个 CRT 文件是公开的，比如 Symantic 的常见 Intermediate CRT 就可以从 [Symantec Class 3 Secure Server CA - G4](https://www.tbs-certificates.co.uk/FAQ/en/Symantec_Class_3_Secure_Server_CA-G4_MPKI.html) 下载到，GoDaddy 的所有不同格式的 CRT 文件都公开在 [GoDaddy Repository](https://certs.godaddy.com/repository/)，为了让配置的 Https 网站被浏览器认为是可信的，这两个 CRT 文件可以合并后配置到 Web Server 中，这相当于在 Web Server 服务器上安装了该 CA 的 Intermediate CRT 证书。合并方式可以参考 [How do I make my own bundle file from CRT files?](https://support.comodo.com/index.php?/Default/Knowledgebase/Article/View/643/17/)

使用 CRT 文件和 私钥配置 HTTPS 网站服务

这里的具体操作方式取决于使用的 Web Server 类型，比如 Apache 和 Nginx 有自己的配置方法，就不列了，各个 Web Server 的文档都有详细说明，GoDaddy 的文档 [INSTALL SSL CERTIFICATES](https://www.godaddy.com/help/install-ssl-certificates-16623) 对主流的 Web Server 如何进行 `Https` 配置作了非常全面的总结。

## Https 网站

这里我们可以使用一个可用于测试的 Python Flask 实现的 Https Web Server：

    from OpenSSL import SSL
	from flask import Flask
	from flask import render_template

	app = Flask(__name__)
	context = SSL.Context(SSL.SSLv23_METHOD)
	context.use_privatekey_file('/Users/rwang/testssl/test.key')
	context.use_certificate_file('/Users/rwang/testssl/test.crt')


	@app.route("/")
	def hello():
		return "Howdy, there!", 200 

	if __name__ == "__main__":
    	app.run(host='127.0.0.1',port=8443, 
        	debug = True, ssl_context=context)

启动这个测试服务器，就可以从浏览器中通过 `https://127.0.0.1:8443` 这个地址访问了，如果你使用了 `Self signed Certificate`，浏览器会提示危险，选择仍然继续就可以看到 `Howdy, there!` 的欢迎语了。注意，如果需要通过在同一个局域网内的 Android 设备访问这个测试服务器，请把 `127.0.0.1` 替换为本机在局域网内的 ip。

## HttpsURLConnection 

这里分两种情况来讨论。

- 使用 Self signed Certificate

	自签名证书可以用于 Android 应用程序的 Api 接口通信中，按照 [Certificate pinning with self-singed certificate](http://www.codeproject.com/Articles/826045/Android-security-Implementation-of-Self-signed-SSL) 的说法，有下面的有点：

	> __Increased security__ - with pinned SSL certificates, the app is independent of the device’s trust store. Compromising the hard coded trust store in the app is not so easy - the app would need to be decompiled, changed and then recompiled again - and it can’t be signed using the same Android keystore that the original developer of the app used.  
	__Reduced costs__ - SSL certificate pinning gives you the possibility to use a self-signed certificate that can be trusted. For example, you’re developing an app that uses your own API server. You can reduce the costs by using a self-signed certificate on your server (and pinning that certificate in your app) instead of paying for a certificate. Although a bit convoluted, this way, you've actually improved security and saved yourself some money.

	当然，文中也提到使用自签名证书也有不足的地方：

	> __Less flexibility__ - when you do SSL certificate pinning, changing the SSL certificate is not that easy. For every SSL certificate change, you have to make an update to the app, push it to Google Play and hope the users will install it.

	使用自签名证书，需要自定义 `TrustManager`：

		class MyTrustManager implements X509TrustManager {
	        X509Certificate cert;
	
	        MyTrustManager(X509Certificate cert) {
	            this.cert = cert;
	        }
	
	        @Override
	        // for server only
	        public void checkClientTrusted(X509Certificate[] chain, String authType)
	                throws CertificateException {
				// 我们在客户端只做服务器端证书校验。
	        }
	
	        @Override
	        // only trust the given certificate or certificate issued by it
	        public void checkServerTrusted(X509Certificate[] chain, String authType)
	                throws CertificateException {
	            // 确认服务器端证书和代码中 hard code 的 CRT 证书相同。
				if (chai[0].equals(this.cert)){
	                if(Utils.DEBUG){
	                    Log.i(Utils.DEBUG_TAG, "checkServerTrusted Certificate from server is valid!");
	                }
	                return;// found match
	            }
				throw new CertificateException("checkServerTrusted No trusted server cert found!");
	        }
	
	        @Override
	        public X509Certificate[] getAcceptedIssuers() {
	            return new X509Certificate[0];
	        }
	
	    }

	然后使用其定义 `SSLContext` 实例：

	    SSLContext sc = SSLContext.getInstance("TLS");
	    TrustManager tm = new MyTrustManager(readCert(certStr));
	    sc.init(null, new TrustManager[]{
	            tm
	    }, null);

	其中，`readCert` 方法的实现为：

		private static X509Certificate readCert(String cer) {
	        if (cer == null || cer.trim().isEmpty())
	            return null;
	        InputStream caInput = new ByteArrayInputStream(cer.getBytes());
	        X509Certificate cert = null;
	        try {
	            CertificateFactory cf = CertificateFactory.getInstance("X.509");
	            cert = (X509Certificate) cf.generateCertificate(caInput);
	        } catch (Exception e) {
	            if (Utils.DEBUG) {
	                e.printStackTrace();
	            }
	        } finally {
	            try {
	                if (caInput != null) {
	                    caInput.close();
	                }
	            } catch (Throwable ex) {
	            }
	        }
	        return cert;
	    }

	最后使用 `HttpsURLConnection` 进行网络通信：

	    URL url = new URL("https://www.example.com/");   
		HttpsURLConnection urlConnection = (HttpsURLConnection)url.openConnection(); 
	    conn.setSSLSocketFactory(sc.getSocketFactory());
	    // By default, this implementation of HttpURLConnection requests that servers use gzip compression
	    // and it automatically decompresses the data for callers of getInputStream().
	    // The Content-Encoding and Content-Length response headers are cleared in this case.
	    // Gzip compression can be disabled by setting the acceptable encodings in the request header:
	    // http://developer.android.com/reference/java/net/HttpURLConnection.html
	    conn.setRequestProperty("Accept-Encoding", "");
		StringBuffer response = new StringBuffer();
	    OutputStream os = null;
	    BufferedReader rd = null;
	    InputStream is = null;
	    int statusCode = -1;
	    try {
	        conn.setRequestMethod("POST");
	        os = conn.getOutputStream();
	        os.write(payload);
	        os.close();
	
	        // Get Response
	        statusCode = conn.getResponseCode();
	        InputStream is;
	        if(statusCode > HttpURLConnection.HTTP_BAD_REQUEST){
	            is = conn.getErrorStream();
	        }else{
	            is = conn.getInputStream();
	        }
	        rd = new BufferedReader(new InputStreamReader(is));
	        String line;
	        while ((line = rd.readLine()) != null) {
	            response.append(line);
	            response.append('\n');
	        }
	    } catch (Throwable e) {
	        if (Utils.DEBUG) {
	            e.printStackTrace();
	        }
	    } finally {
	        try {
	            if (is != null) {
	                is.close();
	            }
	        } catch (Throwable ex) {
	        }
	        try {
	            if (os != null) {
	                os.close();
	            }
	        } catch (Throwable ex) {
	        }
	        try {
	            if (rd != null) {
	                rd.close();
	            }
	        } catch (Throwable ex) {
	        }
	        try {
	            if (conn != null) {
	                conn.disconnect();
	            }
	        } catch (Throwable ex) {
	        }
	    }

	需要注意的是：

	1. conn.setRequestProperty("Accept-Encoding", ""); 如果自己实现了压缩算法，不希望系统自动添加 gzip 压缩的话必须添加。详情请参考上面代码注释中的文档。  
	2. conn.setSSLSocketFactory(sc.getSocketFactory()); 这样设置后，自定义的 `SSLContext` 将被用于本次 Https 通信，不会影响应用中其他可能的 Https 通信。 

- 使用经过 CA 认证的证书 

	如果服务器端正确配置了使用 CA 认证后的证书，Android 客户端应用程序可以直接使用 `HTTPSURLConnection` 访问：

	    URL url = new URL("https://www.example.com/");   
		HttpsURLConnection urlConnection = (HttpsURLConnection)url.openConnection();   
		InputStream in = urlConnection.getInputStream();

	如果需要做更严格的验证，也可以这样自定义 `TrustManager`：

		class MyTrustManager implements X509TrustManager {
	        X509Certificate cert;
	
	        MyTrustManager(X509Certificate cert) {
	            this.cert = cert;
	        }
	
	        @Override
	        // for server only
	        public void checkClientTrusted(X509Certificate[] chain, String authType)
	                throws CertificateException {
				// 我们在客户端只做服务器端证书校验。
	        }
	
	        @Override
	        // only trust the given certificate or certificate issued by it
	        public void checkServerTrusted(X509Certificate[] chain, String authType)
	                throws CertificateException {
	            // 确认服务器端证书的 Intermediate CRT 和代码中 hard code 的 CRT 证书主体一致。
		        if (!chain[0].getIssuerDN().equals(certificate.getSubjectDN())) {
		            throw new CertificateException("Parent certificate of server was different than expected signing certificate");
		        }
		
		        try {
		            // 确认服务器端证书被代码中 hard code 的 Intermediate CRT 证书的公钥签名。
		            chain[0].verify(certificate.getPublicKey());
		
		            // 确认服务器端证书没有过期
		            chain[0].checkValidity();
		        } catch (Exception e) {
		            throw new CertificateException("Parent certificate of server was different than expected signing certificate");
		        }
	        }
	
	        @Override
	        public X509Certificate[] getAcceptedIssuers() {
	            return new X509Certificate[0];
	        }
	
	    }

	思路就是在代码中 hard code CA 认证后交付的 Intermediate CRT 文件作为客户端证书对服务器端证书进行校验。根据 [开发安全的Android应用](http://www.devtf.cn/?p=1158) 提出的观点，可以避免最终用户证书有效期可能比较短的问题。

	之后使用 `HttpsURLConnection` 的方式和使用自签名证书时相同就不特别说明了。


### 更多资源：

- [How to create a self-signed SSL Certificate](http://www.akadia.com/services/ssh_test_certificate.html)
- [Android证书信任问题](http://drops.wooyun.org/tips/3296)
- [Official Doc of checkClientTrusted and checkServerTrusted](http://docs.oracle.com/javase/7/docs/api/javax/net/ssl/X509TrustManager.html)
- [SSL证书是三级证书好还是二级证书好？有什么不同？](http://www.wosign.com/new/FAQ/Level3_cert_advantage.htm)


[1]: http://en.wikipedia.org/wiki/HTTPS
[2]: http://en.wikipedia.org/wiki/Man-in-the-middle_attack
[3]: https://grahamedgecombe.com/work/diss.pdf
[4]: http://en.wikipedia.org/wiki/Certificate_authority
[5]: http://stackoverflow.com/questions/21845904/trust-only-particular-certificate-issued-by-ca-android



