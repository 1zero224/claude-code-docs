---
title: 升级自托管 Claude Managed Agents
url: https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade
description: 升级 Kubernetes 或 Docker Compose 上的自托管 Claude Managed Agents 部署，并在新发布版本无法正常工作时回滚。
---

<Note>
  如需申请自托管 Claude Managed Agents 的访问权限，请联系您的 Anthropic 客户团队。如果您所属的 Claude Console 组织已获得访问权限，请[登录 Claude Console](https://platform.claude.com)并切换到该组织。然后重新加载此页面。
</Note>

本页面介绍如何在 Kubernetes 或 Docker Compose 上升级部署，以及如何回滚。在 Kubernetes 上，停机时间取决于升级跨越的幅度：

| 升级 | 是否需要先备份数据库 | Kubernetes 上的停机时间 |
| ----- | ----- | ----- |
| 补丁：仅第三位数字变化 | 否。数据库 schema 保持不变。 | 无 |
| 次要：第二位数字加一，例如从 `1.2.0` 到 `1.3.0` | 是。schema 会发生变化。 | 无，除非 `CHANGELOG.md` 的某个部分说明必须停止服务器 |
| 跳过一个次要发布版本，例如从 `1.1.0` 到 `1.3.0`，或第一位数字变化 | 是。schema 会发生变化。 | 从您停止旧服务器的那一刻起，直到新服务器就绪为止 |

对于各种升级，Docker Compose 栈在重启期间都处于停机状态。

## 升级之前

阅读解压后归档顶层目录中 `CHANGELOG.md` 里新发布版本的部分。该部分开头列出了特定于该发布版本的升级步骤。如果您跳过了某些发布版本，请按从旧到新的顺序，遵循比您当前版本更新的每一个部分。

然后查看[需要检查的设置和路径](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#settings-and-paths-to-check)。

<Warning>
  在 Kubernetes 上，升级可能会删除某些会话的文件，且不会有任何错误报告。升级之前，请参阅[Kubernetes：已删除会话的卷](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#deleted-sessions-volumes)。
</Warning>

次要或主要升级包括一次数据库备份。转储文件以明文形式保存所有对话，因此本页面上的命令会使用 [age 加密工具](https://age-encryption.org)对其进行加密。只需创建一次密钥对，并将私钥 `cma-backup.agekey` 与备份分开存放：

```bash
age-keygen -o cma-backup.agekey
age-keygen -y cma-backup.agekey > cma-backup.agepub
```

## 在 Kubernetes 上升级

本页面上的 `kubectl` 命令会从一个 shell 变量中读取服务器 Deployment 的名称。在您运行这些命令的每个终端中运行以下两行。第一行查找该名称，第二行将其打印出来：

```bash
server=$(kubectl -n cma-server get deployment -l anthropic.com/role=server -o jsonpath='{.items[*].metadata.name}')
echo "$server"
```

如果第二行打印出 `cma-server`，请继续。如果打印出其他名称，请在升级之前阅读[您的对象名称](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#object-names)。如果没有打印任何内容，请检查 `kubectl` 是否指向正确的集群，然后再次运行这两行。

<Steps>
  <Step title="构建并推送新镜像">
    解压新归档。以新版本号构建并推送其中的三个镜像，具体参见[构建并推送镜像](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes#build-and-push-the-images)。如果您运行了 [egress gateway](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity#environment-variable-credentials)（出口网关），还需构建并推送该网关的镜像。

    如果您构建了自己的沙箱镜像，也请以新版本号重新构建并推送该镜像。将其 Dockerfile 与新归档中的 `kubernetes/sandbox/Dockerfile` 进行比较，并参阅[自有镜像需要满足的条件](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/call-the-api#what-an-image-of-your-own-needs)。
  </Step>

  <Step title="在 values 文件中设置新版本">
    如果某个 values 文件设置了 `image.tag`，请将其改为新版本号。
  </Step>

  <Step title="停止所有服务器（如果升级需要）">
    仅在以下情况下执行此步骤：

    * 您运行的某台服务器落后新发布版本超过一个次要发布版本。
    * 第一位数字（即主版本号）增加。
    * `CHANGELOG.md` 中比您运行的发布版本更新的某个部分说明必须停止服务器。

    从此步骤开始直到新服务器就绪为止，客户端都无法访问该部署，因此请规划一个维护窗口。

    ```bash
    kubectl -n cma-server scale deployment/$server --replicas=0
    kubectl -n cma-server wait --for=delete pod -l anthropic.com/role=server --timeout=90s
    ```
  </Step>

  <Step title="备份数据库">
    在进行次要或主要升级之前，或者如果您停止了服务器，请执行此操作。如果旧发布版本无法在已更改的 schema 上运行，此备份就是唯一的回退途径。

    请使用与您的数据库服务器相同 PostgreSQL 主版本的 `pg_dump`。对于 `<role>` 和 `<database>`，请使用 `cma-server-values.yaml` 中 `database.url` 的 `user` 和 `dbname` 值。`pg_dump` 会提示输入该角色的密码。

    ```bash
    set -o pipefail
    PGSSLMODE=require pg_dump -h <database-host> -U <role> -d <database> -Fc \
      | age -r "$(cat cma-backup.agepub)" -o cma_server-<old-version>.dump.age
    ```

    如果任一命令输出错误，则其留下的文件不是有效备份。请先修复原因并重新运行该命令，然后再继续。

    如果服务器正在运行，备份只包含转储开始前客户端写入的内容。
  </Step>

  <Step title="先升级服务器，再升级沙箱">
    在新归档的顶层目录中运行以下命令，并使用您安装时所用的 values 文件：

    ```bash
    helm upgrade cma-server kubernetes/server/chart --namespace cma-server -f cma-server-values.yaml --wait
    helm upgrade cma kubernetes/chart --namespace cma -f cma-values.yaml --wait
    ```

    如果您运行了多个环境，请为每个沙箱 release 运行第二条命令，并使用该 release 的名称、命名空间和 values 文件。

    如果您停止了服务器，第一条命令会迁移 schema、恢复服务器数量，并在新服务器就绪后返回。
  </Step>
</Steps>

在服务器运行的情况下，Helm 会在停止旧服务器之前先启动每台新服务器，因此就绪服务器的数量永远不会下降。在次要升级中，第一条命令会在旧服务器继续提供服务的同时更改 schema，然后替换这些旧服务器。正在被替换的服务器上打开的事件流会正常结束，因此跟随事件流的客户端需要重新连接。

如果某条命令失败，请参阅[如果升级失败](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#if-the-upgrade-fails)。

## 在 Docker Compose 上升级

对于各种升级，步骤都相同。请在发布版本的 Compose 目录（即存放 `docker-compose.yml` 的目录）中运行每条 `docker compose` 命令。在解压后的归档中，该目录为 `compose`。请使用未设置 `ANTHROPIC_BASE_URL`、`ANTHROPIC_API_KEY` 和 `ANTHROPIC_AUTH_TOKEN` 的终端，因为 Compose 会将 shell 中这些变量的值传递给服务器。

<Steps>
  <Step title="备份数据库">
    在进行次要或主要升级之前，请在旧发布版本的 Compose 目录中执行此操作：

    ```bash
    set -o pipefail
    docker compose exec -T postgres sh -c 'pg_dump -U "$POSTGRES_USER" -d "$POSTGRES_DB" -Fc' \
      | age -r "$(cat cma-backup.agepub)" -o cma_server-<old-version>.dump.age
    ```

    在该命令中，请保持 `$POSTGRES_USER` 和 `$POSTGRES_DB` 不变。单引号中的文本在 `postgres` 容器内运行，该容器在这些变量中保存其角色和数据库的名称。

    备份期间栈仍在提供服务，因此转储文件不包含转储开始后客户端写入的任何内容。如果任一命令输出错误，则其留下的文件不是有效备份。请先修复原因并重新运行该命令，然后再继续。
  </Step>

  <Step title="解压新归档">
    将新归档放在新发布版本可以长期存放的目录中，例如旧发布版本的父目录。这些命令会在该目录中解压归档，新栈将从解压后的文件运行。

    如果您的 umask 比 `022` 更严格，请先运行 `umask 022`。然后在旧发布版本的 Compose 目录中运行以下命令。将 `<directory>` 替换为存放新归档的目录的完整路径，将 `<archive>` 替换为归档名称（不含 `.tar.gz`）。

    ```bash
    releases="<directory>"
    tar -xzf "$releases/<archive>.tar.gz" -C "$releases" && cp .env compose.config.yaml "$releases/<archive>/compose/"
    ```

    第二行还会将您的 `.env` 和 `compose.config.yaml` 复制到解压后的 `compose` 目录中。在执行这两行期间，旧栈会持续运行。如果某一行输出错误，请修复原因并重新运行这两行。
  </Step>

  <Step title="重新进行您的其他更改">
    只有 `.env` 和 `compose.config.yaml` 会被沿用。请在新发布版本的 Compose 目录中重新进行您的其他更改：

    * **您添加的文件：** 复制该文件。
    * **您编辑过的随附文件：** 在新发布版本的副本中进行相同的编辑。不要用旧文件覆盖它，因为这会撤销该发布版本对该文件的更改。
    * **智能体身份：** 使用 `cp -a` 复制 `agent-identity` 目录（如果该目录归 root 所有，请使用 `sudo`），并再次编辑 `docker-compose.yml` 的 `volumes:` 块。
    * **您自己的 Compose 文件：** 复制该文件，并检查其中的相对路径。

    然后进行[需要检查的设置和路径](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#settings-and-paths-to-check)中的 Docker Compose 更改。
  </Step>

  <Step title="停止旧栈并启动新栈">
    请在您解压归档的终端中运行以下几行，因为它们会用到该终端中的 `$releases` 变量。保持在旧发布版本的 Compose 目录中，并将 `<archive>` 替换为相同的名称。如果您为旧栈导出了 `COMPOSE_FILE`，请先导出相同的值。

    ```bash
    ls "$releases/<archive>/compose/.env" && docker compose down
    cd "$releases/<archive>/compose" && docker compose up --build -d
    ```

    只有在新发布版本的 `.env` 就位时，第一行才会停止旧栈。`docker compose down` 会保留数据库。不要添加 `-v`，它会删除数据库。新栈会接管该数据库，其 `migrate` 容器会在服务器启动前更新 schema。
  </Step>
</Steps>

如果某条命令失败，请参阅[如果升级失败](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#if-the-upgrade-fails)。

## 如果升级失败

在回滚之前，请在日志中查找原因：

* **Kubernetes：** 如果迁移 Job 失败，`helm upgrade` 会在 Job 重试期间等待几分钟，然后失败。在此等待期间，请在第二个终端中运行 `kubectl -n cma-server logs job/$server-migrate`，并按照[在 Kubernetes 上升级](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#upgrade-on-kubernetes)中的方式设置 `$server`。Job 停止重试时，Kubernetes 会删除该日志。如果您错过了，请再次运行 `helm upgrade`，这会重新启动该 Job。
* **Docker Compose：** `docker compose up --build -d` 的输出可能不会显示原因。请运行 `docker compose logs` 查找原因。

| 日志显示的内容 | 含义 | 处理方法 |
| ----- | ----- | ----- |
| 一行包含 `refusing to migrate: servers of another release are still running against this database` 的内容 | 某些升级需要先停止所有服务器，而旧发布版本的某台服务器仍在运行。迁移未做任何更改。 | 按照[在 Kubernetes 上升级](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#upgrade-on-kubernetes)中的方式停止所有服务器，然后再次运行 `helm upgrade`。如果某台服务器未正常关闭，请先等待 20 秒：在此之前，迁移仍会将该服务器视为正在运行。 |
| 重新部署旧发布版本时，一行包含 `cannot downgrade` 的内容 | 旧发布版本无法在已更改的 schema 上运行。 | 按照[回滚](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#roll-back)中的方式恢复备份。 |
| 迁移产生的其他错误 | 您无需回滚。在 Kubernetes 上，迁移成功之前，服务器保持原样。 | 修复原因后，再次运行 `helm upgrade` 或 `docker compose up --build -d`。 |

在 Docker Compose 上，在新发布版本的目录中运行 `docker compose up` 可能会停止并报出 `required variable CMA_PG_USER is missing a value`。这说明新发布版本的 `.env` 缺少保存您的栈数据所用的名称：其 Compose 项目、其卷，以及其数据库角色和名称。新栈尚未启动，旧栈可能已停止，您的数据未受影响。不要手动向 `.env` 添加 `CMA_PG_USER` 行。仅添加该行时，栈会以该发布版本自己的名称启动，且不含任何数据。您的数据随后仍留在旧卷中。请改为在旧发布版本的 Compose 目录中运行 `docker compose up -d`。然后在该目录中、在含有 `$releases` 变量的终端中运行以下这一行。将 `<archive>` 替换为"解压新归档"步骤中的归档名称：

```bash
sh "$releases/<archive>/compose/keep-names.sh" && cp .env "$releases/<archive>/compose/"
```

重新进行您对新发布版本的 `.env` 所做的任何更改。然后继续执行"停止旧栈并启动新栈"步骤。

迁移可能成功，但新服务器未能就绪。要查找原因，请阅读服务器的日志：

* **Kubernetes：** `kubectl -n cma-server logs -l anthropic.com/role=server --prefix --tail=20`
* **Docker Compose：** `docker compose logs cma-server`

例如，在 Kubernetes 上，包含 `has invalid keys` 的行会指出 `server.config` 下新发布版本不接受的某项设置。请修复原因并再次运行升级命令，然后再考虑恢复备份，因为恢复备份会丢失数据。在 Kubernetes 上，schema 更改后，旧发布版本中正在运行的服务器会继续提供服务，但如果它停止，可能无法再次启动。

## 回滚

本节中的 `kubectl` 命令使用[在 Kubernetes 上升级](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#upgrade-on-kubernetes)中的 `$server` 变量。在回滚之前，请查看[如果升级失败](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#if-the-upgrade-fails)。然后根据升级类型进行回滚：

* **补丁升级之后：** [重新部署旧发布版本](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#deploy-the-old-release-again)。两个发布版本使用相同的 schema。

* **次要升级或主要升级之后：** 打开新归档中的 `CHANGELOG.md`。阅读您要回退到的发布版本之后的每个发布版本的章节。

  * 如果某个章节说明旧发布版本无法在已迁移的数据库上启动，请恢复备份。
  * 否则，请重新部署旧发布版本。如果这一步因 `cannot downgrade` 而失败，请恢复备份。

<Warning>
  恢复会丢失备份开始后客户端写入的所有内容。
</Warning>

### 重新部署旧发布版本

* **Kubernetes：** 进入旧归档的顶层目录。使用该发布版本当时的 values 文件，运行[在 Kubernetes 上升级](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#upgrade-on-kubernetes)中的两条 `helm upgrade` 命令。例如，如果您为升级添加了 `earlierNames`，请将其移除。仅在第一条命令成功后才运行第二条命令。
* **Docker Compose：** 在新发布版本的 Compose 目录中运行 `docker compose down`，然后在旧发布版本的 Compose 目录中运行 `docker compose up --build -d`。

如果旧发布版本无法在已更改的 schema 上运行，日志会显示 `cannot downgrade`：

* **Kubernetes：** 迁移 Job 会失败，服务器保持原样。[如果升级失败](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#if-the-upgrade-fails)说明了如何读取该 Job 的日志。
* **Docker Compose：** 服务器不会启动。使用 `docker compose logs migrate` 读取日志。

在 Docker Compose 上，`docker compose logs` 的输出可能会改为显示 `stored value is encrypted but no encryption key is configured`。这种情况下，旧发布版本的 `.env` 中没有 `CMA_STORE__CREDENTIALS_ENCRYPTION_KEY` 这一行。请从新发布版本的 `.env` 中复制该行，然后再次运行 `docker compose up --build -d`。

### 在 Kubernetes 上恢复备份

您的数据库服务器需要有容纳数据库第二个副本的空间。对于 `<role>` 和 `<database>`，请使用 `cma-server-values.yaml` 中 `database.url` 的 `user` 和 `dbname` 值。

<Steps>
  <Step title="将备份恢复到新数据库">
    以管理员身份连接到数据库服务器上的 `postgres` 数据库，然后运行：

    ```sql
    CREATE DATABASE <database>_restored OWNER <role>;
    ```

    然后，在存放备份的机器上，使用您的密钥和备份文件的路径运行以下命令。请使用与您的数据库服务器相同主版本的 `pg_restore`。它会提示输入该角色的密码，成功时不打印任何内容。

    ```bash
    age -d -i cma-backup.agekey cma_server-<old-version>.dump.age \
      | PGSSLMODE=require pg_restore -h <database-host> -U <role> -d <database>_restored --no-owner --exit-on-error
    ```

    如果输出了错误，请停止。您的数据库没有变化。以管理员身份运行 `DROP DATABASE <database>_restored;`，然后重新开始此步骤。
  </Step>

  <Step title="停止所有服务器">
    ```bash
    kubectl -n cma-server scale deployment/$server --replicas=0
    kubectl -n cma-server wait --for=delete pod -l anthropic.com/role=server --timeout=90s
    ```
  </Step>

  <Step title="切换到已恢复的数据库">
    以管理员身份连接到 `postgres` 数据库，然后运行：

    ```sql
    ALTER DATABASE <database> RENAME TO <database>_upgraded;
    ALTER DATABASE <database>_restored RENAME TO <database>;
    ```

    如果第一条语句提示有其他会话正在使用该数据库，请关闭这些会话，然后再次运行这两条语句。连接池程序可能会持有此类会话。
  </Step>

  <Step title="部署旧发布版本">
    按照[重新部署旧发布版本](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#deploy-the-old-release-again)中的 Kubernetes 条目操作。第一条命令还会恢复服务器数量。如果该命令失败，但 `kubectl -n cma-server get pods` 显示旧服务器已就绪，请参阅[如果密钥检查失败](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#if-the-key-check-fails)。
  </Step>

  <Step title="删除您搁置的数据库">
    该数据库仍保存着客户端在备份之后写入的内容。旧发布版本正常工作后，使用 `DROP DATABASE <database>_upgraded;` 删除该数据库。
  </Step>
</Steps>

#### 如果密钥检查失败

在"部署旧发布版本"步骤中，第一条 `helm upgrade` 命令会等待旧服务器就绪，然后检查沙箱用于连接的密钥。如果该命令失败，但 `kubectl -n cma-server get pods` 显示旧服务器已就绪，则说明该检查失败了。

您的数据已恢复，服务器也能正常工作，但沙箱暂时还无法连接到服务器。沙箱使用其自身命名空间 `cma` 中 `cma-environment` Secret 里的密钥进行连接。该密钥比您的备份更新，因此已恢复的数据库无法识别它。

要确认原因，请再次运行第一条命令。在其等待期间，打开第二个终端并在其中也设置 `$server`。然后运行 `kubectl -n cma-server logs job/$server-environment-default`，将其中的 `default` 替换为您的环境名称。查找包含 `Secret cma/cma-environment records key` 的行。如果日志显示其他错误，请修复其所指出的问题。

如果日志显示了该行，请使用 `kubectl -n cma delete secret cma-environment` 删除该 Secret，然后再次运行第一条命令。该命令会使用新密钥重新创建该 Secret。然后运行第二条命令。

### 在 Docker Compose 上恢复备份

<Steps>
  <Step title="仅启动数据库">
    在新发布版本的 Compose 目录中运行 `docker compose down`。然后在旧发布版本的 Compose 目录中运行 `docker compose up -d --wait postgres`。在后续步骤中请保持在该目录中。
  </Step>

  <Step title="将备份恢复到新数据库">
    在旧发布版本的 Compose 目录中，将您的密钥和备份文件的路径代入后运行以下命令：

    ```bash
    docker compose exec -T postgres sh -c 'psql -U "$POSTGRES_USER" -d postgres \
        -c "DROP DATABASE IF EXISTS ${POSTGRES_DB}_restored" -c "CREATE DATABASE ${POSTGRES_DB}_restored"' \
      && age -d -i cma-backup.agekey cma_server-<old-version>.dump.age \
      | docker compose exec -T postgres sh -c 'pg_restore -U "$POSTGRES_USER" -d "${POSTGRES_DB}_restored" --no-owner --exit-on-error'
    ```

    如果输出了 `DROP DATABASE` 和 `CREATE DATABASE`，且没有错误，则表示成功。

    在此处及后续步骤中，请保持 `$POSTGRES_USER` 和 `$POSTGRES_DB` 不变。单引号中的文本在 `postgres` 容器内运行，该容器在这些变量中保存其角色和数据库的名称。

    如果输出了错误，您的数据库没有变化，因此您可以再次运行整条命令。如果无法恢复备份，请在新发布版本的 Compose 目录中运行 `docker compose up --build -d`，重新启动该发布版本。
  </Step>

  <Step title="切换到已恢复的数据库">
    ```bash
    docker compose exec -T postgres sh -c 'psql -U "$POSTGRES_USER" -d postgres \
        -c "ALTER DATABASE $POSTGRES_DB RENAME TO ${POSTGRES_DB}_upgraded" \
        -c "ALTER DATABASE ${POSTGRES_DB}_restored RENAME TO $POSTGRES_DB"'
    ```
  </Step>

  <Step title="启动旧发布版本">
    运行 `docker compose up --build -d`。
  </Step>

  <Step title="删除您搁置的数据库">
    该数据库仍保存着客户端在备份之后写入的内容。旧发布版本正常工作后，删除该数据库：

    ```bash
    docker compose exec -T postgres sh -c 'psql -U "$POSTGRES_USER" -d postgres -c "DROP DATABASE ${POSTGRES_DB}_upgraded"'
    ```
  </Step>
</Steps>

### 恢复之后

恢复会使环境密钥回到备份时的状态：

* **备份之后您吊销的密钥：** 该密钥会重新生效。请再次吊销。
* **备份之后您创建的密钥：** 该密钥会消失。请为使用该密钥的 worker 创建新密钥。

[为您自己的 worker 替换环境密钥](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes#replace-an-environment-key-for-your-own-workers)中提供了相关命令，这些命令也适用于 Docker Compose。Docker Compose 栈启动的沙箱无需任何操作，因为该栈会在启动时替换它们的密钥。

要列出某个环境的密钥（包括已吊销的密钥），请按以下方式运行 `list-keys`：

* 请使用您按照[在自托管服务器上调用 API](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/call-the-api) 设置的终端。不要使用运行 `docker compose` 的终端。
* 在 Kubernetes 上，请先在另一个终端中运行 `kubectl -n cma-server port-forward svc/$server 8400:8400`，并在该终端中也设置 `$server`。
* 在归档的顶层目录中为每个环境运行一次该命令，并使用该环境的 ID。

```bash
echo "$ANTHROPIC_API_KEY" | ./cma-server-admin list-keys \
  -base-url http://127.0.0.1:8400 -env env_01595EKxaaTTGwwY3kyXdtbs \
  -key-file - -all
```

### 测试备份

要验证转储文件能否恢复，请只运行"将备份恢复到新数据库"步骤。该步骤位于[在 Kubernetes 上恢复备份](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#restore-the-backup-on-kubernetes)和[在 Docker Compose 上恢复备份](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#restore-the-backup-on-docker-compose)中。该步骤不会更改正在使用的数据库。然后删除新数据库。在 Docker Compose 上，其名称为您的数据库名称末尾加上 `_restored`。

## 需要检查的设置和路径

请在运行升级命令之前阅读本节。

检查您的[设置文件](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#where-settings-go)中的 `web_tools` 部分。如果该文件没有此部分，或该部分未设置任何键，升级后 `web_search` 和 `web_fetch` 工具将处于关闭状态。此时，使用默认工具集的智能体会在没有这些工具的情况下运行，且不会报错。工具集启用了其中任一工具的智能体则无法启动会话。要保留这些工具，请参阅[Web 工具需要服务器设置](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences#web-tools-need-a-server-setting)。

在 Kubernetes 上，检查每个沙箱 values 文件（例如 `cma-values.yaml`）是否设置了 `egress.mode`。要保留沙箱对公共互联网的访问，请在 `egress` 下添加 `mode: publicInternet`。如果某个文件未设置模式，chart 会应用其默认模式 `closed`。随后，沙箱 chart 的 `helm upgrade` 命令会停止并报出 `egress.mode is closed and egress.closed.hosts is empty` 之类的错误，且不做任何更改。

同样检查这些文件中的 `sandbox.idleTimeout`。如果某个文件未设置该项，会话的 pod 会在智能体的回合结束约 5 分钟后被移除。智能体留下仍在运行的进程会随 pod 一起停止。默认情况下，`/workspace` 中的文件会保留。要将 pod 保留 30 分钟，请在 `sandbox` 下添加 `idleTimeout: 30m`。

检查设置文件中是否有 `inference.attribution`，以及服务器环境中是否有 `CMA_INFERENCE__ATTRIBUTION`。如果找到，请删除该行。您要升级到的发布版本没有此设置，只要设置了其中任一项，其服务器就不会启动。

当 Claude API 是您的推理提供商时，升级后每次模型调用都会携带其会话的 ID，以及该会话的智能体和环境的 ID。请参阅[服务器向其推理提供商发送的内容](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-data-and-access#what-the-server-sends-to-its-inference-provider)。

### Kubernetes：已删除会话的卷

升级可能会删除您已删除的每个会话的 `/workspace` 卷，以及不在服务器数据库中的每个会话（例如因恢复而丢失的会话）的该卷。您可能会丢失这些会话的文件，且不会有任何错误报告。一旦新发布版本的第一台服务器开始响应，删除就可能开始。对于服务器 chart，这发生在其 `helm upgrade` 期间。

请在旧服务器仍在运行时、在停止其中任何一台之前找出这些会话：运行[释放会话的卷](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes#free-a-sessions-volume)第 1 步中的命令。如果您运行了多个环境，请在每个沙箱 release 的命名空间中运行该命令。对于您需要其文件的会话，不要执行该部分的其他步骤。要保留这些文件，请使用您自己的 Kubernetes 工具将其复制出来。任何 API 调用、管理命令或您服务器上的控制台页面都无法访问这些卷。

### Kubernetes：您的对象名称

`cma-server-values.yaml` 必须在其 `earlierNames` 设置中说明您的安装使用哪些名称。要查找服务器 Deployment 的名称，请运行 `kubectl -n cma-server get deployment -l anthropic.com/role=server`。

| Deployment 的名称 | `cma-server-values.yaml` 中的行 |
| ----- | ----- |
| `cma-onprem` | `earlierNames: true` |
| `cma-server` | `earlierNames: false` |

如果该文件中没有这一行，请在升级之前添加，并在以后的每次升级中保留它。如果没有这一行，服务器 chart 的 `helm upgrade` 命令会停止并报出一个提及 `earlierNames` 的错误，且不做任何更改。

设置 `earlierNames: true` 时，chart 的对象会保留其 `cma-onprem` 名称，服务器仍能找到您以这些名称创建的 Secret。这些页面使用中间一列中的名称。设置 `earlierNames: true` 时，请改用最后一列中的名称：

| 对象 | 这些页面上的名称 | 设置 `earlierNames: true` 时的名称 |
| ----- | ----- | ----- |
| 服务器的 Deployment 和 Service | `cma-server` | `cma-onprem` |
| 迁移 Job | `cma-server-migrate` | `cma-onprem-migrate` |
| 每个环境的 Job | `cma-server-environment-<name>` | `cma-onprem-environment-<name>` |
| 包含数据库密码的 Secret | `cma-server-db` | `cma-onprem-db` |
| 包含推理提供商凭据的 Secret | `cma-server-upstream` | `cma-onprem-upstream` |
| 包含服务器 API 密钥和加密密钥的 Secret | `cma-server-keys` | `cma-onprem-server` |

无论 `earlierNames` 如何设置，命名空间都名为 `cma-server`，服务器的镜像都名为 `cma-server-image`。

每个设置了 `egress.mode: gateway` 的沙箱 values 文件都必须在 `egress.gateway` 下指明服务器的两个 Service。如果某个文件没有指明，请添加这两行。此示例中的名称适用于设置了 `earlierNames: true` 的安装：

```yaml
egress:
  gateway:
    serverService: cma-onprem
    serverEgressService: cma-onprem-egress
```

### Docker Compose：归档的布局和路径

如果您要从中升级的发布版本没有 `compose` 目录，则其 `docker-compose.yml` 位于归档顶层，与 `bin` 并列。在新归档中，该文件深一层，位于 `compose` 中。

在"重新进行您的其他更改"步骤中，还需进行以下更改：

* **您自己的 Compose 文件：** 如果旧归档没有 `compose` 目录，您的文件也要深一层。其中 `./bin` 之类的路径需改为 `../bin`。如果该文件引用了 `cma-onprem` 服务，请将该名称改为 `cma-server`。
* **位于 `/etc/cma-onprem/` 或 `/run/cma-onprem/` 下的路径：** 将每个此类路径改为以 `/etc/cma-server/` 或 `/run/cma-server/` 开头。请在新发布版本的 `.env` 和 `compose.config.yaml` 中、在您自己的 Compose 文件中，以及在您添加的每一行 `volumes:` 中查找。在 `volumes:` 行中，只更改第一个冒号之后的部分，即容器内的路径。
