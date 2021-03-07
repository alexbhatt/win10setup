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
# Administrator: PowerShell
# install chocolatey
	Set-ExecutionPolicy Bypass -Scope Process
	Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

# show all installed packages
	choco list -l
	choco upgrade all

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

```sh
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

```sh
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

```sh
# Administrator: PowerShell

	choco install atom

	refreshenv

# add to apm and atom to PATH
	$userenv = [System.Environment]::GetEnvironmentVariable("Path", "User")
	[System.Environment]::SetEnvironmentVariable("PATH", $userenv + ";$HOME\AppData\Local\atom\bin", "User")
```

```sh
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

```sh
# Administrator: PowerShell

	choco install r.project -y
	choco install rtools -y
	choco install r.studio -y
```

Use the package manager [renv](https://rstudio.github.io/renv/articles/renv.html) right from the start on a project-by-project basis. There is really no reason not to. Works brilliant with Docker.

```r
# R

# allow development
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
Download and install python via the exe installer.
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

# add the jupyter kernel
	python -m ipykernel install --user --name=penv

# activate the environment, you'll see (penv) in the CLI after its activated
	.\Scripts\Activate.ps1 # PS
	.\Scripts\Activate.bat # cmd
	source .\Scripts\activate  # bash

# add your Packages
	pip install numpy scipy pandas # manage data
	pip install scikit-learn # machine learning
	pip install tensorflow # machine learning; install not working
	pip install plotly matplotlib seaborn # data visualisation

# save the requirements
	pip freeze > requirements.txt

# close up shop
	deactivate

# this will allow recall of the environment in the future using
	python -m pip install -r requirements.txt
```

### Jupyter
Lightweight IDE for Python and R notebooks run in your default browser. These are very sharable.

```sh
# PowerShell
	pip install jupyterlab notebook
	pip install --user ipykernel

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

## [Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/install-win10#manual-installation-steps)
Follow the [microsoft guide](https://docs.microsoft.com/en-us/windows/wsl/install-win10), for best results activate WSL2.

### [Docker](https://www.docker.com/products/docker-desktop)
Docker is a container management system for reproducible environments.
It is completely agnostic.
It is necessary to allow us to send analysis to a kubernetes based system like the PHE OpenShift high performance computer cluster.

#### Install

1. WSL activated; can be done without, but its less efficient
1. Download [`Docker Desktop Installer.exe`](https://www.docker.com/products/docker-desktop) and run as LocalAdmin
1. Add to user group in Administrator: PowerShell
1. RESTART the machine to enable the group changes
1. RUN Docker Desktop; it will configure to run on start-up after this

```powershell
# Administrator: PowerShell
	net localgroup docker-users "alex.bhattacharya@phe.gov.uk" /add
```
```sh
docker image list
docker pull IMAGENAME
```
### [Airflow](https://airflow.apache.org/docs/apache-airflow/stable/start/local.html)
+ Prerequisite: Python
+ Prerequisite: Docker

Airflow is a DAG manager for data pipelines, it can be installed locally or via Docker.

#### local install
```sh
# GitBash
# setup a venv
	python -m venv airflow_local
	cd airflow_local
	python -m pip --upgrade pip
	source /Scripts/activate

# https://airflow.apache.org/docs/apache-airflow/stable/start/local.html
	export AIRFLOW_HOME=~/airflow
	AIRFLOW_VERSION=2.0.1
	PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
	CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
	pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"

# save the environment
	pip freeze > requirements.txt
```
#### [docker install](https://airflow.apache.org/docs/apache-airflow/stable/start/docker.html)
```sh
# GitBash
	mkdir airflow
	cd airflow
	curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.0.1/docker-compose.yaml'
	mkdir ./dags ./logs ./plugins
	echo -e "AIRFLOW_UID=$(id -u)\nAIRFLOW_GID=0" > .env
	docker-compose up airflow-init
	docker-compose up
```

### Linux distribution
+ Prerequisite: WSL

An alternative distro would be Debian, but Ubuntu is the most common.
It is helpful to have a WSL Linux distro available for testing out containers.
Note you may not have access to the wider system environment and AD access from within WSL.

1. WSL activated
1. [manually download a Linux distribution as the Microsoft Store is unavailable](https://docs.microsoft.com/en-us/windows/wsl/install-manual)
1. [Follow the guide to install](https://docs.microsoft.com/en-us/windows/wsl/install-on-server)
1. Setup root user account on the first time you run the distro, this is separate from your AD account and is purely for managing the distro in the closed environment.

#### Ubuntu 20.04
```sh
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
```sh
## WSL2
# updates the install with the core dependencies and installed programs
	sudo apt update && sudo apt -y upgrade && sudo apt autoremove

# some common dependencies
	sudo apt install wget curl
	sudo apt install software-properties-common
	sudo apt install apt-transport-https
	sudo apt install gnupg-agent
	sudo apt install ca-certificates
	sudo apt install libcurl4-openssl-dev libcurl4-gnutls-dev libssl-dev libxml2-dev
	sudo apt install unixodbc-dev msodbcsql17

# install python
	sudo apt update
	sudo apt install libpython3-dev
	sudo apt install -y python3 python3-pip python3-venv ipython

	pip3 install --user jupyterlab NumPy SciPy pandas Matplotlib seaborn

# to launch jupyter, need --no-browser since in WSL
	jupyter lab --no-browser
	http://localhost:8888/

# install R
# https://support.rstudio.com/hc/en-us/articles/360049776974-Using-RStudio-Server-in-Windows-WSL2
	sudo apt install dirmngr
	sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
	sudo add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu focal-cran40/'
	sudo apt install -y r-base r-base-core r-recommended r-base-dev
	sudo apti install -y gdebi-core build-essential

# Install RStudio server
	wget https://rstudio.org/download/latest/stable/server/bionic/rstudio-server-latest-amd64.deb
	sudo gdebi rstudio-server-latest-amd64.deb
	sudo rstudio-server start

#   login using UNIX username:password
	http://localhost:8787/

```
