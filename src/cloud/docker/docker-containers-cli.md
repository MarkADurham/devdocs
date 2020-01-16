---
group: cloud-guide
title: Docker CLI Containers
functional_areas:
  - Cloud
  - Docker
  - Configuration
---

The CLI containers provide `magento-cloud` and `{{site.data.var.ct}}` commands to perform file system operations and to interact with the application.

The following CLI containers, most of which are based on a [PHP-CLI version 7 image], provide `magento-cloud` and `{{site.data.var.ct}}` commands to perform file system operations:

-  `build`—extends the CLI container to perform operations with writable filesystem, similar to the build phase
-  `deploy`—extends the CLI container to use read-only file system, similar to the deploy phase
-  `cron`—extends the CLI container to run cron
-  `node`—based on node image, used for NPM activities

For example, you can check the state of the your project using the _ideal-state_ wizard:

Run the `{{site.data.var.ct}}` ideal-state command.

```bash
docker-compose run deploy ece-command wizard:ideal-state
```

{:.procedure}
Sample response:

```terminal
 - Your application does not have the "post_deploy" hook enabled.
The configured state is not ideal
```
{:.no-copy}

All build and deploy processes are defined and configured using {{site.data.var.ct}} and the Magento Cloud template.

## CLI container commands

These commands are available in any of the containers.

| Command    | Target Containers   |  Notes
| ------------- |  ------------------ |------------------
| cloud-build | build | Used to build the application in production mode, configured via build hook in .magento.app.yml
| cloud-deploy | deploy | Used to deploy the application, configured via deploy hook in .magento.app.yml
| cloud-post-deploy | deploy | Used to deploy the application, configured via deploy hook in .magento.app.yml
| ece-command | deploy | Used to run other commands from ece-tools CLI Tool
| magento-command | deploy | Used to run bin/magento commands
| magento-installer | deploy | Just runs build and then deploy hooks
| mftf-command | deploy | Used to run MFTF command for testing
| run-cron | cron | Used to run cron jobs

To understand the processing for each command, review the scripts in the [Magento Cloud Docker GitHub repository][scripts].

## Build container

Container name |Docker base image
-------- | -------- |
build | [magento/magento-cloud-docker-php], which is based on the Docker [php] image

The build container mimics the behavior of the Magento Cloud build process so that testing the build  and deploy process is as close to testing in production as possible.

You can also run build commands manually from the build container to perform individual steps from the build process. For example, you can run the following command to deploy static content.

```bash
docker-compose run build magento-command setup:static-content:deploy
```

## Cron container

Container name |Docker base image
-------- | -------- |
cron | [magento/magento-cloud-docker-php], which is based on the Docker [php] image

The Cron container is based on PHP-CLI images, and executes operations in the background immediately after the Docker environment starts. This container uses the cron configuration defined in the [`crons` property of the `.magento.app.yaml` file]({{ site.baseurl }}/cloud/project/project-conf-files_magento-app.html#crons). This container has no custom configuration.

The cron container runs the scheduled cron jobs automatically. You can also use it to run cron jobs manually.

{:.procedure}
To view the cron log:

```bash
docker-compose run deploy bash -c "cat /app/var/cron.log"
```

-  The `setup:cron:run` and `cron:update` commands are not available on Cloud and Docker for Cloud environments
-  Cron only works with the CLI container to run the `./bin/magento cron:run` command

{:.procedure}
To run cron jobs manually:

```bash
docker-compose run cron /usr/local/bin/php bin/magento cron:run
```

### Disable the cron container

If cron runs cause performance problems, you can prevent the cron container from running automatically by adding following snippet to your `docker-compose.override.yml` file.

```yaml
cron:
  entrypoint: "bash -c"
```

After disabling the cron container, use docker compose to run cron jobs manually.

## Deploy container

Container name |Docker base image
-------- | --------
deploy | [magento/magento-cloud-docker-php], which is based on the [php] Docker image

This deploy container mimics the Magento Cloud deploy process so that testing the build and deploy process is as close to testing in production as possible.

You can also run build and deploy commands manually from the deploy container. For example, the following command reindexes the Magento store:

```bash
docker-compose run build magento-command index:reindex
```

## Node container

Container name |Docker base image
-------- | --------
node | [node]

The Node container is based on the [official Node Docker image][node]. You can use the container to install NPM dependencies, such as Gulp, or run any Node-based command line tools.

[PHP-CLI version 7 image]: https://hub.docker.com/r/magento/magento-cloud-docker-php
[magento/magento-cloud-docker-php]: https://hub.docker.com/r/magento/magento-cloud-docker-php
[scipts]: https://github.com/magento/magento-cloud-docker/tree/develop/images/php/cli/bin
[Cloud Docker scripts]: https://github.com/magento/magento-cloud-docker/tree/develop/images/php/cli/bin
[magento/magento-cloud-docker-php]: https://hub.docker.com/r/magento/magento-cloud-docker-php
[php]: https://hub.docker.com/_/php
[node]: https://hub.docker.com/_/node
