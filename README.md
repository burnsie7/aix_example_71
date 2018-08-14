# NOTE: These instructions are specfically for IBM PDP AIX 7.1 on POWER8

### Expand /opt FileSystem by 1G
`chfs -a size=+1G /opt`

### FROM YOUR MAC
#### Upload and extract Datadog AIX Agent's dependencies
`scp ddagent_aix_7.1_deps.tar.gz u0020940@<LPAR_IP_ADDRESS>:/tmp/`

### FROM THE LPAR
#### Create a datadog dir in /opt
```
mkdir /opt/datadog
cd /opt/datadog
```
_Assuming CWD is /opt/datadog_

```
mv /tmp/ddagent* .
gunzip ddagent_aix_7.1_deps.tar.gz
tar xvf ddagent_aix_7.1_deps.tar
```
#### Install yum and basic RPMs from AIX Toolbox
`chmod +x ddagent_aix_7.1_deps/yum.sh`
`ddagent_aix_7.1_deps/yum.sh`

#### Install packages: wget, python-pip
`yum install -y wget python-pip`

#### Fetch Datadog Installer (Example is Beta 4, replace URL with latest available beta)
_Assuming CWD is /opt/datadog_
`wget http://s3.amazonaws.com/dd-agent/custom/datadog-aix-installer.7-beta-4.bsx`

#### Extract Datadog Installer
_Assuming CWD is /opt/datadog_

```
chmod +x datadog-aix-installer.7-beta-4.bsx
./datadog-aix-installer.7-beta-4.bsx
```
---------------------------------------------
### Note: Optional Virtualenv steps ahead. Helps to keep python environments separated for updates to later Betas

#### Install pip packages: virtualenv
`pip install virtualenv`

#### Remove and re-build the python virtualenv
_Assuming CWD is /opt/datadog_
```
rm -rf datadog-unix-agent/venv
python -m virtualenv datadog-unix-agent/venv
source datadog-unix-agent/venv/bin/activate
```
---------------------------------------------

#### Install Python Requirements
`pip install -r datadog-unix-agent/requirements.txt`

#### Fetch and install pre-compiled pustil wheel, specially compiled for 7.1 ONLY
_Assuming CWD is /opt/datadog_

`pip install ddagent_aix_7.1_deps/psutil-5.4.6-cp27-cp27-aix_7_1.whl`

Note: Known bug with hostname formatting. [Github issue link](https://github.com/DataDog/datadog-unix-agent/issues/43)
#### Edit the config file to manually specify the hostname (replacing underscores with hyphens) and DD API Key
Example datadog.yaml:
```
hostname: l490vp026-pub
dd_url: https://app.datadoghq.com
api_key: 123456789123456789
log_level: debug
```
`vi datadog-unix-agent/datadog.yaml`

#### Run the Datadog AIX Agent in background mode
Known bug with having to be in the `agent.py` dir to honor the `datadog.yaml`.
```
cd datadog-unix-agent
./agent.py -b start
```
