---
title: "Why I use Restic as my backup tool"
date: 2020-09-15T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["backup", "technical"]
categories: ["Sysadmin"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/<path_to_repo>/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---


## Restic

As a certified digital hoarder, I've lost more files in my life than I can count. One day, after losing a particular file that contained a week's worth of work, I decided I'd had enough. I searched the internet, determined to find a backup solution that would save me from my own clumsiness. and after some digging, I stumbled upon ***[Restic](https://restic.net/).*** 
Restic is an open-source backup tool that emphasizes ease of use and versatility. It offers robust features like deduplication, encryption, and snapshotting that make backing up data a breeze.

The main reasons which drown me to Restic were:

-   **Free** **Open Source:** anyone can view, edit, and contribute to the source code, fostering an environment of collaboration and continuous improvement.
-   **Easy to Setup & Deploy:** since it’s written in Go. It is a single binary executable that you can run without a server or complex setup
-   **Multiple Storage Backends:** it can support local storage, network drives, or cloud storage.
-   **Deduplication and Encryption:** uses deduplication to save storage space. Deduplication works by only storing new parts of files that have changed since the last backup. Restic encrypts your data before it leaves your machine. This means your data is always secure, whether it’s sitting on your backup server or being transmitted over the internet.

## Using Restic
### 1- Initializing Backup Repository
Before you can start backing up your data, you need to initialize a backup repository. This is where Restic will store your backup data.

To initialize a repository, you use the `init` command followed by the `-r` option and the location where you want to store your backups. For example, if you want to store your backups in a local directory called `/srv/mybackup`, you would use the following command:
```bash
restic -r /srv/mybackup init
```
### 2- Taking a Backup
After initializing your repository, you can start backing up your data. To do this, you use the `backup` command followed by the `-r` option and your repository location, then the path to the data you want to back up.

For example, to back up a directory called `/home/user/documents`, you would use the following command:
```bash
restic -r /srv/mybackup backup /home/user/documents
```
Restic will then back up your data and print a summary when it's done.

### 3- Listing Snapshots
In Restic, you can easily view all the snapshots you have taken with the `snapshots` command. This command will list all the snapshots in your repository, along with their ID, date, and the directories they contain.

To list your snapshots, simply use the `snapshots` command followed by the `-r` option and your repository location. For example, if your repository is located in a local directory called `/srv/mybackup`, you would use the following command:
```bash
restic -r /srv/mybackup snapshots
```
The output of this command will show a list of your snapshots, like this:
```bash
ID Time Host Tags Paths
 ---------------------------------------------------------------------- 
 ae4rtg 2023-11-26 11:23:00 mycomputer /home/user/documents 
 def456 2023-11-27 10:43:00 mycomputer /home/user/documents 
 ---------------------------------------------------------------------- 
 2 snapshots
 ```
 
 In this example, ae4rtg and def456 are the IDs of the snapshots.
 ### 4- Restoring a Backup
 If you need to restore your data from a backup, you can use the `restore` command followed by the `-r` option, your repository location, the snapshot ID you want to restore from, and the `-t` option followed by the location where you want to restore your data.

For example, to restore a snapshot with the ID `ae4rtg` to a directory called `/restore`, you would use the following command:
```bash
restic -r /srv/mybackup restore ae4trg -t /restore
```
## Conclusion
In conclusion, Restic is a powerful tool for backing up your data. Its ease of use, combined with features like deduplication, encryption, snapshotting, and easy restores make it an excellent choice for anyone looking for a robust and reliable backup solution. As with any tool, it may not be perfect for every situation, but its versatility and simplicity make it worth considering for your backup needs.