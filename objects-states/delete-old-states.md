# Delete old states

Deletes all states in ``0_userdata.0.*`` which haven't been updated since 180 days.

## Script

```javascript
// v0.1
const allObjectIds = $('0_userdata.0.*');
const deleteOlderThan = Date.now() - (1000 * 60 * 60 * 24 * 180); // 180 days

console.log(`Deleting all states which were not updated since ${formatDate(deleteOlderThan, 'DD.MM.YYYY hh:mm:ss')}`)

async function deleteOld() {
    for (const objId of allObjectIds) {
        const obj = await getObjectAsync(objId);
        if (obj && obj.type === 'state') {
            const state = await getStateAsync(objId);
            if (typeof state === 'object' && state.ts < deleteOlderThan) {
                console.log(`Deleting state with id ${objId} - not updated since ${formatDate(state.ts, 'DD.MM.YYYY hh:mm:ss')}`);
                //await deleteObject(objId);
            }
        }
    }
}

deleteOld();
```

[![ioBroker Master Kurs](https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png?2024)](https://haus-automatisierung.com/iobroker-kurs/?refid=iobroker-scripts)
