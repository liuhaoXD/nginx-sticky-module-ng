# Nginx Sticky Module 


版本更新日志见 Changelog.txt 

# 相关描述

该 Nginx module 工作在通过为请求增加 sticky cookie 的方式，实现在反向代理时，始终
将指定请求转发到同一台 upstream 服务器。

在与后端服务器交互时，客户端（浏览器）有时需要始终与同一台后端服务器建立连接（比如实现
会话保持）。

由于很多来自于不同浏览器的请求可能来源于同一个IP（它们使用了一些代理技术），后端服务器
的负载很容易不均衡，所以通过IP地址（ip_hash upstream module）实现会话保持并不是一个
优秀的方案。

借由cookie，我们可以将每个浏览器的请求唯一到分配指定的 upstream 服务器

当 sticky module 由于一些原因而无法正常工作时，选择 upstream 的方案会重新切换回经典的
轮询策略，或者直接返回 "Bad Gateway" 响应（取决于 no_fallback 标识是否启用）。

在浏览器不支持 cookie 的情况下，本 module 将无法正常工作。

> Sticky module 的设计基于尽力（best effort）算法，它并不能解决任何安全性问题，它的
> 设计和实现初衷，只是保证一般用户的请求能一直被转发到相同的后端服务器，仅此而已！

# 安装

你需要使用源码重新编译 Nginx，同时按照指南将本module包含进去。
一般来说，你可以使用并修改下面的指令来修改Nginx编译方式（请按照实际环境修改module路径）。

```bash
./configure ... --add-module=/absolute/path/to/nginx-sticky-module-ng
make
make install
```

# 使用

```nginx
upstream {
  sticky;
  server 127.0.0.1:9000;
  server 127.0.0.1:9001;
  server 127.0.0.1:9002;
}

sticky [name=route] [domain=.foo.bar] [path=/] [expires=1h] [hash=index|md5|sha1] [no_fallback] [secure] [httponly];
```

- name:    用于定位固定 upstream 服务器的 cookie 名字； 
  默认值: route。

- domain:  cookie 生效的域名；
  默认值: 空。浏览器会自行设定。

- path:    cookie 生效的uri路径；
  默认值: /。

- expires: cookie 的有效时间间隔；
  默认值: 空。本次会话有效。
  限制值: 时间间隔必须大于1秒

- hash:    编码 upstream 服务器的 hash 算法。不能与下面提到的 hmac 选项同时使用；
  默认值: md5。

    - md5|sha1: 广泛使用的 hash 摘要算法。
    - index:    这个并不是 hash，而是内存中 upstream 列表内，对应 upstream 服务器
    的索引，相比之下这种方式开销更小，查询也更快。注意：这种方式不能保证upstream列表
    匹配的一致性，一旦reload时upstream服务器列表发生了改变，原有的index很可能会映射到
    与之前不同的服务器上，请**谨慎使用**。
 
- hmac:    编码 upstream 服务器的 HMAC hash 算法。它与上面提到的 hash 选项类似，不同
  的是它额外需要一个 hmac_key 来保证散列过程的安全性，不能与 hash 选项同时使用；
  默认值: 空，见 hash。
    - md5|sha1: 广泛使用的 hash 摘要算法。

- hmac_key: 启用 hmac 散列时指定的 key。指定了hmac选项后，该项也是必填的；
  默认值: 空。

- no_fallback: 启用了该标识后，如果请求携带着 sticky cookie，同时对应的后端服务器不可用时，
  Nginx将返回502响应（Bad Gateway 或者 Proxy Error）响应。

- secure    启用 secure cookies；仅通过 https 进行传输。

- httponly  启用以防止cookie通过js被泄露


# 实现机制

- 见 docs/sticky.{vsd,pdf}	

# 已知问题和注意事项

- 不同 location 的多个 upstream 配置同时启用 sticky-module 时，应当分别为其指定不同的 path/route选项，详见
  https://bitbucket.org/nginx-goodies/nginx-sticky-module-ng/issue/7/leaving-cookie-path-empty-in-module

- sticky module 在 upstream 配置块的 server 指令启用了 backup 选项时，无法正常工作。
- sticky module 自1.2.3版本以后，可能会与 nginx_http_upstream_check_module 正常工作。
- sticky module 会要求 nginx 启用 SSL 支持 (在使用了 "secure" 选项时)

# TODO

见 Todo.md
  
# Authors & Credits

- Jerome Loyet, initial module
- Markus Linnala, httponly/secure-cookies-patch
- Peter Bowey, Nginx 1.5.8 API-Change 
- Michael Chernyak for Max-Age-Patch 
- anybody who suggested a patch, created an issue on bitbucket or helped improving this module 



# 版权和许可

    This module is licenced under the BSD license.
  
    Copyright (C) 2010 Jerome Loyet (jerome at loyet dot net)
    Copyright (C) 2014 Markus Manzke (goodman at nginx-goodies dot com)

  
    Redistribution and use in source and binary forms, with or without
    modification, are permitted provided that the following conditions
    are met:
  
    1. Redistributions of source code must retain the above copyright
    notice, this list of conditions and the following disclaimer.
  
    2. Redistributions in binary form must reproduce the above copyright
    notice, this list of conditions and the following disclaimer in the
    documentation and/or other materials provided with the distribution.
  
    THIS SOFTWARE IS PROVIDED BY AUTHOR AND CONTRIBUTORS ``AS IS'' AND
    ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
    IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
    ARE DISCLAIMED.  IN NO EVENT SHALL AUTHOR OR CONTRIBUTORS BE LIABLE
    FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
    DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS
    OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)
    HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
    LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY
    OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF
    SUCH DAMAGE.
  
