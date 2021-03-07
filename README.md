# Windows 10 Analytic Setup

To ensure a clean and robust analytic environment a few dependencies are required.
Local Administrator rights are necessary to install some of these.
Where possible, I have installed via code rather then using the *.exe installer.
SQL Server Management Studio is not included here as that should be installed by ICT.

### [Chocolatey](https://chocolatey.org/)
This is a Windows package manager for allowing us to install and update installations through the CLI for our toolkit.
It _always_ requires LocalAdmin to run, and can be run in either `cmd.exe` or `PowerShell`.
This makes maintaining the tools and versions considerably easier.
_If WinGet (developed my Microsoft) is available then this will be updated._

```sh
# Administrator: cmd.exe
# install chocolatey
	`@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"`

# general syntax
	choco install packagename
	choco upgrade packagename
```

## [Windows Terminal](https://docs.microsoft.com/en-us/windows/terminal/get-started)
This is an integrated terminal which allows use of many CLI tools in a clean tab-based interface.
You will need to download and install following instructions from the [GitHub](https://github.com/microsoft/terminal) page.
Once installed, you can set your profiles for the various tools in the settings.json.

```sh
# PowerShell
# Note your .config files will live in $HOME

# create a base layer for all project repos
mkdir $HOME\repos
```
Set the GitBash profile in Windows Terminal to start in `%USERPROFILE%\\repos` the rest will probaly want to start in `%USERPROFILE%`.

```powershell
# Administrator: PowerShell/cmd
# reset the terminal (imported by chocolatey)
	refreshenv

	choco install microsoft-windows-terminal

# allows us to run Administrator: PowerShell in Windows Terminal with password
	choco install gsudo
```


## [Git](https://git-scm.com/downloads)
Git is essential for version control.
Both [GitHub](https://github.com/alexbhatt) and the internal [GitLab](https://gitlab.phe.gov.uk/alex.bhattacharya) use the same installation.

```powershell
# Administrator: PowerShell/cmd
	choco install git
```

Open and the config file and add the following to the profile.
```bash
# GitBash
	vim ~/.gitconfig
```

```vim
# Git user global config file
[user]
	name = Alex Bhattacharya
	email = alex.bhattacharya@phe.gov.uk

# set usernames for various git instances
[credential "https://github.com"]
	username = alexbhatt

[credential "https://gitlab.phe.gov.uk"]
	username = alex.bhattacharya
```

It may be helpful to setup an SSH keypair for GitLab in GitBash.
Paste the SSH key into the [GitLab profile manually](https://gitlab.phe.gov.uk/profile/keys).

```bash
# GitBash
# generate the SSHs
	ssh-keygen -t ed25519 -C "alex.bhattacharya@phe.gov.uk"
# copy it
	cat ~/.ssh/id_ed25519.pub | clip
# test it
	ssh -T git@gitlab.phe.gov.uk
```

### VIM
Vim is a CLI text editor packaged up in GitBash and Linux that works through the terminal; [read this guide](https://github.com/damassi/learn-vim). Its very helpful for quick edits.

```sh
# GitBash
# use vim to create a settings file
	vim ~\.vimrc
```
```vim
# vim persistent settings
	set numbers
```

### [Atom](https://atom.io/)
Atom is a lightweight text editor made by Git. I like it.
It has excellent packages for markdown and is very customisable.

```powershell
# Administrator: PowerShell

	choco install atom

	refreshenv

# add to apm and atom to PATH
	$userenv = [System.Environment]::GetEnvironmentVariable("Path", "User")
	[System.Environment]::SetEnvironmentVariable("PATH", $userenv + ";$HOME\AppData\Local\atom\bin", "User")
```

```powershell
# PowerShell

# syntax highlighting
	apm install linter
	apm install linter-markdown
	apm install linter-ui-default

# markdown support
	apm install pp-markdown
	apm install markdown-scroll-sync
```

## [R](https://cran.r-project.org/mirrors.html)
The primary open-source programming used for scientific use and epidemiology.
Can be used in conjunction with Python.

+ [Rtsudio](https://rstudio.com/products/rstudio/download/): This IDE can run both R and Python code. Also has great markdown support.
+ [Rtools](https://cran.r-project.org/bin/windows/Rtools/): This is necessary dependency for functional programming and development and is often a dependency in R.

```powershell
# Administrator: PowerShell/cmd

# Recommendation is to install these from website installers
	choco install r.project
	choco install r.studio
	choco install rtools
```

Use the package manager [renv](https://rstudio.github.io/renv/articles/renv.html) right from the start on a project-by-project basis. There is really no reason not to. Works brilliant with Docker.

```r
# R

# allow developement
	install.packages("devtools")

# manage packages
	install.packages("renv")
