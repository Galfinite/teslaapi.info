---
description: >-
  An energy site is a home installation of batteries or solar panels (formerly
  from SolarCity, which was acquired by Tesla).
---

# State And Settings

{% hint style="warning" %}
Work In Progress
{% endhint %}

{% api-method method="get" host="https://owner-api.teslamotors.com" path="/api/1/energy\_sites/:site\_id/live\_status" %}
{% api-method-summary %}
Site Data
{% endapi-method-summary %}

{% api-method-description %}
Shows a real-time view of the power output of the site.  
The Tesla Android app polls this method every 3 seconds to show the kW output.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name=":site\_id" type="integer" required=true %}
The `{energy_site_id}` from the products list
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}

{% api-method-headers %}
{% api-method-parameter name="Authorization" type="string" required=true %}
Bearer `{access_token}` from authentication
{% endapi-method-parameter %}
{% endapi-method-headers %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}
The `solar_power` value is the number of kilowatts \(kW\) currently being generated.  
Sometimes, the `grid_status` and `grid_services_active` fields are missing from the response object.
{% endapi-method-response-example-description %}

```javascript
{
	"response": {
		"solar_power": 0,
		"grid_status": "Unknown",
		"grid_services_active": false,
		"timestamp": "2019-12-23T16:01:11-05:00"
	}
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="get" host="https://owner-api.teslamotors.com" path="/api/1/energy\_sites/:site\_id/site\_info" %}
{% api-method-summary %}
Site Configuration
{% endapi-method-summary %}

{% api-method-description %}
Get installation and configuration details about the site.  
The `site_name` field value can be changed using the **Site Name** command.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name=":site\_id" type="integer" required=true %}
The `{energy_site_id}` from the products list
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}

{% api-method-headers %}
{% api-method-parameter name="Authorization" type="string" required=true %}
Bearer `{access_token}` from authentication
{% endapi-method-parameter %}
{% endapi-method-headers %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}
This installation has solar panels only and is located in the `America/New_York` time zone.  
If the site does not have a name set \(using the **Site Name** command\), the `site_name` field will be missing from the response object.
{% endapi-method-response-example-description %}

```javascript
{
  "response": {
    "id": "00000000-0000-0000-0000-000000000000",
    "site_name": "My Site",
    "installation_date": "2017-01-01T00:00:00-05:00",
    "user_settings": {
      "storm_mode_enabled": null,
      "sync_grid_alert_enabled": false,
      "breaker_alert_enabled": false
    },
    "components": {
      "solar": true,
      "battery": false,
      "grid": true,
      "backup": false,
      "gateway": "neo",
      "load_meter": false,
      "tou_capable": false,
      "storm_mode_capable": false,
      "flex_energy_request_capable": false,
      "car_charging_data_supported": false,
      "configurable": false,
      "grid_services_enabled": false
    },
    "time_zone_offset": -300
  }
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="get" host="https://owner-api.teslamotors.com" path="/api/1/energy\_sites/:site\_id/calendar\_history" %}
{% api-method-summary %}
Historical Calendar Data
{% endapi-method-summary %}

{% api-method-description %}
Generate a report for solar, grid, and battery data up to a given date, aligned with the start of various calendar periods. Reports for certain periods will return subtotals for smaller constituent periods.  
This method is used to render bar and line graphs in the Tesla Android app.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name=":site\_id" type="integer" required=true %}
The `{energy_site_id}` from the products list
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}

{% api-method-headers %}
{% api-method-parameter name="Authorization" type="string" required=true %}
Bearer `{access_token}` from authentication
{% endapi-method-parameter %}
{% endapi-method-headers %}

{% api-method-query-parameters %}
{% api-method-parameter name="end\_date" type="string" required=false %}
ISO 8601 datetime, _e.g._ `2019-12-23T17:39:18.546Z`. The response report interval ends on this datetime and starts at the beginning of the given `period` at 1:00 AM. Defaults to the current time.
{% endapi-method-parameter %}

{% api-method-parameter name="time\_zone" type="string" required=false %}
IANA/Olsen time zone identifier, _e.g._ `America/New_York`. Seems to have no effect on response data.
{% endapi-method-parameter %}

{% api-method-parameter name="period" type="string" required=true %}
Amount of time to include in report. One of `day`, `week`, `month`, `year`, and `lifetime`. When `kind` is `power`, this parameter is ignored, is not required, and is always treated as `day`.
{% endapi-method-parameter %}

{% api-method-parameter name="kind" type="string" required=true %}
Type of report to generate. One of `power`, `energy`, and `self_consumption`.
{% endapi-method-parameter %}
{% endapi-method-query-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}
**time\_series**  
The `time_series` array will contain objects with subtotals for one or more smaller periods inside the specified `period`.  
  
