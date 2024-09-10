# Get all disconnected instances

## Script

```javascript
// v0.1
const connectedSelector = 'system.adapter.*.*.connected';
const resultObjId = '0_userdata.0.disconnectedInstances';
let instanceTrigger = null;

function getAdapterName(obj) {
    if (typeof obj.common.titleLang === 'object') {
        if (Object.prototype.hasOwnProperty.call(obj.common.titleLang, 'de')) {
            return obj.common.titleLang.de;
        } else {
            return obj.common.titleLang.en;
        }
    }

    return obj.common.title;
}

async function refreshList(connectedIds) {
    try {
        const resultList = [];

        for (const connectedStateId of connectedIds) {
            const aliveState = await getStateAsync(connectedStateId);

            // Offline instances
            if (aliveState && !aliveState.val) {
                const instanceId = aliveStateId.replace('.alive', '');
                const instanceObj = await getObjectAsync(instanceId);

                // Ignore scheduled adapters
                if (instanceObj.common.mode == 'daemon') {
                    resultList.push({
                        'name': getAdapterName(instanceObj),
                        'instance': instanceId.replace('system.adapter.', ''),
                        'offlineSince': formatDate(aliveState.ts, 'TT.MM.JJJJ SS:mm:ss')
                    });
                }
            }
        }

        await setStateAsync(resultObjId, { val: JSON.stringify(resultList, null, 2), ack: true });
    } catch (err) {
        await setStateAsync(resultObjId, { val: JSON.stringify([]), ack: true });

        console.error(err);
    }
}

async function createTriggersAndUpdate() {
    const connectedIds = Array.prototype.slice.apply($(connectedSelector));

    // refresh now
    refreshList(connectedIds);

    // Remove old trigger and recreate a new trigger on all alive IDs
    if (instanceTrigger) {
        unsubscribe(instanceTrigger);
    }
    instanceTrigger = on({ id: connectedIds, change: 'ne' }, () => {
        refreshList(connectedIds);
    });
}

createState(resultObjId, {
    name: {
        en: 'Offline instances',
        de: 'Offline-Instanzen',
        ru: 'Оффлайн инстанции',
        pt: 'Exemplos offline',
        nl: 'Offline instituut',
        fr: 'Nombre de cas',
        it: 'Sezioni offline',
        es: 'Casos fuera de servicio',
        pl: 'Instancja offline',
        uk: 'Оффлайн екземпляри',
        'zh-cn': '线性事件',
    },
    type: 'string',
    role: 'json',
    read: true,
    write: false,
    def: '[]',
}, () => {
    // Refresh connectedIds (every 10min) - maybe new instances have been created / installed
    schedule('*/10 * * * *', () => {
        createTriggersAndUpdate();
    });

    // Run on startup
    createTriggersAndUpdate();
});
```

Result

```json
[
  {
    "name": "Awtrix Light",
    "instance": "awtrix-light.0",
    "offlineSince": "21.12.2023 12:18:44"
  },
  {
    "name": "BackItUp",
    "instance": "backitup.0",
    "offlineSince": "02.11.2023 10:38:53"
  },
  {
    "name": "ModBus",
    "instance": "modbus.0",
    "offlineSince": "03.03.2023 13:05:41"
  },
  {
    "name": "MQTT-Client",
    "instance": "mqtt-client.0",
    "offlineSince": "07.01.2024 21:17:07"
  },
  {
    "name": "Benachrichtigungsmanager",
    "instance": "notification-manager.0",
    "offlineSince": "09.12.2023 15:12:27"
  },
  {
    "name": "REST API",
    "instance": "rest-api.0",
    "offlineSince": "31.01.2024 14:20:26"
  },
  {
    "name": "ZigBee",
    "instance": "zigbee.0",
    "offlineSince": "06.02.2024 14:39:04"
  }
]
```

[![ioBroker Master Kurs](https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png?2024)](https://haus-automatisierung.com/iobroker-kurs/?refid=iobroker-scripts)
