<p align="center">
		<img src=".github/img/readme-header.png" width="1280" alt="Header Image">
</p>
<p align="center">
	<a href="https://github.com/serversideup/github-action-docker-swarm-deploy/blob/main/LICENSE" target="_blank"><img src="https://badgen.net/github/license/serversideup/github-action-docker-swarm-deploy" alt="License"></a>
	<a href="https://github.com/sponsors/serversideup"><img src="https://badgen.net/badge/icon/Support%20Us?label=GitHub%20Sponsors&color=orange" alt="Support us"></a>
  <br />
  <a href="https://community.serversideup.net"><img alt="Discourse users" src="https://img.shields.io/discourse/users?color=blue&server=https%3A%2F%2Fcommunity.serversideup.net"></a>
  <a href="https://serversideup.net/discord"><img alt="Discord" src="https://img.shields.io/discord/910287105714954251?color=blueviolet"></a>
</p>

## Introduction
This is a GitHub Action intended to simplify the deployment experience with GitHub Actions + Docker Swarm.

### Features:
- 😃 Simple to use
- 🏗️ Bring your own container + configuration
- 💯 Replicate 100% of production from Development to CI to Deployment
- 💪 Use with self-hosted registries
- 🧮 Store MD5 hashes in environment variables for deployment
- 🔐 Use with private registries
- 🏠 Use .env files for deployment

## Usage
Here is an example workflow:

```yml
name: Production Deployment
on:
  push:
    branches:
      - main

jobs:
  deploy:
    needs: build
    runs-on: ubuntu-24.04
    steps:
      - uses: serversideup/github-action-docker-swarm-deploy@v3
        with:
          ssh_deploy_private_key: "${{ secrets.SSH_DEPLOY_PRIVATE_KEY }}"
          ssh_remote_hostname: "${{ secrets.SSH_REMOTE_HOSTNAME }}"
          registry: "ghcr.io"
          registry-username: "${{ github.actor }}"
          registry-token: "${{ secrets.GITHUB_TOKEN }}"
          stack_name: "${{ env.PROJECT_NAME }}"
          md5_file_path: "./.infrastructure/conf/traefik/prod/traefik.yml"
          md5_variable_name: "SPIN_MD5_HASH_TRAEFIK_YML"
          env_file_base64: "${{ secrets.ENV_FILE_BASE64 }}"
```
## How to use the action

The following inputs are available:

| Parameter               | Description                                                                                     | Default                                              | Required |
|-------------------------|--------------------------------------------------------------------------------------------------|------------------------------------------------------|----------|
| docker_compose_file_path| Set your docker compose file path with the CLI options.                                          | `-c docker-compose.yml -c docker-compose.prod.yml`   | false    |
| env_file_base64         | The base64 encoded .env file to load into the container.                                         |                                                      | false    |
| log_level               | The log level to use for the Docker CLI.                                                         | `debug`                                              | false    |
| md5_file_path           | Set the path to the file you would like to get the MD5 checksum for.                             |                                                      | false    |
| md5_variable_name       | Set the name of the variable to store the MD5 checksum in.                                       | `MD5_CHECKSUM`                                       | false    |
| registry                | Comma-separated list of container registries to authenticate with (e.g., "docker.io,ghcr.io").   | `docker.io`                                          | false    |
| registry-token          | The token or password to use to authenticate with the container registry.                        |                                                      | ⚠️ true  |
| registry-username       | The username to use to authenticate with the container registry.                                 |                                                      | ⚠️ true  |
| ssh_deploy_private_key  | The private key you have authenticated to connect to your server via SSH.                        |                                                      | ⚠️ true  |
| ssh_deploy_user         | The user that you would like to connect as on the remote server via SSH.                         | `deploy`                                             | ⚠️ true  |
| ssh_remote_hostname     | The hostname or IP address of the server you want to connect to.                                 |                                                      | ⚠️ true  |
| ssh_remote_known_hosts  | The public key of your SSH server to validate we are connecting to the right server.             |                                                      | false    |
| ssh_remote_port         | The SSH port of the remote server you would like to connect to.                                  | `22`                                                 | false    |
| stack_name              | The name of your Docker stack.                                                                   |                                                      | ⚠️ true  |

