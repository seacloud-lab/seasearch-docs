# Example SeaSearch Configuration

These examples pass `SS_` environment variables directly to SeaSearch 1.0 Docker containers.

Save the selected configuration as `compose.yml`. Store credentials and interpolated values in a project `.env` file alongside it, restrict access to that file, and do not commit it to version control. The `environment` entries below explicitly pass values into the container; `.env` alone does not do so.

## Single node with local storage

```yaml
services:
  seasearch:
    image: seafileltd/seasearch:1.0-latest
    ports:
      - "4080:4080"
    volumes:
      - ./data:/opt/seasearch/data
    environment:
      SS_FIRST_ADMIN_USER: ${SS_FIRST_ADMIN_USER:?Set the initial admin username}
      SS_FIRST_ADMIN_PASSWORD: ${SS_FIRST_ADMIN_PASSWORD:?Set the initial admin password}
      SS_DATA_PATH: /opt/seasearch/data
      SS_STORAGE_TYPE: disk
      SS_LOG_TO_STDOUT: "true"
      SS_LOG_LEVEL: info
```

Set initial credentials in `.env`:

```dotenv
SS_FIRST_ADMIN_USER=admin
SS_FIRST_ADMIN_PASSWORD=<strong-password>
```

The host directory `./data` is mounted at the container's `SS_DATA_PATH`. Keep it when replacing the container. On platforms that require the NoMKL image, use `seafileltd/seasearch-nomkl:1.0-latest` as described in [Single-node Deployment](../usage/deploy_single_node.md).

Start the service:

```sh
docker compose up -d
```

After initialization, you may remove both administrator entries from `environment` and their values from `.env`. Changing them does not reset an existing password. Remove the Compose entries too, since their `:?` checks would otherwise still require the values.

## Single node with S3-compatible storage

Keep the local example's image, port, volume, and logging settings, but replace its `environment` section with:

```yaml
    environment:
      SS_FIRST_ADMIN_USER: ${SS_FIRST_ADMIN_USER:?Set the initial admin username}
      SS_FIRST_ADMIN_PASSWORD: ${SS_FIRST_ADMIN_PASSWORD:?Set the initial admin password}
      SS_DATA_PATH: /opt/seasearch/data
      SS_STORAGE_TYPE: s3
      SS_MAX_OBJ_CACHE_SIZE: 10GB
      SS_S3_BUCKET: ${SS_S3_BUCKET:?Set the bucket}
      SS_S3_ENDPOINT: ${SS_S3_ENDPOINT:?Set the endpoint host}
      SS_S3_ACCESS_ID: ${SS_S3_ACCESS_ID:?Set the access key ID}
      SS_S3_ACCESS_SECRET: ${SS_S3_ACCESS_SECRET:?Set the secret access key}
      SS_S3_AWS_REGION: ${SS_S3_AWS_REGION:-us-east-1}
      SS_S3_USE_HTTPS: "true"
      SS_S3_USE_V4_SIGNATURE: "true"
      SS_S3_PATH_STYLE_REQUEST: ${SS_S3_PATH_STYLE_REQUEST:-true}
      SS_LOG_TO_STDOUT: "true"
      SS_LOG_LEVEL: info
```

For AWS S3, add the following to `.env` alongside the initial administrator credentials. Replace the bucket, credentials, endpoint, and region with your own values:

```dotenv
SS_S3_BUCKET=<existing-bucket>
SS_S3_ENDPOINT=s3.us-east-1.amazonaws.com
SS_S3_ACCESS_ID=<access-key-id>
SS_S3_ACCESS_SECRET=<secret-access-key>
SS_S3_AWS_REGION=us-east-1
SS_S3_PATH_STYLE_REQUEST=false
```

For another S3-compatible service, use its API endpoint host (optionally with a port), region, and supported address style. Do not include `https://` or a bucket path in the endpoint. Path-style requests are commonly needed for self-hosted storage. Keep HTTPS enabled for production; to use a test service without TLS, explicitly change `SS_S3_USE_HTTPS` to `"false"` in Compose.

