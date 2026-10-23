These are the selectable fields in the packaged YAML. Only selected fields
are created. See the [installation guide](../README.md) before changing selection.
The template examples use WeeWX's configured unit labels.

# OpenWeather fields
Fields are prefixed with `ow_` to avoid conflicts:

**Current Weather Fields (20 fields):**
- `ow_temperature` (REAL) - Current temperature
- `ow_feels_like` (REAL) - Feels-like temperature
- `ow_temp_min` (REAL) - API-reported minimum temperature
- `ow_temp_max` (REAL) - API-reported maximum temperature
- `ow_humidity` (REAL) - Relative humidity percentage
- `ow_pressure` (REAL) - Atmospheric pressure
- `ow_sea_level` (REAL) - Sea level pressure
- `ow_grnd_level` (REAL) - Ground level pressure
- `ow_wind_speed` (REAL) - Wind speed
- `ow_wind_direction` (REAL) - Wind direction in degrees
- `ow_wind_gust` (REAL) - Wind gusts
- `ow_cloud_cover` (REAL) - Cloud cover percentage
- `ow_visibility` (REAL) - Visibility in meters
- `ow_rain_1h` (REAL) - Rain in last hour
- `ow_rain_3h` (REAL) - Rain in last 3 hours
- `ow_snow_1h` (REAL) - Snow in last hour
- `ow_snow_3h` (REAL) - Snow in last 3 hours
- `ow_weather_main` (VARCHAR(20)) - Weather category
- `ow_weather_description` (VARCHAR(50)) - Weather description
- `ow_weather_icon` (VARCHAR(10)) - Weather icon code

**Air Quality Fields (9 fields):**
- `ow_aqi` (INTEGER) - Air Quality Index (1-5 scale)
- `ow_pm25` (REAL) - PM2.5 concentration
- `ow_pm10` (REAL) - PM10 concentration
- `ow_ozone` (REAL) - Ozone concentration
- `ow_no2` (REAL) - Nitrogen dioxide concentration
- `ow_so2` (REAL) - Sulfur dioxide concentration
- `ow_co` (REAL) - Carbon monoxide concentration
- `ow_nh3` (REAL) - Ammonia concentration
- `ow_no` (REAL) - Nitrogen monoxide concentration

### Report Integration

You can create custom reports using the new fields:

```html
<!-- In your skin templates -->
<p>Current Temperature: $current.ow_temperature</p>
<p>Feels Like: $current.ow_feels_like</p>
<p>Weather: $current.ow_weather_description</p>
<p>Air Quality Index: $current.ow_aqi</p>
<p>PM2.5 Level: $current.ow_pm25 μg/m³</p>

<!-- Historical data -->
<p>Today's Max Temperature: $day.ow_temp_max.max</p>
<p>This Week's Average AQI: $week.ow_aqi.avg</p>
```

### Database Queries

Access data directly from the database:

```sql
-- Recent readings with OpenWeather data
SELECT datetime(dateTime, 'unixepoch', 'localtime') as date,
       outTemp, ow_temperature, ow_feels_like, ow_weather_main,
       ow_aqi, ow_pm25
FROM archive
WHERE ow_temperature IS NOT NULL
ORDER BY dateTime DESC
LIMIT 24;

-- Daily averages with weather conditions
SELECT date(datetime(dateTime, 'unixepoch', 'localtime')) as date,
       ROUND(AVG(ow_temperature), 1) as avg_temp,
       ROUND(AVG(ow_aqi), 1) as avg_aqi
FROM archive
WHERE ow_temperature IS NOT NULL
GROUP BY date
ORDER BY date DESC;
```