### Removing the "ssh_remote_known_hosts" warning
![image](.github/img/known-hosts-warning.png)
For simplicity sake, we will automatically scan the known public SSH keys of your server and attempt to make a connection. The problem with this is it opens you up to a man-in-the-middle attack.

To ensure you're validating the identity of your server, you can set the `ssh_remote_known_hosts` input with the public key of your server. You can set this value to a GitHub secret like `SSH_REMOTE_KNOWN_HOSTS`:

```yml
- uses: serversideup/github-action-docker-swarm-deploy@v3
  with:
    registry-token: "${{ secrets.GITHUB_TOKEN }}"
    registry-username: "${{ github.actor }}"
    ssh_deploy_private_key: "${{ secrets.SSH_DEPLOY_PRIVATE_KEY }}"
    ssh_remote_hostname: "${{ secrets.SSH_REMOTE_HOSTNAME }}"
    ssh_remote_known_hosts: "${{ secrets.SSH_REMOTE_KNOWN_HOSTS }}"
    stack_name: "${{ env.PROJECT_NAME }}"
  env:
    SPIN_IMAGE_DOCKERFILE_PHP: "ghcr.io/${{ github.repository }}:${{ github.sha }}"
    SPIN_DEPLOYMENT_ENVIRONMENT: production
```
#### Setting the "ssh_remote_known_hosts" secret
![image](.github/img/known-hosts-warning.png)

You can set the `ssh_remote_known_hosts` secret by getting the public key of your server and setting it as a GitHub secret. You can run this from your **local machine** (not in GitHub Actions) to get the public key of your server:

```bash
ssh-keyscan -p 22 -H myserver.example.com 2>/dev/null | sort -u
```

Replace `myserver.example.com` with the hostname of your server. You can also change the port by changing `-p 22` to your desired port.

The output will look similar to this, with hashed hostnames:

```
|1|BfvcToAPMeAK0zR9FShCnP5CCaw=|ltUazkjjoIKsoBFQMF5yOTJt/Ks= ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBKIAz4U9GvgyBttgCnvi4AfBq3CdQ9XqAryrIyO1O60
|1|J/BpMKspk0BwPAxR28Dzc7gVGgw=|RAimV4/7iS4jlmFmDAfex/nKDUQ= ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQChvNZNpcjdSXJWVdnhieQXRgBVUUwpexLz0dbDegUj68vrzXsgtrnGtf+sJlRhI6C7jBZDfxk2jXL1ASfxEQUqbvptZTG68uusD1DYx3wtb/kTqvJ3JkFuWJbt2zLyZktPrueHA9cvuquW46M6wSZN5AZddNitUZ09Bpb+dTVZkjbEDOiGoHRDj5M86e1rr/8UGNrAVZl/hckup3lfu3B3P0LKnGnMw+/DXIKvJiwVJ3OdHzyq6D/x9uNgcOUA7UPgUbV30gyFtWr2Az6Vn/ZolDOGasK9iI5WjvBdXwyWNwEnnR539RutiwbS/XTnb0Jj/fFS5NM2/AM3nCT37D4uQA7aJFka7keUTJZJIVanziz9Ty76lloweLDKHN2CyvUijjSx5HaqV9Dr2nTefTPPvzz1D9xU0WJX8KC77Wcu8qEjqSwNihJqucXQvq4xeBZ85OGPbvzAFYqZdjynzVsLP50E7kmdaW3VJx88hbg+vyXrJD1urcOVPNtGoMpN2Mc=
|1|cYUT42KbDx+rQD0YSpKgDxbFWBc=|D13n4gWdMyJ+C2nifoEEeFmezmE= ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBN7wgADkhTHi7WER2pCZ5/10HBmhSIAq9zS1rWiBG5A8t2ATh5QnJ17XtPKXEJGPH8nogry/bZ+WKxI4zojGD+Q=
```

Copy the output and set it as a GitHub secret (usually called `SSH_REMOTE_KNOWN_HOSTS`).

![image](.github/img/secrets.png)

