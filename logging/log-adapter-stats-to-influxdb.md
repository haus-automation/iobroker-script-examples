
# Log adapter stats to InfluxDB

This scripts logs cpu and memory to InfluxDB (for all running instances)

## Requirements

- [influxdb adapter](https://github.com/ioBroker/ioBroker.influxdb)

## Script

```javascript
// v0.1
const aliveSelector = 'system.adapter.*.*.alive';
const additionalStates = ['cpu', 'memRss'];

let instanceTrigger = null;
let additionalTrigger = null;

function getInstanceByObjId(objId) {
    return objId.substring(0, objId.lastIndexOf('.'));
}

async function storeValue(objId, value) {
    if (!isNaN(value)) {
        await this.sendToAsync('influxdb.0', 'storeState', {
            id: objId,
            state: {
                ts: Date.now(),
                val: value,
                ack: true,
                from: this.name
            }
        });
    }
}

async function logZero(aliveStateId) {
    for (const additionalState of additionalStates) {
        const additionalId = aliveStateId.replace('.alive', `.${additionalState}`);
        await storeValue(additionalId, 0);
    }
}

async function refreshAlive() {
    const aliveIds = Array.prototype.slice.apply($(aliveSelector));
    const aliveInstances = []; // just log values of alive instances
    const additionalIds = [];

    for (const aliveStateId of aliveIds) {
        // Collect ids of additional states (for logging)
        for (const additionalState of additionalStates) {
            const additionalId = aliveStateId.replace('.alive', `.${additionalState}`);
            additionalIds.push(additionalId);
        }

        // Collect running instances once
        const aliveState = await getStateAsync(aliveStateId);
        if (aliveState && aliveState.val) {
            const instance = getInstanceByObjId(aliveStateId);
            aliveInstances.push(instance);
        } else {
            // log 0 when stopped
            await logZero(aliveStateId);
        }
    }

    console.log(`Alive instances: ${JSON.stringify(aliveInstances)}`);

    // Remove old trigger and recreate a new trigger on all alive IDs
    if (instanceTrigger) {
        unsubscribe(instanceTrigger);
    }
    instanceTrigger = on({ id: aliveIds, change: 'ne' }, (obj) => {
        const instance = getInstanceByObjId(obj.id);
        if (obj.state.val) {
            // is alive - add to list
            console.log(`Instances ${instance} has been started`);
            if (!aliveInstances.includes(instance)) {
                aliveInstances.push(instance);
            }
        } else {
            // instance stopped - remove from list
            console.log(`Instances ${instance} has been stopped`);
            const idx = aliveInstances.indexOf(instance);
            if (idx > -1) {
                aliveInstances.splice(idx, 1);
            }

            // log 0 when stopped
            logZero(obj.id);
        }

        console.log(`Alive instances (changed): ${JSON.stringify(aliveInstances)}`);
    });

    // Remove old trigger and recreate a new trigger for additional information
    if (additionalTrigger) {
        unsubscribe(additionalTrigger);
    }
    additionalTrigger = on({ id: additionalIds, change: 'any' }, (obj) => {
        const instance = getInstanceByObjId(obj.id);

        // just log values of running instances
        if (aliveInstances.includes(instance)) {
            storeValue(obj.id, obj.state.val);
        }
    });
}

// Refresh aliveIds (every 10min) - maybe new instances have been created / installed
schedule('*/10 * * * *', () => {
    refreshAlive();
});

// Run on startup
refreshAlive();
```

[![ioBroker Master Kurs](https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png)](https://haus-automatisierung.com/iobroker-kurs/?refid=iobroker-scripts)
