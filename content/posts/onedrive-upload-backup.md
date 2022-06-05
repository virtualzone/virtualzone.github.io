---
title: "Back up server to OneDrive’s special App Folder"
date: 2021-09-02T11:30:03+00:00
tags:
    - github
    - onedrive
    - tool
author: "Heiner"
aliases:
    - /2021/09/back-up-server-to-onedrives-special-app-folder/
---

I’m a convinced user of OneDrive Personal. Bundled with M365, it’s a cheap option to get 1 TB of cloud storage. Having plenty of cloud storage at hand, I’m also using my OneDrive to run automated backups of my servers. There are various solutions capable of uploading files to OneDrive, including [rclone](https://rclone.org/). However, I was looking for a solution which enables me to grant my backup script only access to one specific folder instead of my entire cloud drive – better safe than sorry. I couldn’t find any. This is why I developed [OneDrive Uploader](https://github.com/virtualzone/onedrive-uploader). Here is what it can do for you and how to use it.

Microsoft OneDrive supports so-called “special folders”, which includes the “App Folder” (App Root). This is a directory intended for applications to storage their own files, without being able to access other files in your OneDrive Folder. OneDrive Uploader supports these special folders, restricting the access of your backup script to its own files. However, you can also use OneDrive Uploader to upload and download files from other locations as long as you grant it access.

I’ve written OneDrive Uploader in Go, which is a great programming language that compiles natively to various operating systems and platforms. As a result, OneDrive Uploader is available for Linux, MacOS and Windows and supports AMD64, ARM and ARM64.

To get started with OneDrive Uploader, you’ll need to create an access token in Microsoft’s Azure Portal. To do this, follow these steps:

1. Log in to the Microsoft [Azure Portal](https://portal.azure.com/).
1. Navigate to “App registrations”.
1. Create a new application with supported account type “Accounts in any organizational directory (Any Azure AD directory – Multitenant) and personal Microsoft accounts (e.g. Skype, Xbox)” and the following Web redirect URL: http://localhost:53682/
1. Copy the Application (client) ID.
1. Navigate to “Certificates & secrets”, create a new Client secret and copy the Secret Value (not the ID).
1. Navigate to “API permissions”, click “Add permission”, choose “Microsoft Graph”, select “Delegated”. Then search and add the required permissions:
  -  Access to App Folder only: Files.ReadWrite.AppFolder, offline_access, User.Read
  -  Access to entire OneDrive: Files.Read, Files.ReadWrite, Files.Read.All, Files.ReadWrite.All, offline_access, User.Read

Great! You’ve now created an Azure App which you can use to grant OneDrive Uploader access to your OneDrive. Don’t worry, the App is not visible anywhere, nor can anyone access your OneDrive.

You can now download the OneDrive Uploader executable for your operating system and platform. You can either choose the matching binary from the GitHub releases page, or simply execute this command:

```curl -s -L https://git.io/JRie0 | bash```

Now create a configuration file named config.json. Replace <client id from azure app> and <client secret from azure app>:

```json
{
    "client_id": "<client id from azure app>",
    "client_secret": "<client secret from azure app>",
    "scopes": [
        "Files.ReadWrite.AppFolder",
        "offline_access"
    ],
    "redirect_uri": "http://localhost:53682/",
    "secret_store": "./secret.json",
    "root": "/drive/special/approot"
}
```

As you can see in the config.json above, we specify the special app folder as OneDrive Uploader’s root directory. The two scopes grant access to the this app folder and allows automatic renewing the necessary access token without user interaction (which is essential for unattended backups).

Perform the log in using this command and follow the instructions printed on your console:

```onedrive-uploader login```
You can now use OneDrive Uploader. To view the available commands, refer to the project’s GitHub page or type:

```onedrive-uploader help```
To use OneDrive Uploader in your backup script, you can be guided by this shell script snippet:

```bash
#!/bin/bash
DIR_FORMAT="%Y-%m-%d" # DD-MM-YYYY format
TODAY=`date +"${DIR_FORMAT}"`
TARGET=/mnt/backup/$TODAY
UPLOADER="/usr/local/bin/onedrive-uploader -c /home/username/backup-script/config.json"
```

## Perform your local backup and store it in ${TARGET}

```bash
echo "Uploading..."
cd ${TARGET}
${UPLOADER} mkdir ${TODAY}
for i in `ls`; do
    ${UPLOADER} upload $i ${TODAY};
    HASH_REMOTE=`${UPLOADER} sha256 $TODAY/$i | tr '[A-Z]' '[a-z]'`
    HASH_LOCAL=`sha256sum $i | tr '[A-Z]' '[a-z]' | awk '{ print $1 }'`
    if [[ "$HASH_REMOTE" != "$HASH_LOCAL" ]]; then
        echo "Hashes for '$i' do not match! Remote = $HASH_REMOTE vs. Local = $HASH_LOCAL"
    fi
done
```

This bash script uploads all files from the local directory $TARGET to its app folder in your OneDrive. It creates a sub-folder named ```YYYY-MM-DD``` (i.e. 2021-08-30). For each file, after having finished the upload, it checks she SHA256 hash so that you can be sure the upload is intact.