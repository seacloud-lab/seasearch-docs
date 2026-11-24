# SeaSearch Proxy Configuration

SeaSearch Proxy forwards client requests to the appropriate SeaSearch data nodes. It runs in the SeaSearch cluster gateway image, either alongside Cluster Manager or in a separate container.

These settings target SeaSearch 1.0 Docker deployments. Pass variables through the service's `environment` section or a service-level `env_file`; a project `.env` file alone does not pass them to the container. See [Example Configuration](example.md#gateway) for Compose examples.

## Gateway mode and manager connection

| Variable | Description | Default |
| --- | --- | --- |
| `SS_GATEWAY_NODE_TYPE` | Docker startup mode: leave empty to run both Proxy and Manager, or set to `proxy` to run only Proxy. The value `manager` runs only Manager. | Empty (both services) |
| `SS_CLUSTER_MANAGER_URL` | Manager URL reachable from the Proxy container, for example `http://manager:4081/`. Set explicitly for a separate Proxy container. | `http://127.0.0.1:4081/` in combined mode; unset in proxy-only mode |

In combined mode, the entrypoint overwrites the manager URL with the local address. In proxy-only mode, it preserves the value you supply. Do not use `127.0.0.1` for a manager in another container or on another host.

The Docker entrypoint fixes the Proxy listener at `0.0.0.0:4082`; setting `SS_CLUSTER_PROXY_HOST` or `SS_CLUSTER_PROXY_PORT` does not change it. To change the published host port, edit the Compose `ports` mapping, for example `"8080:4082"`.

## etcd

| Variable | Description | Default |
| --- | --- | --- |
| `SS_ETCD_ENDPOINTS` | Comma-separated etcd client endpoints, such as `etcd-1:2379,etcd-2:2379`. Required. | Unset |
| `SS_ETCD_PREFIX` | Cluster namespace. Must match the data nodes and Manager. | `/seasearch` |

## Logging

Defaults below apply without Compose overrides. Relative paths are resolved inside the container, from `/opt/cluster`.

| Variable | Description | Default |
| --- | --- | --- |
| `SS_LOG_TO_STDOUT` | Write logs to standard output for `docker logs` or a logging driver instead of log files. Shared with Manager when both run in one container. | `false` |
| `SS_CLUSTER_PROXY_LOG_DIR` | Container directory for file logs. Mount persistent storage here if needed. | `./log` (`/opt/cluster/log`) |
| `SS_CLUSTER_PROXY_LOG_LEVEL` | Log verbosity, for example `debug`, `info`, `warn`, or `error`. | `INFO` |

## Apply changes

After editing the container environment, recreate the affected service. For the separate Proxy [example](example.md#separate-manager-and-proxy):

```sh
docker compose up -d --force-recreate proxy
```

For a combined gateway, use `gateway` instead. See [Cluster Deployment](../usage/deploy_cluster.md) for node registration and operational procedures.
