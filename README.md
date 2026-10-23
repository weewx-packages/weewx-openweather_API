# WeeWX OpenWeather API

This service retrieves current weather and air-pollution observations from
OpenWeather, maps selected fields to `ow_*` observations, and adds them to WeeWX
archive records. It runs alongside your station driver. It supports temperature,
humidity, pressure, wind, precipitation, visibility, weather descriptions and
pollutants such as PM2.5, PM10 and ozone when supplied by the selected API.

This repository mirrors [inguy24/weewx-openweather_API](https://github.com/inguy24/weewx-openweather_API).
Use the separately repaired catalog archive `1.0.0.post2026102201` below.
A source-branch ZIP does not include those packaging repairs. Use `1.0.0.post2026102201`; the preceding
`1.0.0.post20261022` catalog archive is yanked. The unchanged runtime reports
1.0.1, while the installer suffix identifies the packaging revision.

## Install

Use WeeWX 5.5–5.x and Python 3.10–3.14, with `PyYAML` installed in the WeeWX
Python environment. You need an OpenWeather API key with access to the selected
APIs and working station latitude/longitude in `weewx.conf`.

```sh
python -m pip install PyYAML
```

Stop WeeWX and back up its configuration and archive database. The installer
adds selected columns to the initialized `wx_binding` database. If the station
archive has not been initialized, initialize it first with
`weectl database create --config /path/to/weewx.conf`.

Install the published archive with WeeWX 5's extension installer:

```sh
weectl extension install https://github.com/weewx-packages/catalog/releases/download/openweather-api/v1.0.0.post2026102201/openweather-api-1.0.0.post2026102201.tar.gz
```

Use the `weectl` from your WeeWX environment. Add
`--config /path/to/weewx.conf` when the station configuration is not found
automatically. Package installations may require `sudo`; activate the WeeWX
virtual environment for pip installations. Restart your WeeWX service after
configuration, for example `sudo systemctl restart weewx` on systemd.
The installer does not restart it for you.

Answer the installer's API key, field selection and interval prompts.
The installer copies `openweather.py` and `openweather_fields.yaml` together
into the station's `bin/user`, registers
`user.openweather.OpenWeatherService` in `data_services`, and adds
`[OpenWeatherService]`. It keeps new configuration in memory until the WeeWX
extension installer saves the configuration. There is no separate mandatory
`/etc/weewx/openweather_fields.conf` file.

## Use and configure

Keep the generated field selection, mappings and conversion sections together.
A minimal edit to the generated configuration is:

```ini
[OpenWeatherService]
    enable = true
    api_key = YOUR_OPENWEATHER_KEY
    [[intervals]]
        current_weather = 3600
        air_quality = 7200
```

This excerpt augments the installer-generated configuration; it cannot replace
its `field_selection`, `field_mappings` and unit metadata. The default intervals
are one hour for weather and two hours for air quality. Check API permissions
and limits with OpenWeather; selecting fewer fields does not necessarily reduce
the number of HTTP calls. Data is added on archive events, not as a skin report.

Use fields such as `$current.ow_temperature` and `$current.ow_pm25` in templates.
The [field and query reference](docs/fields.md) retains all selectable names,
report examples and SQLite queries. The [YAML definition](openweather_fields.yaml)
is the authoritative field mapping supplied with the source.

## Upgrade, change selection or remove

Back up `weewx.conf` and the database, stop WeeWX, and reinstall the fixed archive
to run its interactive setup again. Review the resulting settings before
restarting. Stock WeeWX 5.5.2 has no extension reconfiguration command; runtime
messages suggesting one are outdated. Edit simple settings such as the API key
in `weewx.conf`, and keep field mappings consistent when changing selection.

Remove with `weectl extension uninstall OpenWeather` and restart WeeWX.
Existing database columns and historical data remain.

## Diagnostics and limits

Use WeeWX's environment and the installed module for command help:

```sh
PYTHONPATH=/path/to/station/bin python -m user.openweather --help
```

`--test-install`, `--test-api` and `--test-all` exist, but their built-in
diagnostics assume `/etc/weewx/weewx.conf`. The API modes make real network
requests. For another layout, inspect the WeeWX service log instead of treating
those diagnostics as station-independent tests.
Missing credentials, selections or mappings disable collection. API availability
and returned fields depend on OpenWeather; local fixture checks do not verify a
live account or its subscriptions.

## License and origin

Shane Burkhardt's project retains its [GPLv3 license](LICENSE).
See [CONTRIBUTING.md](CONTRIBUTING.md) for source contribution guidance.
