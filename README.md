# Objective
Unmount the workspace storage account previously mounted using the legacy mounting driver which has performance concerns and remount it using SMB with Managed Identity. This approach removes the dependency on storage account access keys and leverages Managed Identity for SMB (Preview).

Supports both **Azure Machine Learning (AML)** compute instances and **AI Hub (AFH)** compute instances.

# Reference:
[Use Managed Identities with Azure Files (preview) | Microsoft Learn](https://learn.microsoft.com/en-us/azure/storage/files/files-managed-identities?tabs=linux)

# Prerequisites

- Access to the workspace storage account
- Permission to create and assign User-Assigned Managed Identities
- Ability to create a new compute instance
- A snapshot of the *-code file shares on storage account per the documentation [Use share snapshots with Azure Files
](https://learn.microsoft.com/en-us/azure/storage/files/storage-snapshots-files?tabs=portal) - The script is not expected to change anything on the file share content but having backup prior to anychange is always recommended action


# Setup Steps
1. Enable Managed Identity for SMB

- Go to the Storage Account
- Navigate to Settings → Configuration
- Set Managed Identity for SMB to Enabled


2. Create and Configure a User-Assigned Managed Identity

- Create a User-Assigned Managed Identity
- Assign the following roles on the workspace storage account:

- - Storage File Data Privileged Contributor
- - Storage Blob Data Contributor
- - Storage File Data SMB MI Admin




3. Update Script Configuration
Edit the `config.env` file:

- `INSTANCE_TYPE` → Set to `"AML"` for Azure Machine Learning or `"AFH"` for AI Hub
- `STORAGE_ACCOUNT` → Workspace storage account name

**For AML instances:**
- `SHARE_NAME` → The single code file share to be mounted (e.g. `workspaceId-code`)

**For AI Foundry instances:**
- `SHARE_NAMES` → Space-separated list of file share names in the format `workspaceId-code`
  ```
  SHARE_NAMES="xxxxxx-code xxxxxxx-code"
  ```
  The workspace IDs can be found from the folder names under `/afh/projects/` (the portion after the project name, e.g. `project1-<workspaceId>`).

- Leave all other parameters unchanged


4. Create a Compute Instance

- Create a new compute instance
- Assign the User-Assigned Managed Identity created in Step 2


5. Run the Mount Script

Upload the script folder to your Notebooks (or shared directory for AFH).
From the compute instance terminal, run:
```
bash setup.sh
```



# Notes

- This setup is required once per compute instance
- A service is automatically created to persist the mount across restarts
- You should see improved performance, especially for operations like git clone
- This is an experimental, best-effort script
- **AML mode**: remounts the single hostname-based code path
- **AFH mode**: discovers all fuse mount paths for each configured share (via `findmnt`) and remounts them with managed identity. The share names are validated against `/afh/projects/` as a double-check