The endpoint is required even for AWS. The local volume is still needed for local data and the object cache. See [S3 settings](seasearch.md#s3-compatible-storage) for details and optional SSE-C configuration.

## Single node with Alibaba Cloud OSS

Keep the local example's image, port, and volume, and replace its `environment` section with:

```yaml
    environment:
      SS_FIRST_ADMIN_USER: ${SS_FIRST_ADMIN_USER:?Set the initial admin username}
      SS_FIRST_ADMIN_PASSWORD: ${SS_FIRST_ADMIN_PASSWORD:?Set the initial admin password}
      SS_DATA_PATH: /opt/seasearch/data
      SS_STORAGE_TYPE: oss
      SS_MAX_OBJ_CACHE_SIZE: 10GB
      SS_OSS_ACCESS_ID: ${SS_OSS_ACCESS_ID:?Set the OSS access key ID}
      SS_OSS_ACCESS_SECRET: ${SS_OSS_ACCESS_SECRET:?Set the OSS secret access key}
      SS_OSS_BUCKET: ${SS_OSS_BUCKET:?Set the OSS bucket}
      SS_OSS_ENDPOINT: ${SS_OSS_ENDPOINT:?Set the OSS endpoint}
      SS_LOG_TO_STDOUT: "true"
      SS_LOG_LEVEL: info
```

Add the four OSS values to `.env`, alongside the initial administrator credentials. For example, use `oss-cn-hangzhou.aliyuncs.com` as the endpoint for a bucket in that region.

## Cluster data nodes

Use the S3 example on each data node and add these entries to its existing `environment` mapping:

```yaml
      SS_CLUSTER_ENABLE: "true"
      SS_CLUSTER_ID: ${SS_CLUSTER_ID:?Set a unique node ID}
      SS_ETCD_ENDPOINTS: ${SS_ETCD_ENDPOINTS:?Set the etcd client endpoints}
      SS_ETCD_PREFIX: ${SS_ETCD_PREFIX:-/seasearch}
```

Set a distinct `SS_CLUSTER_ID` in each node's `.env`. Use the same S3 bucket, etcd endpoints, and etcd prefix across the cluster. Each data node needs its own persistent local directory. For example:

```dotenv
SS_CLUSTER_ID=1
SS_ETCD_ENDPOINTS=192.168.1.11:2379,192.168.1.12:2379,192.168.1.13:2379
SS_ETCD_PREFIX=/seasearch
```

Use etcd client ports (normally `2379`), not peer ports (normally `2380`). Follow [Cluster Deployment](../usage/deploy_cluster.md) to register node IDs and their reachable addresses after starting the data nodes and Manager.

## Gateway

The following examples assume reachable data nodes and etcd are already available. Keep Manager and etcd accessible only on trusted networks.

### Combined Manager and Proxy

On the gateway host, use a separate Compose project:

```yaml
services:
  gateway:
    image: seafileltd/seasearch-cluster-gateway:1.0-latest
    ports:
      - "4082:4082"
    environment:
      SS_GATEWAY_NODE_TYPE: ""
      SS_ETCD_ENDPOINTS: ${SS_ETCD_ENDPOINTS:?Set the etcd client endpoints}
      SS_ETCD_PREFIX: ${SS_ETCD_PREFIX:-/seasearch}
      SS_LOG_TO_STDOUT: "true"
      SS_CLUSTER_MANAGER_LOG_LEVEL: info
      SS_CLUSTER_PROXY_LOG_LEVEL: info
```

Set the same etcd endpoints and prefix as the data nodes in this project's `.env`. The container starts both services and fixes the Manager URL used by Proxy to `http://127.0.0.1:4081/`. Only the Proxy port is published to the host.

Start the gateway, then register nodes using their IDs and addresses reachable from the gateway:

```sh
docker compose up -d
docker compose exec gateway register-cluster "1:192.168.1.21:4080,2:192.168.1.22:4080,3:192.168.1.23:4080"
```

### Separate Manager and Proxy

This example runs the services in separate containers on one Docker host. Only one Manager should run for the cluster. Proxy reaches it over the Compose network using the service name `manager`:

```yaml
services:
  manager:
    image: seafileltd/seasearch-cluster-gateway:1.0-latest
    environment:
      SS_GATEWAY_NODE_TYPE: manager
      SS_ETCD_ENDPOINTS: ${SS_ETCD_ENDPOINTS:?Set the etcd client endpoints}
      SS_ETCD_PREFIX: ${SS_ETCD_PREFIX:-/seasearch}
      SS_LOG_TO_STDOUT: "true"
      SS_CLUSTER_MANAGER_LOG_LEVEL: info

  proxy:
    image: seafileltd/seasearch-cluster-gateway:1.0-latest
    ports:
      - "4082:4082"
    environment:
      SS_GATEWAY_NODE_TYPE: proxy
      SS_CLUSTER_MANAGER_URL: http://manager:4081/
      SS_ETCD_ENDPOINTS: ${SS_ETCD_ENDPOINTS:?Set the etcd client endpoints}
      SS_ETCD_PREFIX: ${SS_ETCD_PREFIX:-/seasearch}
      SS_LOG_TO_STDOUT: "true"
      SS_CLUSTER_PROXY_LOG_LEVEL: info
```

Start Manager, register the data nodes, then start Proxy:

```sh
docker compose up -d manager
docker compose exec manager register-cluster "1:192.168.1.21:4080,2:192.168.1.22:4080,3:192.168.1.23:4080"
docker compose up -d proxy
```

For separate hosts, publish Manager's container port `4081` on a trusted interface and change `SS_CLUSTER_MANAGER_URL` to its reachable private address. The Compose service name `manager` does not resolve across independent Docker hosts. See [Proxy Configuration](seasearch_proxy.md) and [Manager Configuration](cluster_manager.md) for all documented options.

## Apply configuration changes

After changing environment variables, recreate the affected service, for example:

```sh
docker compose up -d --force-recreate seasearch
```

For the gateway examples, substitute `gateway`, `manager`, or `proxy` as appropriate. A plain container restart does not update its environment. Keep persistent data volumes when recreating data-node containers.
