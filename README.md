# sing-box DNS Guide: Mitigate DNS Pollution with Fake IP and Route-DNS

[English](#english) | [简体中文](#简体中文) | [Русский](#русский)

---

## <a name="english"></a>English

### The Anatomy of DNS Pollution and Leakage
For users behind restrictive firewalls (such as the GFW), **DNS Pollution (DNS Hijacking/Poisoning)** is the primary mechanism of censorship. When your device queries a blocked domain (e.g., `google.com`), the firewall intercepts the request and injects a spoofed IP response before the real DNS server can reply.

Additionally, **DNS Leaks** occur when your proxy client routes traffic correctly but continues to resolve domain names through your local ISP's DNS servers. This exposes your entire browsing history to your ISP.

---

### Mitigating DNS Pollution in sing-box
**sing-box** provides an exceptionally advanced DNS routing engine to combat these vulnerabilities, primarily through two methods: **Route-DNS** and **Fake IP**.

#### 1. Route-DNS (Split DNS Resolution)
This method separates DNS queries based on the destination domain:
- **Domestic Domains**: Resolved via local, high-speed DNS servers (e.g., `223.5.5.5` in China) using standard UDP/DoH.
- **Global Domains**: Routed through the encrypted proxy tunnel and resolved via secure upstream DNS servers (e.g., `8.8.8.8` or `1.1.1.1`) using DoH (DNS over HTTPS) or DoT.

#### 2. The Fake IP Mechanism
Instead of performing real DNS resolution locally, the Fake IP engine interceptor immediately returns a mock IP address (typically from the `198.18.0.0/15` subnet) to the client application (like your browser). 
1. The browser initiates a connection to the Fake IP.
2. sing-box captures this connection, matches the Fake IP back to the original domain name, and forwards the packet to the remote proxy server.
3. The remote proxy server resolves the domain name in a clean, censorship-free network environment.

This eliminates local DNS pollution entirely, secures your privacy, and decreases connection setup times.

---

### sing-box Advanced DNS Configuration Example
Here is a structured configuration snippet for sing-box's DNS block:

```json
{
  "dns": {
    "servers": [
      {
        "tag": "dns_proxy",
        "address": "https://8.8.8.8/dns-query",
        "detour": "proxy_outbound"
      },
      {
        "tag": "dns_direct",
        "address": "223.5.5.5",
        "detour": "direct"
      },
      {
        "tag": "dns_fakeip",
        "address": "fakeip"
      }
    ],
    "rules": [
      {
        "outbound": "any",
        "server": "dns_direct"
      },
      {
        "query_type": [ "AAAA" ],
        "server": "dns_direct"
      },
      {
        "rule_set": "geosite-geolocation-cn",
        "server": "dns_direct"
      },
      {
        "query": [ "google.com", "github.com" ],
        "server": "dns_fakeip"
      }
    ],
    "fakeip": {
      "inet4_range": "198.18.0.0/15"
    }
  }
}
```

---

### The Easy Alternative: Kite VPN
Configuring split DNS, handling Fake IP mappings, and avoiding IPv6 leaks in sing-box requires deep networking expertise. One syntax mistake can break your internet connection or expose your traffic.

**Kite VPN** implements a zero-configuration, proprietary DNS shield that:
- **Blocks DNS Hijacking**: Dynamically intercepts and encrypts all DNS traffic.
- **Pre-configured Fake IP & Routing**: Instantly separates domestic and global queries for maximum local speed and zero leaks.
- **Zero Configuration**: Ready to protect your privacy on iOS, Android, macOS, and Windows.

#### Secure Your Connection Instantly
Download Kite VPN for leak-free, high-speed browsing:
- **Official Website**: [KiteVPN](https://kitevip.vip?f=github)
- Start your free trial today.

---

## <a name="简体中文"></a>简体中文

### DNS 污染与 DNS 泄露原理解析
在受到严格审查的网络环境（如 GFW 封锁）下，**DNS 污染（或称 DNS 劫持/投毒）**是最常见的封锁手段。当您的设备尝试解析被屏蔽的域名（例如 `google.com`）时，防火墙会拦截该请求，并抢先返回一个虚假的错误 IP，导致您无法正常访问。

此外，当代理客户端成功路由了网络流量，但域名解析请求仍通过本地运营商（ISP）的 DNS 服务发送时，就会发生 **DNS 泄露**。这会让您的全部上网轨迹对运营商完全公开。

---

### sing-box 防污染解决方案
**sing-box** 提供了目前最先进的 DNS 路由引擎，主要通过以下两种技术解决 DNS 安全问题：**分流 DNS（Route-DNS）**与 **Fake IP（虚拟 IP）**。

#### 1. 分流 DNS（基于规则的解析）
该方案根据目标域名分类解析：
- **国内域名**：分配给本地高速 DNS 服务器（如阿里的 `223.5.5.5`），直接使用 UDP 或 DoH 解析以获取最低延迟。
- **海外域名**：通过加密代理隧道，路由至远端安全的 DNS 服务器（如 `8.8.8.8` 或 `1.1.1.1`），并强制使用 DoH/DoT 加密查询。

#### 2. Fake IP（虚拟 IP 映射）机制
Fake IP 引擎不向系统返回真实的公网 IP，而是在收到解析请求时，立即返回一个保留网段（通常是 `198.18.0.0/15`）的虚拟 IP。
1. 浏览器收到虚拟 IP 后，立即向该 IP 发起 TCP 连接。
2. sing-box 拦截该连接，在内部映射表中找回其对应的原始域名，然后将数据包原封不动转发给海外代理服务器。
3. 真实的 DNS 解析在免受干扰的海外代理服务器端完成。

这种设计彻底规避了本地 DNS 污染，防止了任何形式的 DNS 泄露，并且大幅加快了连接建立速度。

---

### sing-box 进阶 DNS 配置范例
以下是一个防污染、支持 Fake IP 的 sing-box DNS 块配置示例：

```json
{
  "dns": {
    "servers": [
      {
        "tag": "dns_proxy",
        "address": "https://8.8.8.8/dns-query",
        "detour": "proxy_outbound"
      },
      {
        "tag": "dns_direct",
        "address": "223.5.5.5",
        "detour": "direct"
      },
      {
        "tag": "dns_fakeip",
        "address": "fakeip"
      }
    ],
    "rules": [
      {
        "outbound": "any",
        "server": "dns_direct"
      },
      {
        "query_type": [ "AAAA" ],
        "server": "dns_direct"
      },
      {
        "rule_set": "geosite-geolocation-cn",
        "server": "dns_direct"
      },
      {
        "query": [ "google.com", "github.com" ],
        "server": "dns_fakeip"
      }
    ],
    "fakeip": {
      "inet4_range": "198.18.0.0/15"
    }
  }
}
```

---

### 免折腾方案：Kite VPN 智能安全 DNS
手动配置 sing-box 分流 DNS 和 Fake IP 规则非常繁琐，稍有不慎就会导致网络瘫痪或 DNS 泄露。

**Kite VPN** 内置了经过专业优化的**智能 DNS 防护盾**：
- **封杀 DNS 劫持**：底层强制对所有解析流量进行加密重定向。
- **开箱即用的 Fake IP 与分流**：智能识别国内与国际解析请求，国内直连不降速，海外防污染解锁。
- **全平台一键连接**：无需编写任何代码，在 iOS、Android、macOS 和 Windows 上皆可一键享受纯净网络。

#### 立即守护网络隐私
下载 Kite VPN，体验无污染的顺畅网络：
- **官方网站**：[KiteVPN](https://kitevip.vip?f=github)
- 点击开启免费试用。

---

## <a name="русский"></a>Русский

### Что такое загрязнение DNS и утечка DNS?
В сетях с жесткой цензурой (таких как GFW в Китае) **загрязнение DNS (DNS Poisoning/Spoofing)** является основным методом блокировки. Когда ваше устройство запрашивает IP-адрес заблокированного ресурса (например, `google.com`), брандмауэр перехватывает этот запрос и отправляет поддельный IP-адрес до того, как ответит реальный DNS-сервер.

Кроме того, **утечки DNS (DNS Leaks)** происходят, когда прокси-клиент шифрует ваш веб-трафик, но сами запросы имен доменов отправляются в открытом виде через локальные DNS-серверы вашего провайдера, раскрывая историю посещений.

---

### Борьба с загрязнением DNS в sing-box
**sing-box** предоставляет мощный инструмент маршрутизации DNS-запросов, основанный на двух технологиях: **Route-DNS (раздельный DNS)** и **Fake IP (виртуальный IP)**.

#### 1. Route-DNS (Разделение DNS-запросов)
DNS-запросы классифицируются по правилам:
- **Локальные домены**: Направляются на быстрые внутренние серверы (например, `223.5.5.5` или DNS вашего провайдера) напрямую.
- **Зарубежные домены**: Направляются через шифрованный прокси-туннель на защищенные серверы (Google `8.8.8.8` или Cloudflare `1.1.1.1`) с использованием протоколов DoH/DoT.

#### 2. Механизм Fake IP
Вместо того чтобы ожидать реального ответа от DNS-сервера, модуль Fake IP в sing-box мгновенно возвращает системе фиктивный IP-адрес из диапазона `198.18.0.0/15`.
1. Браузер сразу же открывает TCP-соединение на этот Fake IP.
2. sing-box перехватывает пакет, находит в своей таблице оригинальное имя домена и отправляет его на удаленный прокси-сервер.
3. Удаленный сервер выполняет чистое DNS-разрешение в свободном сегменте сети.

Это полностью предотвращает локальное загрязнение DNS, защищает конфиденциальность и ускоряет загрузку сайтов.

---

### Пример конфигурации DNS в sing-box
Пример настройки блока DNS с поддержкой Fake IP:

```json
{
  "dns": {
    "servers": [
      {
        "tag": "dns_proxy",
        "address": "https://8.8.8.8/dns-query",
        "detour": "proxy_outbound"
      },
      {
        "tag": "dns_direct",
        "address": "223.5.5.5",
        "detour": "direct"
      },
      {
        "tag": "dns_fakeip",
        "address": "fakeip"
      }
    ],
    "rules": [
      {
        "outbound": "any",
        "server": "dns_direct"
      },
      {
        "query_type": [ "AAAA" ],
        "server": "dns_direct"
      },
      {
        "rule_set": "geosite-geolocation-cn",
        "server": "dns_direct"
      },
      {
        "query": [ "google.com", "github.com" ],
        "server": "dns_fakeip"
      }
    ],
    "fakeip": {
      "inet4_range": "198.18.0.0/15"
    }
  }
}
```

---

### Простое решение: Умный DNS в Kite VPN
Ручная настройка раздельного DNS и Fake IP в sing-box требует глубоких знаний. Ошибка в коде JSON может лишить вас интернета или привести к скрытым утечкам данных.

**Kite VPN** предлагает готовую технологию защиты DNS:
- **Защита от подмены DNS**: Все запросы шифруются и безопасно передаются внутри сети.
- **Встроенный Fake IP**: Полное разделение трафика "из коробки" для максимальной скорости и безопасности.
- **Поддержка любых устройств**: Нативные приложения для iOS, Android, macOS и Windows.

#### Попробуйте безопасный интернет прямо сейчас
Скачайте Kite VPN и забудьте о проблемах с DNS:
- **Наш сайт**: [KiteVPN](https://kitevip.vip?f=github)
- Зарегистрируйтесь для получения бесплатного теста.
