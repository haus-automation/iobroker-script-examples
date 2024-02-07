# Get all stopped instances

## Script

```javascript
// v0.3
const aliveSelector = 'system.adapter.*.*.alive';
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

async function refreshList(aliveIds) {
    try {
        const resultList = [];

        for (const aliveStateId of aliveIds) {
            const aliveState = await getStateAsync(aliveStateId);

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

        await setStateAsync('0_userdata.0.stoppedInstances', { val: JSON.stringify(resultList, null, 2), ack: true });
    } catch (err) {
        await setStateAsync('0_userdata.0.stoppedInstances', { val: JSON.stringify([]), ack: true });

        console.error(err);
    }
}

async function createTriggersAndUpdate() {
    const aliveIds = Array.prototype.slice.apply($(aliveSelector));

    // refresh now
    refreshList(aliveIds);

    // Remove old trigger and recreate a new trigger on all alive IDs
    if (instanceTrigger) {
        unsubscribe(instanceTrigger);
    }
    instanceTrigger = on({ id: aliveIds, change: 'ne' }, () => {
        refreshList(aliveIds);
    });
}

createState('0_userdata.0.stoppedInstances', {
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
    // Refresh aliveIds (every 10min) - maybe new instances have been created / installed
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