_energy_  
A request for a `month` will return a time series of objects with subtotals for each day \(starting at 1 AM\), `year` will return the same but with month subtotals, `week` will return days, `lifetime` will return years, and `day` will return a single object.  
  
_power_  
The time series will always be in 15 minute increments \(hour aligned\) starting at 12:00 AM on the given `end_date`, or the current date if `end_date` is missing, inclusive. The time series will finish at 11:45 PM on given day or at the last 15 minute increment before the current time, whichever is earlier, inclusive.  
The `period` parameter is ignored when `kind=power`, so you cannot get a 15 minute time series over an entire month with a single request, for example.  
  
**time\_zone\_offset**  
Number of minutes that the installation and this response report are ahead of UTC.  
  
**Energy imported/exported**  
Values are numbers \(floating-point or integer\) representing kilowatt-hours \(kWh\).  
  
_The comments in the JSON snippets below are not part of the actual HTTP responses._
{% endapi-method-response-example-description %}

{% tabs %}
{% tab title="Energy for a day" %}
```javascript
/*
 * Between 1:00 AM and 12:39 PM on December 23, 2019, 
 * the solar panels generated 3.61 kWh of energy.
 *
 * ?kind=energy&period=day&time_zone=America%2FNew_York&end_date=2019-12-23T17%3A39%3A18.546Z
 */
   
{  
  "response": {
    "serial_number": "00000000-0000-0000-0000-000000000000",
    "period": "day",
    "time_zone_offset": -300,
    "time_series": [
      {
        "timestamp": "2019-12-23T01:00:00-05:00",
        "solar_energy_exported": 3610.0000000000005,
        "grid_energy_imported": 0,
        "grid_services_energy_imported": 0,
        "grid_services_energy_exported": 0,
        "grid_energy_exported_from_solar": 0,
        "grid_energy_exported_from_battery": 0,
        "battery_energy_exported": 0,
        "battery_energy_imported_from_grid": 0,
        "battery_energy_imported_from_solar": 0,
        "consumer_energy_imported_from_grid": 0,
        "consumer_energy_imported_from_solar": 0,
        "consumer_energy_imported_from_battery": 0
      }
    ]
  }
}
```
{% endtab %}

{% tab title="Energy for a month" %}
```javascript
/*
 * For month reports, the time series includes subtotals for each 
 * day between the first day of the month and the given end date 
 * (or the current time if end_date is missing), inclusive.
 *
 * ?kind=energy&period=month&time_zone=America%2FNew_York&end_date=2019-12-04T00%3A00%3A00.000-05%3A00
 */

{
  "response": {
    "serial_number": "00000000-0000-0000-0000-000000000000",
    "period": "month",
    "time_zone_offset": -300,
    "time_series": [{
      "timestamp": "2019-12-01T01:00:00-05:00",
      "solar_energy_exported": 1470,
      "grid_energy_imported": 0,
      "grid_services_energy_imported": 0,
      "grid_services_energy_exported": 0,
      "grid_energy_exported_from_solar": 0,
      "grid_energy_exported_from_battery": 0,
      "battery_energy_exported": 0,
      "battery_energy_imported_from_grid": 0,
      "battery_energy_imported_from_solar": 0,
      "consumer_energy_imported_from_grid": 0,
      "consumer_energy_imported_from_solar": 0,
      "consumer_energy_imported_from_battery": 0
    }, {
      "timestamp": "2019-12-02T01:00:00-05:00",
      "solar_energy_exported": 0,
      "grid_energy_imported": 0,
      "grid_services_energy_imported": 0,
      "grid_services_energy_exported": 0,
      "grid_energy_exported_from_solar": 0,
      "grid_energy_exported_from_battery": 0,
      "battery_energy_exported": 0,
      "battery_energy_imported_from_grid": 0,
      "battery_energy_imported_from_solar": 0,
      "consumer_energy_imported_from_grid": 0,
      "consumer_energy_imported_from_solar": 0,
      "consumer_energy_imported_from_battery": 0
    }, {
      "timestamp": "2019-12-03T01:00:00-05:00",
      "solar_energy_exported": 0,
      "grid_energy_imported": 0,
      "grid_services_energy_imported": 0,
      "grid_services_energy_exported": 0,
      "grid_energy_exported_from_solar": 0,
      "grid_energy_exported_from_battery": 0,
      "battery_energy_exported": 0,
      "battery_energy_imported_from_grid": 0,
      "battery_energy_imported_from_solar": 0,
      "consumer_energy_imported_from_grid": 0,
      "consumer_energy_imported_from_solar": 0,
      "consumer_energy_imported_from_battery": 0
    }, {
      "timestamp": "2019-12-04T00:00:00-05:00",
      "solar_energy_exported": 0,
      "grid_energy_imported": 0,
      "grid_services_energy_imported": 0,
      "grid_services_energy_exported": 0,
      "grid_energy_exported_from_solar": 0,
      "grid_energy_exported_from_battery": 0,
      "battery_energy_exported": 0,
      "battery_energy_imported_from_grid": 0,
      "battery_energy_imported_from_solar": 0,
      "consumer_energy_imported_from_grid": 0,
      "consumer_energy_imported_from_solar": 0,
      "consumer_energy_imported_from_battery": 0
    }]
  }
}
```
{% endtab %}

