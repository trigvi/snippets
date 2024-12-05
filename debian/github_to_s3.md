# Backing up GitHub code repositories to S3 on Linux Debian

These are the steps for creating a Linux Debian bash script that downloads the code for all your GitHub repositories into separate zip files and uploads them to an AWS S3 bucket.

### How to create it

* Create GitHub token by performing these steps: 1) log into your GitHub, 2) click your icon on the top-right corner, 3) Settings, 4) Developer settings, 5) Personal access tokens, 6) Tokens (classic), 7) Generate new token (classic), 8) Enter a name in the *Note* field, 9) Tick *repo* and all its sub checkboxes, 10) Click *Generate token*.

* Install AWS CLI on your Debian machine, instructions at https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

* Using your browser: log into your AWS, create S3 bucket, create IAM user and note down its Key/Secret, give the IAM user permissions to upload to the S3 bucket.

* Create AWS CLI profile on your Debian machine, let's say we call it *joe*.
    ```
    mkdir ~/.aws
    echo '[joe]' >> ~/.aws/credentials
    echo 'aws_access_key_id = <YOUR_VALUE>' >> ~/.aws/credentials
    echo 'aws_secret_access_key = <YOUR_VALUE>' >> ~/.aws/credentials
    echo '[joe]' >> ~/.aws/config
    echo 'region = <YOUR_BUCKET_REGION>' >> ~/.aws/config
    ```

* Create an empty bash script called *backup_github_code_to_s3.sh*, give it strict permissions to prevent other users from viewing it.
    ```
    touch backup_github_code_to_s3.sh
    chmod 700 backup_github_code_to_s3.sh
    ```

* Open *backup_github_code_to_s3.sh*, insert the following contents, set your param values in the first few lines, save the file:
    ```
    #!/bin/bash

    # Set your params
    aws_cli_profile=<YOUR_VALUE>
    aws_s3_base_url=<YOUR_VALUE> # e.g. s3://bucketname/path/to/directory (no trailing slashes)
    aws_s3_region=<YOUR_VALUE>
    github_token=<YOUR_VALUE>
    github_user=<YOUR_VALUE>

    # Create temporary directory for this job
    current_ms=$(date +'%s%N')
    temp_dir_path=/tmp/backup_github_code_to_s3_$current_ms
    mkdir -p $temp_dir_path
    cd $temp_dir_path

    # Fetch list of Github repos
    github_repos=$(curl -sL --request GET https://api.github.com/user/repos -H "Authorization: token $github_token" | grep '"name"' | awk -F\" '{print $4}')
    oldifs=$IFS
    IFS=$'\n'
    github_repos_array=($github_repos)
    IFS=$oldifs

    # Download each Github repo as a zip file and upload to S3
    for github_repo in "${github_repos_array[@]}"
    do
        filename=codebase_$github_repo.zip
        echo "Downloading from Github: $filename ($github_user/$github_repo)"
        curl -sL --request GET "https://api.github.com/repos/$github_user/$github_repo/zipball" -H "Authorization: token $github_token" > $filename

        echo "Uploading to S3: $filename"
        aws s3 cp $filename $aws_s3_base_url/$filename --region $aws_s3_region --profile $aws_cli_profile
    done

    # Remove temporary directory
    rm -rf $temp_dir_path
    ```

* Execute the bash script
    ```
    ./backup_github_code_to_s3.sh
    ```
