# Analytics Setup 

To ensure a clean and robust analytic environment a few dependencies are required.
Local Adminstrator rights are necessary to install some of these.

## [Git](https://git-scm.com/downloads)

You will want to use both GitHub (public) and the internal [GitLab](https://gitlab.phe.gov.uk)
It may be helpful to setup an SSH keypair for GitLab.

1. generate ssh: 	`ssh-keygen -t ed25519 -C "alex.bhattacharya@phe.gov.uk"`
1. copy ssh: 		`cat ~/.ssh/id_ed25519.pub | clip`
1. paste ssh in gitlab ssh user profile page
1. test: 		`ssh -T git@gitlab.phe.gov.uk`

### VIM
Vim is a CLI based text editor packaged up in Git Bash and Linux; [read this guide](https://github.com/damassi/learn-vim). 

```vim
# use vim to create a settings file
vim ~\.vimrc

# add the following settings
set numbers
```

## [R](https://cran.r-project.org/mirrors.html)
The primary open-source programming used for scientific use and epidemiology. Can be used in conjunction with Python. 

### Also install
+ [Rtsudio](https://rstudio.com/products/rstudio/download/): This IDE can run both R and Python code. Also has great markdown support.
+ [Rtools](https://cran.r-project.org/bin/windows/Rtools/): This is necessary dependency for functional programming and development and is often a depenedency in R.

I would recommend using the package manager [renv](https://rstudio.github.io/renv/articles/renv.html) right from the start. Use this to manage your packages on a project-by-project basis.

```r
# Run in R
# everything else should be saved in projects

	install.packages("renv")
	install.packages("devtools")
```

## [Python](https://www.python.org/downloads/)
Download and install base python. You will be able to call it from PowerShell, Rstudio and Jupyter.

+ PIP: This is your python package management tool and you will use this to get the rest of your python tools and updates. It comes packaged with the python installation. 

### Jupyter
Lightweight IDE for Python and R notebooks.

To install run in CLI: `pip install jupyterlab notebook`

To run R within Jupyter, you need to make the R kernel accessible. This will be auto-detected by Jupyter afer its run in R.

```r 
# Run in R
	install.packages('IRkernel')
	IRkernel::installspec()
```

## [Windows Terminal](https://docs.microsoft.com/en-us/windows/terminal/get-started)
This is an integrated terminal which allows use of many CLI tools in a clean interface. You will need to download and install follwing instructions from the [GitHub](https://github.com/microsoft/terminal) page. 

### Installation
Run cmd.exe as Admin to install Chocolatey package manager; then use it to run the following:

```bash
# using a package manager will allow updates in the future
	`@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"`

# Restart the terminal and run again as Admin:

	choco install microsoft-windows-terminal
	choco upgrade microsoft-windows-terminal 

# this will allow us to run as LocalAdmin in a WT profile
	choco install gsudo
```

## [WSL](https://docs.microsoft.com/en-us/windows/wsl/install-win10#manual-installation-steps)
Follow the microsoft guide for activation, for best results actviate WSL2.

### Linux distribution
You will need to [manually download a Linux distribution as the Microsoft Store is unavailable](https://docs.microsoft.com/en-us/windows/wsl/install-manual). [Follow the guide to install](https://docs.microsoft.com/en-us/windows/wsl/install-on-server).

```powershell
# Administrator: PowerShell
cd $HOME

## Download distro (Ubuntu 20.04)
Invoke-WebRequest -Uri https://aka.ms/wslubuntu2004 -OutFile .\Downloads\Ubuntu.appx -UseBasicParsing

## Open it
Rename-Item .\Downloads\Ubuntu.appx .\Downloads\Ubuntu.zip
Expand-Archive .\Downloads\Ubuntu.zip .\Ubuntu

## Run it
\Ubuntu\ubuntu2004.exe

## add to PATH
$userenv = [System.Environment]::GetEnvironmentVariable("Path", "User")
[System.Environment]::SetEnvironmentVariable("PATH", $userenv + ";C:\Users\Administrator\Ubuntu", "User")
```
You will need to setup a user account for root access on the first time you run the distro.
This is seperate from your AD account and is purely for managing the distro.

#### Ubuntu 20.04
```
## on first run
sudo apt update && sudo apt upgrade && sudo apt autoremove

## analytic framework
sudo apt install virtualbox
sudo apt install git

## docker dependencies
sudo apt-get install \
	apt-transport-https
	ca-certificates \
	curl \
	gnupg-agent \
	software-properties-common
```

### [Docker](https://www.docker.com/products/docker-desktop)
Once downloaded and installed, you will need to run PowerShell as Administrator to add yourself to the correct usergroup. Without this, Docker will be unable to start. Note that WSL is a prerequiste step.

1. PowerShellAdmin: `net localgroup docker-users "alex.bhattacharya@phe.gov.uk" /add`
1. RESTART the machine to enable the group changes
1. RUN Docker Desktop; it will configure to run on startup after this

## Airflow
