+++
title = "Github Actions + Hugo"
description = "Github Actions setup for self-hosted Hugo build and deploy"
author = 'viticlin'
date = 2024-01-14T07:07:07+01:00
tags = ["hugo, github actions, self-hosting, deploy"]
draft = true
summary = "Generating static Hugo site with Github Actions and deploying the boring way with scp"
+++

1. create ssh key

```bash
$ ssh-keygen -t ed25519 -f github_actions
```

It will ask for a passphrase, use one or leave it blank.

2. create new user in your server to perform the github action

To lock what Github Actions can do on your server, the best thing to do is only give it permisions to the folder where you publish your page.

In order to do this, create a new user

```
-- creating new user in server
useradd --system --create-home --home "/home/github_actions" --comment "Github Actions runner" -U github_actions
```

add key to authorized_keys. this can be done using ssh-copy

change permissions to the folder you will use

3. ssh-copy github key
3. permission to folder
4. github action
