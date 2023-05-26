# ACR Cleanup GitHub Action

This _GitHub Action_ performs cleanup of an _Azure Container Registry_ (_ACR_) by keeping the last 5 tags and deleting the rest. The cleanup is scheduled to run every _Friday_ at _11 PM_

## Usage

To use this action, add the following YAML configuration to your repository:

```yaml
name: ACR Cleanup

on:
  schedule:
    - cron: "0 23 * * 5" # Every Friday at 11 PM

jobs:
  cleanup:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Login to Azure
        uses: azure/login@v1

      - name: Run cleanup script
        run: |
          #!/bin/bash

          PROJECT_NAME="$1"
          ACR_NAME="${1}acr"

          projects=( $(find ./src -type d -printf "%f\n") )

          for REPOSITORY in "${projects[@]}"
          do
            echo "${PROJECT_NAME}/${REPOSITORY}"

            TAGS=$(az acr repository show-tags --name $ACR_NAME --repository $REPOSITORY --orderby time_desc --top 5 --output tsv)

            # Delete each tag
            for TAG in $TAGS; do
              az acr repository delete --name $ACR_NAME --image $REPOSITORY:$TAG --yes
            done
          done
```

## Configuration

The configuration options for this action are as follows:

`on.schedule` (_required_): Specifies the schedule for the action to run. By default, it is set to run every _Friday_ at _11 PM_ using the cron expression `"0 23 * * 5"`

```bash
* * * * *
| | | | |
| | | | +----- Day of the Week (0 - 7) (Sunday = 0 or 7)
| | | +------- Month (1 - 12)
| | +--------- Day of the Month (1 - 31)
| +----------- Hour (0 - 23)
+------------- Minute (0 - 59)
```

## Script Explanation

The script used in this action performs the following steps:

- Retrieves the list of folder names in the `./src` directory of your repository.

- Iterates through each folder name and performs the cleanup for each repository.

- Retrieves the last 5 tags for each repository using the `az acr repository show-tags` command.

- Deletes each tag using the `az acr repository delete` command.

> Ensure that you have the necessary permissions and credentials to access the _Azure Container Registry_ and execute the _Azure CLI_ commands.
