# Get min and max temperature of yr.no weather

## Requirements

- [yr adapter](https://github.com/ioBroker/ioBroker.yr)

## Script

```javascript
on({id: 'yr.0.forecast.info.object', change: 'ne'}, async (obj) => {
    const dateToday = new Date().getDate();

    const forecastObj = JSON.parse(obj.state.val);
    const forecastToday = forecastObj.properties.timeseries.filter(t => new Date(t.time).getDate() === dateToday);

    const minMax = forecastToday.reduce((acc, t) => {
        const val = t.data.instant.details.air_temperature;

        acc[0] = acc[0] === undefined || val < acc[0] ? val : acc[0];
        acc[1] = acc[1] === undefined || val > acc[1] ? val : acc[1];

        return acc;
    }, []);

    setState(
        'mqtt.0.service.weather.forecast.temperature.today',
        minMax
            .map(temp => `${temp} °C`)
            .join(' to ')
        );
});
```