# future updates of renv
	renv::upgrade()

# run python in R
	install.packages("reticulate")

```

### RMarkdown outputs
In order to render documents from code to PDF, word or slides, you will need an installation of [Pandoc](https://pandoc.org/installing.html) and [MiKTeX](https://miktex.org/).
This is necessary for any automated reporting.

```bash
# Administrator: PowerShell
	choco install pandoc miktex
```

## [Python](https://www.python.org/downloads/)
Download and install python via the exe installer; note this includes PIP.
Python is useful to have installed even if not using as the primary data tool.

### CLI tools
+ __pip__ is your python package management tool and you will use this to get the rest of your python tools, packages and updates.
+ __venv__ is your environment management tool, like renv, and will be used on a project basis.  

```sh
# for updating these
	python -m pip install --upgrade pip
# change to existing venv environment
	python -m venv --upgrade $HOME\repos\VENVDIR
```

```sh
# create a virtual environment
	python -m venv $HOME\repos\penv
	cd $HOME\repos\penv

# activate the environment, you'll see (penv) in the CLI after its activated
	.\Scripts\Activate.ps1 # PS
	.\Scripts\Activate.bat # cmd
	source .\bin\activate  # bash

# add your Packages
	pip install NumPy SciPy pandas Scikit-learn
	pip install tensorflow # not working
	pip install Matplotlib seaborn

# save the requirements
	pip freeze > requirements.txt

# this will allow recall in the future using
	python -m pip install -r requirements.txt

# close up shop
	deactivate
```

### Jupyter
Lightweight IDE for Python and R notebooks run in your default browser. These are very sharable.

```powershell
# PowerShell
	pip install jupyterlab notebook

# run to launch
	jupyter notebook
	jupyter lab
```

To run R within Jupyter, you need to make the R kernel accessible.
This will be auto-detected by Jupyter after its run in R, and only needs to be done once per R installation.

```r
# R
	install.packages('IRkernel')
	IRkernel::installspec()
```
### Airflow
Prerequisite: Python
Airflow is a DAG manager for workflows.

## [Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/install-win10#manual-installation-steps)
Follow the [microsoft guide](https://docs.microsoft.com/en-us/windows/wsl/install-win10), for best results activate WSL2.

### [Docker](https://www.docker.com/products/docker-desktop)
Docker is a container management system for reproducible environments.
It is completely agnostic.
It is necessary to allow us to send analysis to a kubernetes based system like the PHE OpenShift high performance computer cluster.

#### Install

1. WSL activated
1. Download [`Docker Desktop Installer.exe`](https://www.docker.com/products/docker-desktop) and run as LocalAdmin
1. Add to user group in Administrator: PowerShell
1. RESTART the machine to enable the group changes
1. RUN Docker Desktop; it will configure to run on start-up after this

```powershell
# Administrator: PowerShell
	net localgroup docker-users "alex.bhattacharya@phe.gov.uk" /add
```

### Linux distribution
An alternative distro would be Debian, but Ubuntu is the most common.
It is helpful to have a WSL Linux distro available for testing out containers.
Note you may not have access to the wider system environment and AD access from within WSL.

1. WSL actviated
1. [manually download a Linux distribution as the Microsoft Store is unavailable](https://docs.microsoft.com/en-us/windows/wsl/install-manual)
1. [Follow the guide to install](https://docs.microsoft.com/en-us/windows/wsl/install-on-server)
1. Setup root user account on the first time you run the distro, this is separate from your AD account and is purely for managing the distro in the closed environment.

#### Ubuntu 20.04
```powershell
# Administrator: PowerShell
	cd $HOME

# Download distro (Ubuntu 20.04)
	Invoke-WebRequest -Uri https://aka.ms/wslubuntu2004 -OutFile .\Downloads\Ubuntu.appx -UseBasicParsing

# Open it and save in the user account folder
	Rename-Item .\Downloads\Ubuntu.appx Ubuntu.zip
	Expand-Archive .\Downloads\Ubuntu.zip .\AppData\Local\Packages\Ubuntu

# add to PATH
	$userenv = [System.Environment]::GetEnvironmentVariable("Path", "User")
	[System.Environment]::SetEnvironmentVariable("PATH", $userenv + ";C:\Users\Administrator\Ubuntu", "User")

# Run it
.\AppData\Local\Packages\Ubuntu\ubuntu2004.exe

# set root user and password when prompted
```
```bash
## WSL2
# updates the install with the core dependencies and installed programs
sudo apt update && sudo apt upgrade && sudo apt autoremove

## analytic framework
sudo apt install virtualbox
sudo apt install git
sudo apt install

## docker dependencies
sudo apt-get install \
	apt-transport-https
	ca-certificates \
	curl \
	gnupg-agent \
	software-properties-common
```
