<p align="center">
		<img src=".github/readme-header.png" width="1280" alt="Header Image">
</p>
<p align="center">
	<a href="https://github.com/serversideup/github-action-docker-swarm-deploy/blob/main/LICENSE" target="_blank"><img src="https://badgen.net/github/license/serversideup/github-action-docker-swarm-deploy" alt="License"></a>
	<a href="https://github.com/sponsors/serversideup"><img src="https://badgen.net/badge/icon/Support%20Us?label=GitHub%20Sponsors&color=orange" alt="Support us"></a>
  <br />
  <a href="https://community.serversideup.net"><img alt="Discourse users" src="https://img.shields.io/discourse/users?color=blue&server=https%3A%2F%2Fcommunity.serversideup.net"></a>
  <a href="https://serversideup.net/discord"><img alt="Discord" src="https://img.shields.io/discord/910287105714954251?color=blueviolet"></a>
</p>

# Docker Swarm Deploy with GitHub Actions
Hi! We're [Dan](https://twitter.com/danpastori) and [Jay](https://twitter.com/jaydrogers). We're a two person team with a passion for open source products. We created [Server Side Up](https://serversideup.net) to help share what we learn.

### Find us at:

* 📖 [Blog](https://serversideup.net) - get the latest guides and free courses on all things web/mobile development.
* 🙋 [Community](https://community.serversideup.net) - get friendly help from our community members.
* 🤵‍♂️ [Get Professional Help](https://serversideup.net/get-help) - get guaranteed responses within next business day.
* 💻 [GitHub](https://github.com/serversideup) - check out our other open source projects
* 📫 [Newsletter](https://serversideup.net/subscribe) - skip the algorithms and get quality content right to your inbox
* 🐥 [Twitter](https://twitter.com/serversideup) - you can also follow [Dan](https://twitter.com/danpastori) and [Jay](https://twitter.com/jaydrogers)
* ❤️ [Sponsor Us](https://github.com/sponsors/serversideup) - please consider sponsoring us so we can create more helpful resources

### Our Sponsors
All of our software is free an open to the world. None of this can be brought to you without the financial backing of our sponsors.

#### Individual Supporters
<!-- supporters --><a href="https://github.com/deligoez"><img src="https://github.com/deligoez.png" width="40px" alt="deligoez" /></a>&nbsp;&nbsp;<a href="https://github.com/alexjustesen"><img src="https://github.com/alexjustesen.png" width="40px" alt="alexjustesen" /></a>&nbsp;&nbsp;<a href="https://github.com/jeremykenedy"><img src="https://github.com/jeremykenedy.png" width="40px" alt="jeremykenedy" /></a>&nbsp;&nbsp;<!-- supporters -->

# About this project
This is a GitHub Action intended to simplify the development experience while deploying with Docker Swarm.

### Features:
- 😃 Simple to use
- 🏗️ Bring your own container + configuration
- 💯 Replicate 100% of production from Development to CI to Deployment

# Usage
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
      - uses: serversideup/github-action-docker-swarm-deploy@v2
        with:
          ssh_deploy_private_key: "${{ secrets.SSH_DEPLOY_PRIVATE_KEY }}"
          ssh_remote_hostname: "${{ secrets.SSH_REMOTE_HOSTNAME }}"
          registry: "ghcr.io"
          registry-username: "${{ github.actor }}"
          registry-token: "${{ secrets.GITHUB_TOKEN }}"
          stack_name: "${{ env.PROJECT_NAME }}"
          md5_file_path: "./.infrastructure/conf/traefik/prod/traefik.yml"
          md5_variable_name: "SPIN_MD5_HASH_TRAEFIK_YML"
```
### Configuration options

| Parameter               | Description                                                                                     | Default                                                | Required                                |
|-------------------------|-------------------------------------------------------------------------------------------------|--------------------------------------------------------|----------------------------------------|
| docker_compose_file_path| Set your docker compose file path with the CLI options.                                         | `-c docker-compose.yml -c docker-compose.prod.yml`     | false                                  |
| md5_file_path           | Set the path to the file you would like to get the MD5 checksum for.                            | `./path/to/my/file.txt`                                | false                                  |
| md5_variable_name       | The name of the variable you would like to store the MD5 checksum in.                           | `MD5_CHECKSUM`                                         | (only if `md5_file_path` is provided)  |
| registry                | Comma-separated list of container registries to authenticate with (e.g., "docker.io,ghcr.io").  | `docker.io` (Default to Docker Hub if not specified)   | false                                  |
| registry-token          | The token or password to use to authenticate with the container registry.                       |                                                        | true                                   |
| registry-username       | The username to use to authenticate with the container registry.                                |                                                        | true                                   |
| ssh_deploy_private_key  | The private key you have authenticated to connect to your server via SSH.                       |                                                        | ⚠️ true                                |
| ssh_deploy_user         | The user that you would like to connect as on the remote server via SSH.                        | `deploy`                                               | ⚠️ true                                |
| ssh_remote_hostname     | The hostname or IP address of the server you want to connect to.                                |                                                        | ⚠️ true                                |
| ssh_remote_known_hosts  | The public key of your SSH server to validate we are connecting to the right server.            |                                                        | ⚠️ true                                |
| ssh_remote_port         | The SSH port of the remote server you would like to connect to.                                 | `22`                                                   | false                                  |
| stack_name              | The name of your docker stack.                                                                  |                                                        | true                                   |

### Getting the MD5 Checksum of a file
We include an optional input to get the MD5 checksum of a file. This is useful if you're working with Docker Configs and you only want the service to update if the file has changed. You just need to set the following inputs (a full example is available at the top of this document):

```yml
steps:
  - uses: serversideup/github-action-docker-swarm-deploy@v2
    with:
      md5_file_path: "./path/to/my/file.txt"
      md5_variable_name: "MY_FILE_MD5"
```

This will store the MD5 checksum of the file at `./path/to/my/file.txt` in the environment variable `MY_FILE_MD5`.

### Security Disclosures
If you find a security vulnerability, please let us know as soon as possible.

[View Our Responsible Disclosure Policy →](https://www.notion.so/Responsible-Disclosure-Policy-421a6a3be1714d388ebbadba7eebbdc8)