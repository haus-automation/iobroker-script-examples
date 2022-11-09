
# Migrate history data to InfluxDB

## Requirements

- [history adapter](https://github.com/ioBroker/ioBroker.history)
- [influxdb adapter](https://github.com/ioBroker/ioBroker.influxdb)

## Script

```javascript
const migrateObjId = 'yr.0.forecastHourly.0h.air_temperature';
const stepSize = 60 * 60 * 24 * 7 * 1000; // select 1 week per run
const batchLimit = 1000;
const now = Date.now();

sendTo('history.0', 'getHistory', {
    id: migrateObjId,
    options: {
        start: 1,
        end: now,
        aggregate: 'none',
        returnNewestEntries: false,
        limit: 1,
    }
}, async (historyData) => {
    if (historyData.result.length > 0) {
        let migratedValues = 0;
        let startTs = historyData.result[0].ts;
        console.log(`Running migration of ${migrateObjId} (starting @ ${formatDate(startTs, 'DD.MM.YYYY hh:mm:ss')})`);

        while (startTs < now) {
            migratedValues += await migrateRange(startTs, startTs + stepSize);
            startTs += stepSize;
        }

        console.log(`Finished migration of ${migrateObjId} (saved ${migratedValues} values)`);
    } else {
        console.error(`Cannot migrate ${migrateObjId} - no data found`);
    }
});

async function migrateRange(startTs, endTs) {
    return new Promise((resolve) => {
        sendTo('history.0', 'getHistory', {
            id: migrateObjId,
            options: {
                start: startTs,
                end: endTs,
                aggregate: 'none',
                limit: batchLimit,
                ignoreNull: true, // null is not supported by InfluxDB
                // Get more information!
                addId: true,
                from: true,
                ack: true,
                q: true,
            }
        }, (historyData) => {
            const valueCount = historyData.result.length;
            console.log(`Saving batch of ${migrateObjId} / ${valueCount} values / ${formatDate(startTs, 'DD.MM.YYYY hh:mm:ss')} - ${formatDate(endTs, 'DD.MM.YYYY hh:mm:ss')}`);

            if (valueCount === batchLimit) {
                console.warn(`Result returned ${valueCount} values - set smaller step size to migrate all information!`);
            }
            if (valueCount > 0) {
                sendTo('influxdb.0', 'storeState', {
                    id: migrateObjId,
                    state: historyData.result
                }, () => {
                    resolve(valueCount);
                });
            } else {
                resolve(0);
            }
        });
    });
}
```

[![ioBroker Master Kurs](https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png)](https://haus-automatisierung.com/iobroker-kurs/?refid=iobroker-scripts)
