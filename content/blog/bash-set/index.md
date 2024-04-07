---
title: "Modifying shell behavior set -xeuo pipefail"
description: "Elevate your bash scripts"
lead: "Bash scripts are an essential tool for automating tasks and streamlining workflows. However, to truly unleash their potential, it's crucial to understand and utilize some key commands that can significantly enhance the reliability, security, and debuggability of your scripts."
summary: "Bash scripts are an essential tool for automating tasks and streamlining workflows. However, to truly unleash their potential, it's crucial to understand and utilize some key commands that can significantly enhance the reliability, security, and debuggability of your scripts."
date: 2024-04-07T16:27:22+02:00
lastmod: 2024-04-07T16:27:22+02:00
draft: true 
weight: 50
categories: []
tags: ["bash","coding","containers","Dockerfile"]
contributors: ["Sharjeel Aziz"]
pinned: false
homepage: true 
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

In this post, we'll explore four such commands: `set -e`, `set -u`, `set -o pipefail`, and `set -x`, and see how they can help prevent real-world script failures. These commands really enhance scripts that are part of container images, most commonly, entry point scripts where increased control over how the script behaves is required.

**1. Exit on Error**
The `set -e` command is a game-changer when it comes to error handling in Bash scripts. By including this command at the beginning of your script, you instruct Bash to immediately exit if any command returns a non-zero exit status (indicating an error). This helps catch and handle errors early, preventing them from cascading and causing more problems later in the script. 

Consider a script that performs a series of file operations:

```bash
# script without set -e
rm important_file
cp backup_file important_file
```

If the `rm` command fails (e.g., if `important_file` doesn't exist), the script will continue executing, potentially leading to unexpected behavior or data loss. With `set -e`, the script would exit immediately upon the `rm` command failing, preventing further issues.

{{< img src="set_e.png" alt="terminal recording showing bash scripts results"  >}}

As the Bash man page states, "Exit immediately if a pipeline, which may consist of a single simple command, a list, or a compound command returns a non-zero status" [GNU Bash Manual, 4.3.1 The Set-Builtin](https://www.gnu.org/software/bash/manual/bash.html#The-Set-Builtin)


**2. Treat Unset Variables as Errors**
Another powerful command is `set -u`, which tells Bash to treat any reference to an unset variable as an error. This feature is invaluable for catching typos, forgotten variable initialization, and other subtle bugs that can be difficult to diagnose. 

Imagine a script that uses a configuration file:

```bash
# script without set -u
config_file="path/to/config"
rm "$config_filee"
```

Here, `$config_filee` is a typo and refers to an unset variable. Without `set -u`, the script would silently continue, potentially deleting the wrong file or causing other unintended consequences. With `set -u`, the script would halt immediately, alerting you to the typo.

The Bash man page confirms this behavior: "Treat unset variables and parameters other than the special parameters '@' or '*' as an error when performing parameter expansion" [GNU Bash Manual, 4.3.1 The Set-Builtin](https://www.gnu.org/software/bash/manual/bash.html#The-Set-Builtin)

**3. Handle Pipeline Failures**
Consider a script that processes a file, extracts specific lines, and then counts the number of occurrences of a particular pattern.  The script is working with an existing file and a file that does not exist:

```bash
#!/bin/bash

# script without set -o pipefail

cat doesnotexist.txt | grep "error" | wc -l
echo "Processing complete for file that does not exist. Exit status: $?"

cat file.txt | grep "error" | wc -l
echo "Processing complete. Exit status: $?"
```

When we run the script, we get the following output:

```
scripts/set/logs on  main [✘!?] 
➜ ./process.sh 
cat: doesnotexist.txt: No such file or directory
0
Processing complete for file does not exist. Exit status: 0
2
Processing complete. Exit status: 0
```

Even though `cat` failed because the file doesn't exist, the pipeline's exit status is 0 because the last command (`wc -l`) succeeded.

Now, let's modify the script to use `set -o pipefail` and demonstrate how it affects the return status:

```bash
#!/bin/bash

# script with set -o pipefail
set -o pipefail

cat doesnotexist.txt | grep "error" | wc -l
echo "Processing complete for file does not exist. Exit status: $?"

# file exists
cat file.txt | grep "error" | wc -l
echo "Processing complete. Exit status: $?"
```

When we run this script, we get the following output:

```
scripts/set/logs on  main [✘!?] 
➜ ./process_pipefail.sh 
cat: doesnotexist.txt: No such file or directory
0
Processing complete for file does not exist. Exit status: 1
2
Processing complete. Exit status: 0
```

With `set -o pipefail`, the pipeline's exit status is now 1, reflecting the failure of the `cat` command. This behavior allows us to detect failures in any command within the pipeline.

This example demonstrates how `set -o pipefail` can help you catch errors in your pipelines and prevent your scripts from producing misleading results. Here is an interesting article article: [PIPEFAIL: How a missing shell option slowed Cloudflare down](https://blog.cloudflare.com/pipefail-how-a-missing-shell-option-slowed-cloudflare-down)

The Bash man page elaborates on this functionality: "If set, the return value of a pipeline is the value of the last (rightmost) command to exit with a non-zero status, or zero if all commands in the pipeline exit successfully" [GNU Bash Manual, 4.3.1 The Set-Builtin](https://www.gnu.org/software/bash/manual/bash.html#The-Set-Builtin).

**4. Enable Debugging Mode**
Debugging Bash scripts can be a challenging task, but `set -x` makes it much more manageable. This command enables a debugging mode that prints each command in your script to the console as it's executed, along with its arguments. This feature provides valuable insights into the flow and state of your script, making it easier to identify and troubleshoot issues.

Suppose you have a complex script with multiple conditionals and loops:

```bash
# Script without set -x
for file in "$@"; do
    if [ -f "$file" ]; then
        process_file "$file"
    fi
done
```

If the script isn't behaving as expected, adding `set -x` at the beginning will print out each command as it's executed, helping you pinpoint where the issue lies.

The Bash man page describes this mode as follows: "Print a trace of simple commands,  and their arguments after they are expanded and before they are executed" [GNU Bash Manual, 4.3.1 The Set-Builtin](https://www.gnu.org/software/bash/manual/bash.html#The-Set-Builtin).

To get the most out of your Bash scripts, consider using these commands in combination. They are not mutually exclusive and can work together to create robust, secure, and easily debuggable scripts. By incorporating `set -e`, `set -u`, `set -o pipefail`, and `set -x` into your scripting practices, you'll be equipped with a powerful toolset for writing high-quality Bash scripts that can handle real-world challenges.

#### Further Reading:
 - [GNU Bash Manual 4.3.1 The Set Builtin](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html)
