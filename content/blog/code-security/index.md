---
title: "Securing your Code for Production"
description: "When designing cloud-native security, you should start with the assumption that all systems can be compromised. The 4Cs (Cloud, Cluster, Container, Code) of cloud-native security adopt the defense in depth approach and divides it into four layers."
lead: "When designing cloud-native security, you should start with the assumption that all systems can be compromised. The 4Cs (Cloud, Cluster, Container, Code) of cloud-native security adopt the defense in depth approach and divides it into four layers."
date: 2021-07-08T11:38:38-04:00
lastmod: 2021-07-08T11:38:38-04:00
draft: false
weight: 50
images: ["code-security.jpg"]
contributors: ["Sharjeel Aziz"]
tags: ["security","4cs","secure","coding","cloud"]
---

{{< img src="code-security.png" alt="Code Security"  >}}

Defense in depth is a concept where you implement multiple overlapping layers of security to provide protection even if one layer is compromised. Traditionally security was implemented at endpoints, for instance, a firewall controlling network traffic and email filtering. A defense in depth approach would filter emails at the firewall and through anti-virus software on the desktops. Why install the desktop software on desktops when the firewall is filtering emails? Because when you plug an infected USB drive in a desktop, the virus can spread from an infected USB drive.  Firewall and anti-virus on desktops is an example of defense in depth.

Application code is one of the primary attack surfaces over which you have the most control.  

#### Transport Layer Security

If your application needs to communicate over TCP, encrypt your data in transit using TLS. TLS prevents a common threat known as a "man-in-the-middle" attack. Without TLS, communication over TCP is vulnerable to this attack, and a malicious actor can decrypt the communication. After establishing a TCP connection with another application, you can exchange data, but you should perform a TLS handshake immediately to secure further communication. After the handshake is complete and keys exchanged, information may be encrypted and decrypted between applications.  The symmetric cryptography provided by TLS will secure the remainder of communication and encrypt all data in transit.

It is also a good idea to encrypt traffic between services. Mutual TLS  (mTLS) performs a two-sided verification of the communication between two certificate holding services. When using TLS, only the server proves its identity to the client, and it is acceptable for external, public-facing websites. Mutual TLS (mTLS) authentication ensures that traffic is secure and trusted in both directions between service to service.

#### Secure Code Review

You cannot secure your code if you do not review it. Implementing manual code review policies and deploying automated code reviews can identify security vulnerabilities in the code. Here is a list of possible focus areas for code reviews:

* Authentication
* Authorization
* Session management
* Data validation
* Error handling
* Logging
* Encryption

[OWASP Top Ten](https://owasp.org/www-project-top-ten/) and [Common Weakness Enumeration (CWE™)](http://cwe.mitre.org/data/definitions/699.html) are also good references to identify focus areas.

#### Static Code Analysis

These tools do not offer automated fixes but help detect and troubleshoot issues before they become real issues. [Source code analysis](https://owasp.org/www-community/Source_Code_Analysis_Tools) tools, also referred to as Static Application Security Testing (SAST) tools, are designed to analyze source code or compiled code versions to help find security flaws.

#### Dynamic Code Analysis

Dynamic Application Security Testing (DAST) tools analyze running applications. They should be part of the CI/CD pipeline.  DAST tools can identify errors and vulnerabilities that only appear when running in a realistic production environment. OWASP [Zed Attack Proxy (ZAP)](https://www.zaproxy.org/) is a widely used DAST tool.

#### Software Composition Analysis (SCA)

A typical application has direct or transitive dependencies, including open source projects. Direct dependencies are libraries used by your application, and transitive dependencies are the libraries your dependencies are using. SCA tools identify all open source components used in an application and any know vulnerabilities in them. It is best to automate this process to reduce bugs and to keep up to date.  OWASP [Dependency-Check](https://owasp.org/www-project-dependency-check/) is an SCA tool that attempts to detect publicly disclosed vulnerabilities within a project's dependencies. [Synk Open Source](https://snyk.io/product/open-source-security-management/) is another tool in this domain.

When securing your applications, implement security across multiple layers, such as cloud, clusters, containers, and code, assuming that all systems can be compromised. This blog covers just the code security layer, and we will cover the rest in future posts.
