# Delete time range of measurement in InfluxDB

This scripts logs cpu and memory to InfluxDB (for all running instances)

## Requirements

- [influxdb adapter](https://github.com/ioBroker/ioBroker.influxdb)

## Script

```javascript
// v0.1
const influxDbInstance = 'influxdb.0';
const token = 'cIpimeNDz....';
const measurement = 'youtube.0.channels.haus_automation.subscribers';

// Date in local time zone!
const fromYear = 2023;
const fromMonth = 10;
const fromDay = 1;
const fromHour = 0;
const fromMinute = 0;
const fromSecond = 0;

const toYear = 2023;
const toMonth = 10;
const toDay = 5;
const toHour = 23;
const toMinute = 59;
const toSecond = 59;

const deleteFrom = new Date(fromYear, fromMonth - 1, fromDay, fromHour, fromMinute, fromSecond, 0);
const deleteTo = new Date(toYear, toMonth - 1, toDay, toHour, toMinute, toSecond, 999);

async function start() {
    const influxDbInstanceConfig = await getObjectAsync(`system.adapter.${influxDbInstance}`);

    const protocol = influxDbInstanceConfig.native.protocol;
    const host = influxDbInstanceConfig.native.host;
    const port = influxDbInstanceConfig.native.port;
    const org = influxDbInstanceConfig.native.organization;
    const bucket = influxDbInstanceConfig.native.dbname;

    console.log(`Deleting data in bucket "${bucket}" of org "${org}" from ${formatDate(deleteFrom, 'TT.MM.JJJJ SS:mm:ss')} (${deleteFrom.toISOString()}) to ${formatDate(deleteTo, 'TT.MM.JJJJ SS:mm:ss')} (${deleteTo.toISOString()})`);

    httpPost(
        `${protocol}://${host}:${port}/api/v2/delete?bucket=${bucket}&org=${org}`,
        {
            predicate: `_measurement="${measurement}"`,
            start: deleteFrom.toISOString(),
            stop: deleteTo.toISOString(),
        },
        {
            timeout: 10000,
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Token ${token}`
            }
        },
        (error, response) => {
            if (!error) {
                console.log(`Deletion finished with code ${response.statusCode}`);
            } else {
                console.error(error);
            }

            stopScript(scriptName); // stop this script
        }
    );
}

start();
```

[![ioBroker Master Kurs](https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png?2024)](https://haus-automatisierung.com/iobroker-kurs/?refid=iobroker-scripts)
