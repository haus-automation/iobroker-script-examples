# Update Geo Position in System Config

## Script

```javascript
// v0.1
function updateGeoPosition(lon, lat) {
    extendObject(
        'system.config',
        {
            common: {
                longitude: lon,
                latitude: lat
            }
        }
    );
}

updateGeoPosition(8.2, 51.2);
```

## Script (not smart)

```javascript
// v0.1
function updateGeoPosition(lon, lat) {
    const systemConfig = getObject('system.config');
    let changedConfig = false;

    if (systemConfig.common.longitude != lon) {
        systemConfig.common.longitude = lon;
        changedConfig = true;
    }

    if (systemConfig.common.latitude != lat) {
        systemConfig.common.latitude = lat;
        changedConfig = true;
    }

    if (changedConfig) {
        setObject('system.config', systemConfig);
    }
}

updateGeoPosition(8.2, 51.2);
```

[![ioBroker Master Kurs](https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png?2024)](https://haus-automatisierung.com/iobroker-kurs/?refid=iobroker-scripts)
