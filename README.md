# influxdb-2

[InfluxDB](https://www.influxdata.com) template for Runway. Time series
database for metrics, events, and IoT data, with the web UI, Flux, and
InfluxQL.

Single instance with persistence on a volume. InfluxDB 2 — not the 3.x line.

## Deploy

WIP

## Inputs

| Input | Default | Notes |
| --- | --- | --- |
| `image_tag` | `2.9-alpine` | 2.9 current, 2.8 previous release |
| `organization` | `app` | Created on first start |
| `bucket` | `app` | Created on first start |
| `retention` | `0` | e.g. `30d`, `12w`; `0` keeps data forever. First start only |
| `username` | `admin` | Admin user for the web UI |

## Connecting

This one **is** routed: it gets a public HTTPS URL, so Telegraf, Grafana, and
the web UI can reach it. Every API request needs a token.

The credentials are in the app config:

```sh
runway app config ls --app my-influx
```

- Log in to the web UI with `<username>` and `DOCKER_INFLUXDB_INIT_PASSWORD`.
- `DOCKER_INFLUXDB_INIT_ADMIN_TOKEN` is the operator token. Use it to mint
  scoped tokens:

  ```sh
  influx auth create --host https://my-influx.example.com \
    --token <DOCKER_INFLUXDB_INIT_ADMIN_TOKEN> \
    --org app --write-bucket <bucket-id>
  ```

Give Telegraf and apps the scoped token, not the operator token.

## Notes

- **Password and token only apply on the first start**, when the volume is
  empty. Changing them in the config later does nothing in InfluxDB — it just
  leaves your clients with a value that no longer works. Rotate in the UI or
  with `influx user password` / `influx auth`.
- **The template sets no `command`.** The image entrypoint reads the
  `DOCKER_INFLUXDB_INIT_*` variables and runs the setup before starting
  `influxd`. Overriding the command skips that.
- `INFLUX_CONFIGS_PATH` points into the volume: the default `/etc/influxdb2`
  is not persisted and not writable as a non-root user.
- No lockdown: the setup step writes to `/tmp` on first start.
- Telemetry to InfluxData is off (`INFLUXD_REPORTING_DISABLED`).

## License

InfluxDB is MIT. This template is maintained by Runway.
