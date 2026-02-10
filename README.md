## Overview
This PowerShell tool is designed to be executed from a Domain Controller to collect and validate the state of machines in the environment.
It retrieves detailed information about:
* Machine status
* Associated user
* Tags
* Machine Catalog
* Delivery Group

> ⚠️ **Important:** This script must be executed from **PowerShell Easy** as **Administrator**.
> 
> ⚠️ **Requirement:** This script must be executed from a **Domain Controller**.

### Features
The script automatically:
* Detects if it is running on a Domain Controller
* Queries all machines in the environment
* Retrieves:
    * Machine state
    * Logged user
    * Tags
    * Machine Catalog
    * Delivery Group
* Displays a structured overview for troubleshooting and auditing
