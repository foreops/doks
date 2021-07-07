---
title: "Container Scanning with Trivy in Jenkins"
description: "Container security scanning in Jenkins CI with Trivy"
lead: "With unrelenting attacks from malicious hackers on business critical software and infrastructure, the \"Shift-left\" approach for security testing is gaining more momentum inside the enterprises. "
date: 2021-07-02T11:00:00-04:00
lastmod: 2021-07-02T11:00:00-04:00
draft: false
images: ["trivy-jenkins.png"]
weight: 50
contributors: ["Faheem Memon"]
tags: ["security","trivy","container","scanner"]
---

You can identify and address the security vulnerabilities earlier in the software development life-cycle if you integrate a scanner in the continuous integration (CI) pipelines. There are multiple open-source and commercial tools out there that can scan the applications and containers. When comparing these tools, we consider the following aspects:

1. Easier integration with the pipelines through plugins, scripts, or API calls.
2. Faster, ideally synchronous execution. It should not hold up the build pipeline.
3. Ability to easily ignore false positives, unfixable, and realized vulnerabilities.
4. Audit report access from the pipeline and roll-up at the organizational level.
5. Ability to run tests from developer machines easily

CI pipeline integration is often just one part of an enterprise security toolchain. Tools like AquaSecurity, Prisma, Snyk, and others provide additional features such as periodic re-scanning, run-time monitoring, alerting, and a centralized dashboard for the security teams. In this blog post, however, we will look into [Trivy](https://github.com/aquasecurity/trivy). Trivy is an open-source scanner from the AquaSecurity team that can scan container images or filesystem paths for vulnerable operating system packages or application dependencies.

{{< img src="trivy-highlight.png" alt="Trivy: A simple and comprehensive vulernerability" caption="" class="wide" >}}

Let's review our scanner selection criteria with Trivy.

### Getting Started

Let's download `trivy` CLI and test it on our local machine:

```bash
$ curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin v0.18.3
aquasecurity/trivy info checking GitHub for tag 'v0.18.3'
aquasecurity/trivy info found version: 0.18.3 for v0.18.3/macOS/64bit
aquasecurity/trivy info installed /usr/local/bin/trivy
```

There are several binaries available for `*-nix` based systems. If you can't find the right binary, you have an option to use the [`aquasec/trivy`](https://hub.docker.com/r/aquasec/trivy) container instead. First, let's take a quick look at the CLI.

```bash
$ trivy --help
NAME:
   trivy - A simple and comprehensive vulnerability scanner for containers

USAGE:
   trivy [global options] command [command options] target

VERSION:
   0.18.3

COMMANDS:
   image, i          scan an image
   filesystem, fs    scan local filesystem
   repository, repo  scan remote repository
   client, c         client mode
   server, s         server mode
   plugin, p         manage plugins
   help, h           Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --quiet, -q        suppress progress bar and log output (default: false) [$TRIVY_QUIET]
   --debug, -d        debug mode (default: false) [$TRIVY_DEBUG]
   --cache-dir value  cache directory (default: "/Users/faheem/Library/Caches/trivy") [$TRIVY_CACHE_DIR]
   --help, -h         show help (default: false)
   --version, -v      print the version (default: false)
```

When you run a scan, trivy will automatically download the vulnerabilities database and cache it for 12 hours. The database can also be downloaded separately if required.

Let's take `trivy` for a spin and see it in action.

```bash
trivy image node:8-alpine
```

{{< img src="trivy-scan-cli.png" alt="trivy image node:8-alpine" caption="Trivy Scan Report" class="wide" >}}

`node:11-alpine` seems to have several security vulnerabilities, including a few CRITICAL and HIGH. However, all of them are identified in the underlined `Alpine 3.9.4` operating system since we have not deployed any custom applications inside the container. Trivy can differentiate between OS and application dependency base vulnerabilities.

By default, Trivy generates a tabular output on the CLI, but it supports templating, and you can generate reports in multiple formats including HTML file type. Let's create an HTML report for the last run.

```bash
wget https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl
trivy image --format template --template @./html.tpl -o report.html node:11-alpine
```

{{< img src="trivy-scan-html.png" alt="trivy image node:8-alpine" caption="Trivy Scan HTML Report" class="wide" >}}

### Jenkins Integration

Let's add a scanning feature in our `Jenkinsfile`:

```jenkinsfile
...
        stage('Scan') {
            steps {
                // Install trivy
                sh 'curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin v0.18.3'
                sh 'curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl > html.tpl'

                // Scan all vuln levels
                sh 'mkdir -p reports'
                sh 'trivy filesystem --ignore-unfixed --vuln-type os,library --format template --template "@html.tpl" -o reports/nodjs-scan.html ./nodejs'
                publishHTML target : [
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports',
                    reportFiles: 'nodjs-scan.html',
                    reportName: 'Trivy Scan',
                    reportTitles: 'Trivy Scan'
                ]

                // Scan again and fail on CRITICAL vulns
                sh 'trivy filesystem --ignore-unfixed --vuln-type os,library --exit-code 1 --severity CRITICAL ./nodejs'

            }
...
```

{{< img src="trivy-scan-jenkins.png" alt="trivy image node:8-alpine" caption="Trivy Scan HTML Report" class="wide" >}}

The script downloads and runs the scanner on a [nodejs](https://github.com/faheem556/funfact) application path. It runs the scan twice, the second time from the previous cache, and fails the build if it discovers are any CRITICAL vulnerabilities. Finally, the report to the pipeline using [HTML Publisher](https://plugins.jenkins.io/htmlpublisher/) plugin.

You can manage false positives through [`.trivyignore`](https://aquasecurity.github.io/trivy/v0.18.3/examples/filter/#by-vulnerability-ids) file. So you can check in your `.trivyignore` file and the scan step in Jenkins pipeline will pick it up automatically.

Trivy is a fast, lightweight, and user-friendly scanner that can easily integrate in any environment.

Further Reading:

1. [https://en.wikipedia.org/wiki/Shift-left_testing](https://en.wikipedia.org/wiki/Shift-left_testing)
2. [https://aquasecurity.github.io/trivy/v0.18.3/](https://aquasecurity.github.io/trivy/v0.18.3/)
3. [https://github.com/aquasecurity/trivy](https://github.com/aquasecurity/trivy)
4. [https://aquasecurity.github.io/trivy/v0.18.3/comparison/](https://aquasecurity.github.io/trivy/v0.18.3/comparison/)
5. [https://aquasecurity.github.io/trivy/v0.18.3/air-gap/](https://aquasecurity.github.io/trivy/v0.18.3/air-gap/)
6. [https://hub.docker.com/r/aquasec/trivy](https://hub.docker.com/r/aquasec/trivy)
