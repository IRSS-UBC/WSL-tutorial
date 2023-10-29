_Disclaimer: This guide is based on the official Microsoft documentation and personal experience._

# How to set up Windows Subsystem for Linux on Windows 10 or later


This guide is meant to provide a quick introduction on what Windows Subsystem for Linux (WSL) is and how to set it up in a project workflow.

For further reading on WSL, please refer to the [official documentation](https://learn.microsoft.com/en-us/windows/wsl/).


## 🆕 What is WSL?


The latest WLS version 2 (WSL2) is an architecture that powers a full Linux kernel with full Linux capabilities that is meant to run inside a Windows operative system by taking advantage of the physical hardware installed on the local machine. WSL2 offers the classic CLI and further GUI support for many applications that can be run from the Windows Start menu.

Although WLS2 offers a full Linux kernel, it has poor across-OS performance. If your project scripts are located inside the WSL, but your data is stored in an external driver with a Windows filesystem, expect poor speed performance and possible communication errors across OS 🤕. For instance, for those relying on the `lidR` package, unless your data is stored inside WSL2, expect 🔥💣!

If your data must be stored in a Windows drive, consider downgrading to WLS1, but expect limited linux capabilities as it does not provide a full linux kernel.


## ❓ If I wanted to, how do I start?


### Set up


WLS comes with Windows, so you already have it installed. On top of that, remember that installation, updates, and reboots are managed via _Powershell_. 

Here some primary steps to get you started with Ubuntu 22.04, [here](https://learn.microsoft.com/en-us/windows/wsl/install) for the full documentation:

```sh
# List available distributions available on WLS
wsl --list --online

# Install distribution
# e.g. Ubuntu-22.04
wsl --install -d Ubuntu-22.04
```

Follow the instructions on screen and create username and password to access elevated privileges via the `sudo` command.

To check that your distribution had been installed with a WLS version 2, run `wsl -l -v`. If you want to downgrade/upgrade to another version, run `wsl --set-default-version <Version#>` and replace `<Version#>` with either `1` or `2`, as of today at least.


### Update and Reboot


To update to the current security and systems status, WLS can be updated via _Powershell_ with `wsl --update`.

If you are running intensive processing and your WSL gets stuck, idles, or consumes an unexpected amount of resources (CPU, RAM), reboot it by using `wsl --shutdown` inside _Powershell_. The virtual machine will automatically start then next time you call it from the terminal.


### 🛰️ Advanced configuration


For further readings on advanced WSL configuration, refer to the [official documentation](https://learn.microsoft.com/en-us/windows/wsl/wsl-config).

The WSL2 configuration is done on two levels: global configuration, which affects all installed distributions, and local configuration, which affects only the targeted distribution:

 ✔️ The first configuration is defined by the `.wslconfig` file, which can be found inside the `%UserProfile%` folder (`%UserProfile%\.wslconfig`). You will likely have to make the file yourself as it is not initialized at distribution installation;

 ✔️ The second configuration is distro-specific, hence is located or initialized in `/etc/wsl.conf`, where `/etc` represents the filesystem directory of your local distribution. This can be accessed from the terminal or via Windows explorer.


#### Global configuration

By default, WSL uses 50% of the available physical RAM, in addition to a swap partition that works similarly to the [Windows page file](https://learn.microsoft.com/en-us/troubleshoot/windows-client/performance/introduction-to-the-page-file). To customize this parameter, along with the max number of processor available to the WLS, the following lines must be updated inside `.wslconfig`:

```sh
# Settings apply across all distros running on WSL2
[wsl2]

# Limits memory to use no more than 64GB
memory=64GB

# Sets amount of swap storage space to 16GB
swap=16GB

# Limits CPU processor usage (do not include if you want to use them all)
# processors=24

# Enable/Disable GUI interface
guiApplications=false
```

There are many other options available for customization, but these are the basic ones we needed to tweak for a better experience overall, as far as a virtual machine goes 💣.


#### Local configuration

Unless specific requirements must be met, you do not necessarily need to tweak the local configuration. However, if you are planning on using `R` inside WSL, you may want to consider enabling `systemd`. This is a basic suite of tools that allows a Linux system to run smoothly and operate as PID 1 (i.e. process ID 1, in charge of starting and shutting down the system). The downside of enabling `systemd` is a higher CPU and RAM consumption while WSL is running. In my case, I had experienced an increase in idling from 3-5% CPU consumption (no `systemd`) to up to 10-15% CPU consumption with `systemd` enabled 😞.


## 💻 Workflow integration


That's it, once the basic configuration steps are completed, you can call Linux from the Windows start menu, with its dedicated CLI, or call it inside the Windows Terminal, which you can customize its appearance of.

In order to better integrate WSL into your workflow, you want to install an IDE which supports WSL integration. This guide will go over how to set it up in VSC or any JetBrain IDE. 


### Prepare for coding


Unfortunately, due to the nature of the operating system, you can not directly transfer or clone an existing `python` environment or `R` library from Windows to Linux, you have to start from scratch 🙂. 

To begin with, you need to have (1) your Linux repository and UNIX compiler updated and (2) a package manager like `mamba` or `conda` installed (refer to their documentation). This piece of code will do for (1) `sudo apt update && sudo apt upgrade` (password required). Furthermore, whereas `python` comes with the Linux installation, `R` needs to be installed afterward (no separate RTools installation is required):

```sh
# Add repository for package access and download
deb https://cloud.r-project.org/bin/linux/ubuntu jammy-cran40/
sudo apt update

# Install R
sudo apt install r-base r-base-dev
```

To launch R and install the desired packages, run `sudo R` inside the terminal. Similarly, it works for `python` via `mamba` or `conda`.


### Start coding

Now it is finally time to start coding and pick an IDE (e.g. Visual Studio Code, VSC or JetBrain). For all the RStudio fans, you can not connect your Windows version to WSL, but you can install RStudio inside WSL, enable GUI application in the `.wslconfig` file, and fire it from the terminal. However, because it is a GUI inside WSL, expect slowing downs. However, if you feel "experimental" 🧪 you can code directly inside the terminal and forget about GUI-related slowing downs 🏎️. For those less adventurous, you can install plugins inside VSC or JetBrain to improve your R experience.

Regarding VSC, it needs to be installed on Windows first (Linux will find its installation). Then, from the Linux terminal, navigate inside your working directory and run `code .` to fire an instance. You will notice the WSL icon on the bottom left corner of the IDE. From now on, you code runs inside WSL and you can take advantage of simpler parallelization syntax. Regarding all JetBrain products, please refer to their detailed documentation [here for Pycharm](https://www.jetbrains.com/help/pycharm/using-wsl-as-a-remote-interpreter.html)

_If you plan on doing any resource monitoring inside WLS, bear in mind that the it is relative to the virtual machine resources and not fully explanatory of the overall resources used by your machine._


_Author: Tommaso Trotto_
