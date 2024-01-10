# Unity Catalog Migration Toolkit - UCX

## Installation
### Prerequisties
- UC catalog activated in your workspace
- UC metastore created
- Azure/AWS Account level setup
- Install Azure CLI
- Install Databricks CLI

### Download & install
download the [latest release](https://github.com/databrickslabs/ucx/releases) from github to your local system, Unzip or untar the release.

To run the `./install.sh`, you need to:
- Open the ucx folder in VSCode, create a virtual environment
- Make sure you have Python 3.10 (or greater) installed on your workstation
- you've configured authentication for the Databricks Workspace locally at ~/.databrickscfg, which look like this
    ```
    [DEFAULT]
    host = https://adb-xxxxxxxxxxxx.xx.azuredatabricks.net
    account_id = xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx

    [DEV]
    host = https://adb-xxxxxxxxxxxx.xx.azuredatabricks.net
    ```
- Use Azure CLI to login, switch account (to where you databricks located)
- Open a bash terminal(git bash/WSL/Azure cloud shell) and set environment variable `DATABRICKS_CONFIG_PROFILE` to specify the profile
- run `./install.sh`