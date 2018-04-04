# Ping Definition

## Configuration

```toml
# Ping definition
[ping]
  # Name of the related entrypoint
  #
  # Optional
  # Default: "traefik"
  #
  entrypoint = "traefik"
```

| Path    | Method        | Description                                                                                        |
|---------|---------------|----------------------------------------------------------------------------------------------------|
| `/ping` | `GET`, `HEAD` | A simple endpoint to check for Træfik process liveness. Return a code `200` with the content: `OK` |


!!! warning
    Even if you have authentication configured on entrypoint, the `/ping` path of the api is excluded from authentication.

## Examples

The `/ping` health-check URL is enabled with the command-line `--ping` or config file option `[ping]`.
Thus, if you have a regular path for `/foo` and an entrypoint on `:80`, you would access them as follows:

* Regular path: `http://hostname:80/foo`
* Admin panel: `http://hostname:8080/`
* Ping URL: `http://hostname:8080/ping`

However, for security reasons, you may want to be able to expose the `/ping` health-check URL to outside health-checkers, e.g. an Internet service or cloud load-balancer, _without_ exposing your administration panel's port.
In many environments, the security staff may not _allow_ you to expose it.

You have two options:

* Enable `/ping` on a regular entrypoint
* Enable `/ping` on a dedicated port

### Ping health check on a regular entrypoint

To proxy `/ping` from a regular entrypoint to the administration one without exposing the panel, do the following:

```toml
defaultEntrypoints = ["http"]

[entrypoints]
  [entrypoints.http]
  address = ":80"

[ping]
entrypoint = "http"

```

The above link `ping` on the `http` entrypoint and then expose it on port `80`

### Enable ping health check on dedicated port

If you do not want to or cannot expose the health-check on a regular entrypoint - e.g. your security rules do not allow it, or you have a conflicting path - then you can enable health-check on its own entrypoint.
Use the following configuration:

```toml
defaultEntrypoints = ["http"]

[entrypoints]
  [entrypoints.http]
  address = ":80"
  [entrypoints.ping]
  address = ":8082"

[ping]
entrypoint = "ping"
```

The above is similar to the previous example, but instead of enabling `/ping` on the _default_ entrypoint, we enable it on a _dedicated_ entrypoint.

In the above example, you would access a regular path and health-check as follows:

* Regular path: `http://hostname:80/foo`
* Ping URL: `http://hostname:8082/ping`

Note the dedicated port `:8082` for `/ping`.

In the above example, it is _very_ important to create a named dedicated entrypoint, and do **not** include it in `defaultEntrypoints`.
Otherwise, you are likely to expose _all_ services via this entrypoint.

### Using ping for external Load-balancer rotation health check

If you are running traefik behind a external Load-balancer, and want to configure rotation health check on the Load-balancer to take a traefik instance out of rotation gracefully, you can configure [lifecycle.requestAcceptGraceTimeout](/configuration/commons.md#life-cycle) and the ping endpoint will return `503` response on traefik server termination, so that the Load-balancer can take the terminating traefik instance out of rotation, before it stops responding.
