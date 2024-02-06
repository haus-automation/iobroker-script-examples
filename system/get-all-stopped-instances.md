# Get all stopped instances

## Script

```javascript
// v0.2
const aliveIds = Array.prototype.slice.apply($('system.adapter.*.*.alive'));

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

async function refreshList() {
    try {
        const resultList = [];

        for (let aliveStateId of aliveIds) {
            const aliveState = await getStateAsync(aliveStateId);

            // Offline instances
            if (aliveState && !aliveState.val) {
                const instanceId = aliveStateId.replace('.alive', '');
                const instanceObj = await getObjectAsync(instanceId);

                resultList.push({
                    'name': getAdapterName(instanceObj),
                    'instance': instanceId.replace('system.adapter.', ''),
                    'offlineSince': formatDate(aliveState.ts, 'TT.MM.JJJJ SS:mm:ss')
                });
            }
        }

        await setStateAsync('0_userdata.0.stoppedInstances', { val: JSON.stringify(resultList, null, 2), ack: true });
    } catch (err) {
        await setStateAsync('0_userdata.0.stoppedInstances', { val: JSON.stringify([]), ack: true });

        console.error(err);
    }
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
    on({ id: aliveIds, change: 'ne' }, () => {
        refreshList();
    });

    // Init
    refreshList();
});
```

Result

```json
[
  {
    "name": "Awtrix Light",
    "instance": "awtrix-light.0",
    "offlineSince": "27.07.2023 13:56:17"
  },
  {
    "name": "Geburtstage",
    "instance": "birthdays.0",
    "offlineSince": "08.09.2023 00:00:09"
  },
  {
    "name": "DasWetter.com",
    "instance": "daswetter.0",
    "offlineSince": "08.09.2023 15:30:02"
  },
  {
    "name": "DWD",
    "instance": "dwd.0",
    "offlineSince": "08.09.2023 15:05:01"
  },
  {
    "name": "Feiertage (AT, CH, DE, IT)",
    "instance": "feiertage.0",
    "offlineSince": "08.09.2023 09:25:41"
  },
  {
    "name": "iCal Calendar",
    "instance": "ical.0",
    "offlineSince": "08.09.2023 00:00:10"
  },
  {
    "name": "ModBus",
    "instance": "modbus.0",
    "offlineSince": "03.03.2023 13:05:41"
  },
  {
    "name": "REST API",
    "instance": "rest-api.0",
    "offlineSince": "12.07.2023 15:48:37"
  },
  {
    "name": "Youtube",
    "instance": "youtube.0",
    "offlineSince": "08.09.2023 00:00:24"
  },
  {
    "name": "yr.no Wetter",
    "instance": "yr.0",
    "offlineSince": "08.09.2023 15:43:12"
  },
  {
    "name": "ZigBee",
    "instance": "zigbee.0",
    "offlineSince": "26.05.2023 12:30:48"
  }
]
```

[![ioBroker Master Kurs](https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png?2024)](https://haus-automatisierung.com/iobroker-kurs/?refid=iobroker-scripts)