#### Validate the known hosts file
If you need to validate the known hosts file, you can save it in a file on your local machine and attempt to SSH into your server with it:

```bash
ssh -p 22 -i /path/to/test_known_hosts_file myserver.example.com
```

If you cannot connect from your local machine, then you know there is an issue with the known hosts file itself.


### Getting the MD5 Checksum of a file
We include an optional input to get the MD5 checksum of a file. This is useful if you're working with Docker Configs and you only want the service to update if the file has changed. You just need to set the following inputs (a full example is available at the top of this document):

```yml
steps:
  - uses: serversideup/github-action-docker-swarm-deploy@v3
    with:
      md5_file_path: "./path/to/my/file.txt"
      md5_variable_name: "MY_FILE_MD5"
```

This will store the MD5 checksum of the file at `./path/to/my/file.txt` in the environment variable `MY_FILE_MD5`.

### Using an .env file
You can use an .env file to set environment variables for your container. This is useful if you're working with environment variables that are different for each environment. You can set the .env file as a base64 encoded string and it will be decoded and loaded into the container.

```yml
steps:
  - uses: serversideup/github-action-docker-swarm-deploy@v3
    with:
      env_file_base64: "${{ secrets.ENV_FILE_BASE64 }}"
```

To set the value of `ENV_FILE_BASE64`, you can use the following command:

```bash
cat .env | base64
```

Any variable set in the .env file will be available to the deployment to be used in your docker-compose.yml file.

For example, if you have a .env file with the following:

```
DB_HOST=mysql
DB_PORT=3306
DB_USER=root
DB_PASSWORD=password
```

Then you can use the following in your docker-compose.yml file: 

```yml
services:
  mysql:
    environment:
      - DB_HOST=${{ env.DB_HOST }}
      - DB_PORT=${{ env.DB_PORT }}
      - DB_USER=${{ env.DB_USER }}
      - DB_PASSWORD=${{ env.DB_PASSWORD }}
```

