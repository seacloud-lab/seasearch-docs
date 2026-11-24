# SeaSearch Configuration

This page describes the user-facing environment variables for SeaSearch 1.0 Docker containers. For Compose examples, see [Example Configuration](example.md).

## Set environment variables

Pass these variables to the container through the Compose service's `environment` section or a service-level `env_file`. A Compose project `.env` file supplies values for interpolation; adding a variable there does **not** automatically pass it to the container.

Defaults below are those of the application and its Docker entrypoint. Paths refer to paths **inside the container**.

After changing container environment variables, recreate the affected service from the Compose project directory:

```sh
docker compose up -d --force-recreate seasearch
```

Restarting an existing container does not update its environment. Keep the data directory on a persistent volume when recreating containers.

## Initial administrator

| Variable | Description | Default |
| --- | --- | --- |
| `SS_FIRST_ADMIN_USER` | Initial administrator username. Required when initializing a deployment with no users. | Unset |
| `SS_FIRST_ADMIN_PASSWORD` | Initial administrator password. Required together with the username. Use a strong password. | Unset |

These variables create the initial account; changing them later does not reset an existing account's password. The Docker entrypoint translates them into the names used internally by the application. Do not commit credentials to version control.

## Storage

| Variable | Description | Default |
| --- | --- | --- |
| `SS_STORAGE_TYPE` | Storage backend: `disk` for local storage, `s3` for S3-compatible storage, or `oss` for Alibaba Cloud OSS. Use `s3` for the documented cluster deployment. | `disk` |
| `SS_DATA_PATH` | Writable container directory for local data, including the local cache when using object storage. Mount a persistent host directory or volume here. | `./data` (`/opt/seasearch/data` in the image) |
| `SS_MAX_OBJ_CACHE_SIZE` | Maximum local object cache size when using object storage. Accepts sizes such as `10GB`. Plan sufficient local disk space for the cache and other local files. | `10GB` |

### S3-compatible storage

These settings apply when `SS_STORAGE_TYPE=s3`.

| Variable | Description | Default |
| --- | --- | --- |
| `SS_S3_BUCKET` | Existing bucket for SeaSearch data. Required. | Unset |
| `SS_S3_ENDPOINT` | S3 API endpoint host, optionally followed by a port, such as `s3.us-east-1.amazonaws.com` or `minio:9000`. Do not include a URL scheme or bucket path. Required, including for AWS. | Unset |
| `SS_S3_ACCESS_ID` | Access key ID with permission to access the bucket. Required. | Unset |
| `SS_S3_ACCESS_SECRET` | Secret access key. Required. | Unset |
| `SS_S3_AWS_REGION` | Bucket region. Set this to the region required by your provider. | `us-east-1` |
| `SS_S3_USE_HTTPS` | Use HTTPS to connect to storage. Keep enabled for production. | `true` |
| `SS_S3_USE_V4_SIGNATURE` | Use Signature Version 4 authentication. Keep enabled unless your provider requires an older signature version. | `true` |
| `SS_S3_PATH_STYLE_REQUEST` | Use `host/bucket/object` instead of `bucket.host/object`. Commonly needed for self-hosted S3-compatible services. Set according to your provider's requirements. | `true` |
| `SS_S3_SSE_C_KEY` | Optional customer-provided encryption key for SSE-C, if supported by your provider. Requires HTTPS and V4 signatures. Keep the key safe and available for subsequent access to encrypted objects. | Unset (disabled) |

The Docker entrypoint supplies the region, HTTPS, signature, and path-style defaults above.

### Alibaba Cloud OSS

These settings apply when `SS_STORAGE_TYPE=oss`. All four are required.

| Variable | Description | Default |
| --- | --- | --- |
| `SS_OSS_ACCESS_ID` | OSS access key ID. | Unset |
| `SS_OSS_ACCESS_SECRET` | OSS secret access key. | Unset |
| `SS_OSS_BUCKET` | Existing bucket for SeaSearch data. | Unset |
| `SS_OSS_ENDPOINT` | OSS service endpoint, such as `oss-cn-hangzhou.aliyuncs.com`. | Unset |

## Cluster

Use these settings for SeaSearch data nodes, together with shared S3 storage. See [Cluster Deployment](../usage/deploy_cluster.md) for the architecture and node registration procedure.

| Variable | Description | Default |
| --- | --- | --- |
| `SS_CLUSTER_ENABLE` | Enable cluster mode. Set to `true` on each data node. | `false` |
| `SS_CLUSTER_ID` | Node ID. Use a distinct ID for each node, matching the ID used during cluster registration. Do not change it after initialization. | `1` |
| `SS_ETCD_ENDPOINTS` | Comma-separated etcd **client** endpoints, for example `etcd-1:2379,etcd-2:2379`. Required in cluster mode. | Unset |
| `SS_ETCD_PREFIX` | Namespace for the cluster's etcd data. Must match on all data nodes, proxies, and the manager. Use a different prefix for a separate cluster. | `/seasearch` |

## Logging

| Variable | Description | Default |
| --- | --- | --- |
| `SS_LOG_TO_STDOUT` | Write logs to standard output for collection with `docker logs` or a container logging driver. Otherwise, write to log files. | `false` |
| `SS_LOG_DIR` | Container directory for log files when file logging is used. Mount it on persistent storage if logs must survive container replacement. | `./log` (`/opt/seasearch/log` in the image) |
| `SS_LOG_LEVEL` | Log verbosity, for example `debug`, `info`, `warn`, or `error`. Use `info` for normal operation. | `debug` |