{% tab title="Power for a day" %}
```javascript
/*
 * When kind=power, the time_series always starts at midnight on 
 * the given end_date, and proceeds in 15 minute increments. It 
 * stops before the end_date or, if the end_date is in the future,
 * before the current time.
 *
 * ?kind=power&period=day&end_date=2019-12-21T01%3A25%3A00.000-05%3A00&time_zone=America%2FNew_York
 */

{
  "response": {
    "serial_number": "00000000-0000-0000-0000-000000000000",
    "time_zone_offset": -300,
    "time_series": [{
      "timestamp": "2019-12-21T00:00:00-05:00",
      "solar_power": 0,
      "battery_power": 0,
      "grid_power": 0,
      "grid_services_power": 0
    }, {
      "timestamp": "2019-12-21T00:15:00-05:00",
      "solar_power": 0,
      "battery_power": 0,
      "grid_power": 0,
      "grid_services_power": 0
    }, {
      "timestamp": "2019-12-21T00:30:00-05:00",
      "solar_power": 0,
      "battery_power": 0,
      "grid_power": 0,
      "grid_services_power": 0
    }, {
      "timestamp": "2019-12-21T00:45:00-05:00",
      "solar_power": 0,
      "battery_power": 0,
      "grid_power": 0,
      "grid_services_power": 0
    }, {
      "timestamp": "2019-12-21T01:00:00-05:00",
      "solar_power": 0,
      "battery_power": 0,
      "grid_power": 0,
      "grid_services_power": 0
    }, {
      "timestamp": "2019-12-21T01:15:00-05:00",
      "solar_power": 0,
      "battery_power": 0,
      "grid_power": 0,
      "grid_services_power": 0
    }
  ]
}
```
{% endtab %}
{% endtabs %}
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="get" host="https://owner-api.teslamotors.com" path="/api/1/energy\_sites/:site\_id/history" %}
{% api-method-summary %}
Historical Data
{% endapi-method-summary %}

{% api-method-description %}
May be a deprecated method. Seems to be similar to the **Historical Calendar Data** method above, except these responses are missing the `time_zone_offset` field.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name=":site\_id" type="integer" required=true %}
The `{energy_site_id}` from the products list
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}

{% api-method-headers %}
{% api-method-parameter name="Authorization" type="string" required=true %}
Bearer `{access_token}` from authentication
{% endapi-method-parameter %}
{% endapi-method-headers %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```text

```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="get" host="https://owner-api.teslamotors.com" path="/api/1/energy\_sites/:site\_id/status" %}
{% api-method-summary %}
Site Summary
{% endapi-method-summary %}

{% api-method-description %}
**Warning:** This method seems to always return `404 Not Found`. It may have been removed from the API.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name=":site\_id" type="integer" required=true %}
The `{energy_site_id}` from the products list
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}

{% api-method-headers %}
{% api-method-parameter name="Authorization" type="string" required=true %}
Bearer `{access_token}` from authentication
{% endapi-method-parameter %}
{% endapi-method-headers %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=404 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```markup
<!DOCTYPE html>
<html>
<head>
  <title>The page you were looking for doesn't exist (404)</title>
  <style type="text/css">
    body { background-color: #fff; color: #666; text-align: center; font-family: arial, sans-serif; }
    div.dialog {
      width: 25em;
      padding: 0 4em;
      margin: 4em auto 0 auto;
      border: 1px solid #ccc;
      border-right-color: #999;
      border-bottom-color: #999;
    }
    h1 { font-size: 100%; color: #f00; line-height: 1.5em; }
  </style>
</head>

<body>
  <!-- This file lives in public/404.html -->
  <div class="dialog">
    <h1>The page you were looking for doesn't exist.</h1>
    <p>You may have mistyped the address or the page may have moved.</p>
  </div>
</body>
</html>

```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

