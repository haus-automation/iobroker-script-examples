
# Migrate history data to InfluxDB (or SQL)

## Requirements

- [history adapter](https://github.com/ioBroker/ioBroker.history)
- [influxdb adapter](https://github.com/ioBroker/ioBroker.influxdb) or [sql adapter](https://github.com/ioBroker/ioBroker.sql)

## Script

```javascript
// v0.3
const targetInstance = 'influxdb.0'; // or 'sql.0'

const now = Date.now();
const oneHour = 60 * 60 * 1000;
const oneDay = oneHour * 24;
const oneWeek = oneDay * 7;

// EDIT THIS VALUES (IF NECESSARY)
const migrateObjId = 'yr.0.forecastHourly.0h.air_temperature';
const stepSize = oneWeek; // select per run
const batchLimit = 1000;

// DO NOT EDIT
(async (objId) => {
    const migrateRangeFunc = async (startTs, endTs) => {
        const historyData = await sendToAsync('history.0', 'getHistory', {
            id: objId,
            options: {
                start: startTs,
                end: endTs - 1,
                aggregate: 'none',
                limit: batchLimit,
                ignoreNull: false,
                addId: true,
                from: true,
                ack: true,
                q: true,
            }
        });

        const valueCount = historyData?.result.length;
        if (valueCount >= batchLimit) {
            console.warn(`Result returned ${valueCount} records - set smaller step size to migrate all information!`);
        }

        if (valueCount > 0) {
            const saveToDB = historyData.result.filter(r => r.val !== null); // NULL is not supported by InfluxDB
            console.log(`Saving batch of ${objId}: ${valueCount} (${saveToDB.length}) values / ${formatDate(startTs, 'DD.MM.YYYY hh:mm:ss.sss')} - ${formatDate(endTs - 1, 'DD.MM.YYYY hh:mm:ss.sss')}`);

            if (saveToDB.length > 0) {
                await sendToAsync(targetInstance, 'storeState', { id: objId, state: saveToDB });
            }
        }

        return valueCount;
    };

    const countResult = await sendToAsync('history.0', 'getHistory', {
        id: objId,
        options: {
            start: 1,
            end: now,
            aggregate: 'count',
            count: 1,
            ignoreNull: false,
        }
    });
    const recordCount = countResult.result.find(r => r.val !== null)?.val;

    if (recordCount && recordCount > 0) {
        const oldestRecordResult = await sendToAsync('history.0', 'getHistory', {
            id: objId,
            options: {
                start: 1,
                end: now,
                aggregate: 'none',
                ignoreNull: false,
                returnNewestEntries: false,
                limit: 100,
            }
        });

        if (oldestRecordResult?.result.length > 0) {
            let migratedValues = 0;
            let startTs = oldestRecordResult?.result[0].ts;
            console.log(`Running migration of ${objId} (starting @ ${formatDate(startTs, 'DD.MM.YYYY hh:mm:ss.sss')})`);

            while (startTs <= now) {
                migratedValues += await migrateRangeFunc(startTs, startTs + stepSize);
                startTs += stepSize;
            }

            console.log(`Finished migration of ${objId} (saved ${migratedValues} of ${recordCount} records)`);
            if (migratedValues < recordCount) {
                console.warn(`Missing records for ${objId}: ${recordCount - migratedValues}`);
            }
        } else {
            console.error(`Cannot migrate ${objId} - no data found`);
        }
    }
})(migrateObjId);
```

[![ioBroker Master Kurs](https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png?2024)](https://haus-automatisierung.com/iobroker-kurs/?refid=iobroker-scripts)
