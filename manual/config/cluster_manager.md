# SeaSearch Cluster Manager Configuration

Cluster Manager monitors SeaSearch data nodes and manages index assignments. It runs in the SeaSearch cluster gateway image, either alongside Proxy or as the only service in the container.

These settings target SeaSearch 1.0 Docker deployments. Pass variables through the service's `environment` section or a service-level `env_file`; adding them only to the Compose project `.env` file is not sufficient. See [Example Configuration](example.md#gateway).

## Gateway mode

| Variable | Description | Default |
| --- | --- | --- |
| `SS_GATEWAY_NODE_TYPE` | Docker startup mode: leave empty to run both Manager and Proxy, or set to `manager` to run only Manager. The value `proxy` runs only Proxy. | Empty (both services) |

Run only one Manager for a SeaSearch cluster. When Proxy runs separately, configure its [`SS_CLUSTER_MANAGER_URL`](seasearch_proxy.md#gateway-mode-and-manager-connection) to point to this Manager.

The Docker entrypoint fixes the Manager listener at `0.0.0.0:4081`; setting `SS_CLUSTER_MANAGER_HOST` or `SS_CLUSTER_MANAGER_PORT` does not change it. If necessary, change the host port through the Compose `ports` mapping while keeping the container port at `4081`. Keep the management endpoint on a trusted network.

## etcd

| Variable | Description | Default |
| --- | --- | --- |
| `SS_ETCD_ENDPOINTS` | Comma-separated etcd client endpoints, such as `etcd-1:2379,etcd-2:2379`. Required. | Unset |
| `SS_ETCD_PREFIX` | Cluster namespace. Must match all data nodes and proxies. | `/seasearch` |

## Logging

Defaults below apply without Compose overrides. Relative paths are resolved inside the container, from `/opt/cluster`.

| Variable | Description | Default |
| --- | --- | --- |
| `SS_LOG_TO_STDOUT` | Write logs to standard output for `docker logs` or a logging driver instead of log files. Shared with Proxy when both run in one container. | `false` |
| `SS_CLUSTER_MANAGER_LOG_DIR` | Container directory for file logs. Mount persistent storage here if needed. | `./log` (`/opt/cluster/log`) |
| `SS_CLUSTER_MANAGER_LOG_LEVEL` | Log verbosity, for example `debug`, `info`, `warn`, or `error`. | `INFO` |

## Apply changes

After editing the container environment, recreate the affected service. For the separate Manager [example](example.md#separate-manager-and-proxy):

```sh
docker compose up -d --force-recreate manager
```

For a combined gateway, use `gateway` instead. See [Cluster Deployment](../usage/deploy_cluster.md) for node registration and operational procedures.
