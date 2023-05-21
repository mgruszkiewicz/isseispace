---
title: "How to use Backblaze B2 as primary storage in Nextcloud"
date: 2023-01-23T23:30:56+02:00
draft: false
cover: "images/2023-01-23-nextcloud-b2/cover.png"
tags: ['en', 'object-storage']
---

To setup B2 as primary storage in Nextcloud you of course need to install NC first. For that, I used [setup-nextcloud.php](https://download.nextcloud.com/server/installer/setup-nextcloud.php) script - it is a nice, quick way of deploying nextcloud on shared hosting. Normally i would recommend setting it up as a docker container, but that will also work for testing.
While installing Nextcloud just pick default storage place for now.

## Creating bucket in Backblaze

If you don’t already have Backblaze account, you can register [here](https://www.backblaze.com/b2/sign-up.html). Backblaze gives 10GB of free storage + 1GB traffic per day for every user.  
Note: make sure you select correct region, as of right now, accounts can’t be moved between regions

![register form select region](images/2023-01-23-nextcloud-b2/nextcloud-b2-1.png)

Next, after logging in, click on Buckets → Create a Bucket

![create bucket 1](images/2023-01-23-nextcloud-b2/nextcloud-b2-2.png)

Enter unique bucket name, and set “Files in Bucket are” “Private”, as we don’t want to access our files publicly from URL.

![create bucket 2](images/2023-01-23-nextcloud-b2/nextcloud-b2-3.png)

Click on “Create a Bucket”

## Creating application keys in Backblaze

Next, navigate to Account → App Keys

![create key 1](images/2023-01-23-nextcloud-b2/nextcloud-b2-4.png)

On Application Keys page, click on “Add a New Application Key”

![create key 2](images/2023-01-23-nextcloud-b2/nextcloud-b2-5.png)

In popup, enter name of key, allow access only to the bucket we created, and allow list all bucket names for S3 compatibility.

![create key 3](images/2023-01-23-nextcloud-b2/nextcloud-b2-6.png)

After clicking on Create New Key, you will see your new keyID and applicationKey. Note - it will be displayed only once.

![create key 4](images/2023-01-23-nextcloud-b2/nextcloud-b2-7.png)

## Configuring Nextcloud

Go to `config/config.php`file on your Nextcloud server and to `$CONFIG` array add 

```jsx
'objectstore' => [
    'class' => '\\OC\\Files\\ObjectStore\\S3',
    'arguments' => [
            'bucket' => 'nextcloud-testing',
            'autocreate' => false,
            'key'    => '004e8eebeeeef000000000f',
            'secret' => 'K00400000000000000000000/zU',
            'hostname' => 's3.us-west-004.backblazeb2.com',
            'port' => 443,
            'use_ssl' => true,
            'region' => 'auto',
            'use_path_style'=>true
    ],
```

Replace `nextcloud-testing` with the name of your bucket. Hostname should be the endpoint of bucket. 

![nextcloud endpoint config](images/2023-01-23-nextcloud-b2/nextcloud-b2-8.png)

`config.php` should look something like that

```jsx
<?php
$CONFIG = array (
  'instanceid' => '...',
  'passwordsalt' => '...',
  'secret' => '...',
  'trusted_domains' => 
  array (
    0 => 'nc.issei.space',
  ),
  'datadirectory' => '/home/issei/web/nc.issei.space/public_html/data',
  'dbtype' => 'mysql',
  'version' => '25.0.3.2',
  'overwrite.cli.url' => 'https://nc.issei.space',
  'dbname' => 'issei_ncs3',
  'dbhost' => 'localhost',
  'dbport' => '',
  'dbtableprefix' => 'oc_',
  'mysql.utf8mb4' => true,
  'dbuser' => 'issei_ncs3',
  'dbpassword' => '...',
  'installed' => true,
  'objectstore' => [
    'class' => '\\OC\\Files\\ObjectStore\\S3',
    'arguments' => [
            'bucket' => 'nextcloud-testing',
            'autocreate' => false,
            'key'    => '004e8eebeeeef000000000f',
            'secret' => 'K00400000000000000000000/zU',
            'hostname' => 's3.us-west-004.backblazeb2.com',
            'port' => 443,
            'use_ssl' => true,
            'region' => 'auto',
            // required for some non Amazon S3 implementations
            'use_path_style'=>true
    ],
],
);
```

Save the file and visit Nextcloud. 

Note: if you uploaded files prior to setting object store you will no longer see them (but they still exist on local filesystem).

Try to upload a file, if it succeeds, congratulation, your Nextcloud instance will now use B2 as primary storage!

## Basic troubleshooting guide

1. Add to config array `'debug' => true,` It will print any errors.
2. Try to access bucket with awscli or WinSCP on your local machine.