## Resources
- **[Discord](https://serversideup.net/discord)** for friendly support from the community and the team.
- **[GitHub](https://github.com/serversideup/github-action-docker-swarm-deploy)** for source code, bug reports, and project management.
- **[Get Professional Help](https://serversideup.net/professional-support)** - Get video + screen-sharing help directly from the core contributors.

## Contributing
As an open-source project, we strive for transparency and collaboration in our development process. We greatly appreciate any contributions members of our community can provide. Whether you're fixing bugs, proposing features, improving documentation, or spreading awareness - your involvement strengthens the project. Please review our [contribution guidelines](https://serversideup.net/open-source/github-action-docker-swarm-deploy/docs/getting-started/contributing) and [code of conduct](./.github/code_of_conduct.md) to understand how we work together respectfully.

- **Bug Report**: If you're experiencing an issue while using this action, please [create an issue](https://github.com/serversideup/github-action-docker-swarm-deploy/issues/new/choose).
- **Feature Request**: Make this project better by [submitting a feature request](https://github.com/serversideup/github-action-docker-swarm-deploy/issues/new/choose).
- **Documentation**: Improve our documentation by [submitting a documentation change](./README.md).
- **Community Support**: Help others on [Discord](https://serversideup.net/discord).
- **Security Report**: Report critical security issues via [our responsible disclosure policy](https://www.notion.so/Responsible-Disclosure-Policy-421a6a3be1714d388ebbadba7eebbdc8).

Need help getting started? Join our Discord community and we'll help you out!

<a href="https://serversideup.net/discord"><img src="https://serversideup.net/wp-content/themes/serversideup/images/open-source/join-discord.svg" title="Join Discord"></a>

## Our Sponsors
All of our software is free an open to the world. None of this can be brought to you without the financial backing of our sponsors.

<p align="center"><a href="https://github.com/sponsors/serversideup"><img src="https://521public.s3.amazonaws.com/serversideup/sponsors/sponsor-box.png" alt="Sponsors"></a></p>

#### Bronze Sponsors
<!-- bronze -->No bronze sponsors yet. <a href="https://github.com/sponsors/serversideup">Become a sponsor →</a><!-- bronze -->

#### Individual Supporters
<!-- supporters --><a href="https://github.com/GeekDougle"><img src="https://github.com/GeekDougle.png" width="40px" alt="GeekDougle" /></a>&nbsp;&nbsp;<a href="https://github.com/JQuilty"><img src="https://github.com/JQuilty.png" width="40px" alt="JQuilty" /></a>&nbsp;&nbsp;<!-- supporters -->

## About Us
We're [Dan](https://twitter.com/danpastori) and [Jay](https://twitter.com/jaydrogers) - a two person team with a passion for open source products. We created [Server Side Up](https://serversideup.net) to help share what we learn.

<div align="center">

| <div align="center">Dan Pastori</div>                  | <div align="center">Jay Rogers</div>                                 |
| ----------------------------- | ------------------------------------------ |
| <div align="center"><a href="https://twitter.com/danpastori"><img src="https://serversideup.net/wp-content/uploads/2023/08/dan.jpg" title="Dan Pastori" width="150px"></a><br /><a href="https://twitter.com/danpastori"><img src="https://serversideup.net/wp-content/themes/serversideup/images/open-source/twitter.svg" title="Twitter" width="24px"></a><a href="https://github.com/danpastori"><img src="https://serversideup.net/wp-content/themes/serversideup/images/open-source/github.svg" title="GitHub" width="24px"></a></div>                        | <div align="center"><a href="https://twitter.com/jaydrogers"><img src="https://serversideup.net/wp-content/uploads/2023/08/jay.jpg" title="Jay Rogers" width="150px"></a><br /><a href="https://twitter.com/jaydrogers"><img src="https://serversideup.net/wp-content/themes/serversideup/images/open-source/twitter.svg" title="Twitter" width="24px"></a><a href="https://github.com/jaydrogers"><img src="https://serversideup.net/wp-content/themes/serversideup/images/open-source/github.svg" title="GitHub" width="24px"></a></div>                                       |

</div>

### Find us at:

* **📖 [Blog](https://serversideup.net)** - Get the latest guides and free courses on all things web/mobile development.
* **🙋 [Community](https://community.serversideup.net)** - Get friendly help from our community members.
* **🤵‍♂️ [Get Professional Help](https://serversideup.net/professional-support)** - Get video + screen-sharing support from the core contributors.
* **💻 [GitHub](https://github.com/serversideup)** - Check out our other open source projects.
* **📫 [Newsletter](https://serversideup.net/subscribe)** - Skip the algorithms and get quality content right to your inbox.
* **🐥 [Twitter](https://twitter.com/serversideup)** - You can also follow [Dan](https://twitter.com/danpastori) and [Jay](https://twitter.com/jaydrogers).
* **❤️ [Sponsor Us](https://github.com/sponsors/serversideup)** - Please consider sponsoring us so we can create more helpful resources.

## Our products
If you appreciate this project, be sure to check out our other projects.

### ⚡️ Starter Kits
- **[Spin Pro](https://getspin.pro)**: Laravel Sail alternative for running Docker from development → production.

### 📚 Books
- **[The Ultimate Guide to Building APIs & SPAs](https://serversideup.net/ultimate-guide-to-building-apis-and-spas-with-laravel-and-nuxt3/)**: Build web & mobile apps from the same codebase.
- **[Building Multi-Platform Browser Extensions](https://serversideup.net/building-multi-platform-browser-extensions/)**: Ship extensions to all browsers from the same codebase.

### 🛠️ Software-as-a-Service
- **[Bugflow](https://bugflow.io/)**: Get visual bug reports directly in GitHub, GitLab, and more.
- **[SelfHost Pro](https://selfhostpro.com/)**: Connect Stripe or Lemonsqueezy to a private docker registry for self-hosted apps.

### 🌍 Open Source
- **[AmplitudeJS](https://521dimensions.com/open-source/amplitudejs)**: Open-source HTML5 & JavaScript Web Audio Library.
- **[Spin](https://serversideup.net/open-source/spin/)**: Laravel Sail alternative for running Docker from development → production.
- **[Financial Freedom](https://github.com/serversideup/financial-freedom)**: Open source alternative to Mint, YNAB, & Monarch Money.