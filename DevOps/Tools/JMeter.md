[Apache JMeter](https://jmeter.apache.org/) 是一款开源的性能测试工具，最初由 Apache 软件基金会的 Stefano Mazzocchi 开发，主要用于测试 Web 应用程序或 FTP 应用程序的性能。如今，JMeter 已扩展支持多种协议，包括 HTTP、HTTPS、JDBC、LDAP、SOAP、JMS、FTP 等，适用于负载测试、性能测试、功能测试和回归测试。

#### 主要特点

1. **开源免费**：JMeter 是完全免费的，允许开发者自由使用和修改源代码。
2. **跨平台**：基于 Java 开发，可在 Windows、Linux、Mac 等系统上运行。
3. **多协议支持**：支持 HTTP、HTTPS、FTP、JDBC、SOAP、JMS 等多种协议。
4. **可视化测试结果**：提供多种监听器（如查看结果树、汇总报告、聚合报告）来展示测试数据。
5. **分布式测试**：支持多台机器协同测试，提高负载能力。
6. **参数化与动态测试**：支持 CSV 数据参数化、变量替换等功能，使测试更灵活。


### 安装使用

1. **安装** ：
	- **下载 JMeter**：从 [Apache JMeter 官网](https://jmeter.apache.org/) 下载最新版本。
	- **安装 Java**：JMeter 需要 Java 环境（JDK 8 或更高版本）。
	- **配置环境变量**（可选）：将 JMeter 的 `bin` 目录添加到系统 `PATH` 变量。
	- **启动 JMeter**：
	    - Windows：运行 `bin/jmeter.bat`
	    - Linux/Mac：运行 `bin/jmeter.sh`。

2. 