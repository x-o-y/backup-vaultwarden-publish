# Vaultwarden Backup with Github Actions

> [!NOTE] 
> - Backup single source vault account data to Github's repo; then import to another destination vault account.
> - You can use this to sync from Vaultwarden to Bitwarden, and vice versa.
> - We are using **Github Actions**🛡️ for automation. GitHub is all you need. (You don't need a server to run this.)
> - The workflow is divided into 2 parts: `Export Vault to Repo` and `Purge Vault and Import from Repo`; you can choose as you need.
> - The workflow is using **Bitwarden CLI**[^1] to export and import JSON files.

> [!Warning]
> - The destination vault will be purged before import!

## Use Cases
1. In case if your bitwarden/vaultwarden account data cannot be accessed or deleted by mistake, you can restore with the encrypted backups in GitHub – keep the encrypt key in another safe place.
2. Also, if you have two bitwarden/vaultden accounts, you can use this to sync from the source to the destination.

Backup JSON to **GitHub** repository, automated via **GitHub Actions**.
**A GitHub account is all your need**.

```mermaid
flowchart LR
    subgraph B[Backup new changes only]
        direction TB
        actions@{shape: hex, label: "GitHub Actions"}
        repo[GitHub Repository]
        backups@{ shape: docs, label: "JSON backups<br>Timestamped"}
        actions -->repo --> backups
    end

    A[ Source <br> Bitwarden/Vaultwarden <br> Account] -- export encrypted JSON --> B
    B -. purge & import to .-> C[Destination <br> Vaultwarden/Bitwarden <br> Account]

```


## Export Encrypted JSON from vaultwarden/bitwarden to Github repo 📤

- workflow name: `Export Vault to Repo`
- triggered by cron job, everyday at 21:53am UTC by default
- it will export an encrypted (using `JSON_ENCRYPTION_PASSWORD`) JSON named with timestamp: `backup_YYYYMMDD_HHMMSS.json`
- compare the hash value to check whether it's new
- and save them under `vault_backup`

## Purge and Import Encrypted JSON to vaultwarden/bitwarden from Github repo 📥

- workflow name: `Purge Vault and Import from Repo`
- triggered by the successful of the `Export Vault to Repo` workflow
- and if there is a new git push to the repo, otherwise stop the workflow
- it will delete ALL items (move to trash) belong to the default owner of the destination vault
- the items of other owners(organizations) will not be affected
- find the latest JSON file in the backup folder `vault_backup`
- import it to the destination vault

## Github Acitons Secrets 🔑

Your Repository Settings --> Secrets and Variables --> Actions --> New Repository Secret

- JSON_ENCRYPTION_PASSWORD: 
  - the password to encrypt during the export and import of JSON
- SOURCE_CLIENT_ID: 
  - the `client_id`, which can be found in the vault settings 
- SOURCE_CLIENT_SECRET: 
  - the `client_secret`, together with `client_id`
- SOURCE_PASSWORD: 
  - the Master Password to login
- SOURCE_SERVER_URL: 
  - the source host url, e.g., `https://vault.myhost.net`
- The following items are similar to above, as the destination's parameters.
  - DEST_CLIENT_ID
  - DEST_CLIENT_SECRET
  - DEST_PASSWORD
  - DEST_SERVER_URL

## Getting Started 🚀

- create a **private repo** in your Github
- copy the two workflow files to the folder `.github/workflows`:
  - `Export Vault to Repo - CompareHashes.yml`
  - `Purge Vault and Import from Repo - OnNewPush.yml`
- copy the first only if you don't need to import data to another vault
- it will just backup your vault data into the repo backup folder
- adjust the cron schedule as your preference (optional)
- setup the action secrets as mentioned above
- make sure the workflows are enabled
- done.

## Changelog 📝

- 2025.06.10 v1.6.1
  - automatic publish to public repo using .sh script
- 2025.06.08 v1.6
  - add sleep to make sure the repo has synced 
- 2025.06.07 v1.5
  - add hash comparison to avoid duplicate backup
- 2025-06-04 v1.0
  - initial release

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=x-o-y/backup-vaultwarden-publish&type=Date)](https://www.star-history.com/#x-o-y/backup-vaultwarden-publish&Date)

## References 🔗

[^1]: Bitwarden CLI, https://bitwarden.com/help/cli/