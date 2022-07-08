# Get all stopped instances

```javascript
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

function refreshList() {
    const resultList = [];

    for (var i in aliveIds) {
        const aliveStateId = aliveIds[i];
        const aliveState = getState(aliveStateId);

        // Offline adapters
        if (!aliveState.val) {
            const instanceObj = getObject(aliveStateId.replace('.alive', ''));

            resultList.push({
                'instance': getAdapterName(instanceObj),
                'offlineSince': formatDate(aliveState.ts, 'TT.MM.JJJJ SS:mm:ss')
            });

            console.log(aliveState);
        }
    }

    setState('0_userdata.0.stoppedInstances', JSON.stringify(resultList, null, 2), true);
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
        'zh-cn': '线性事件'
    },
    type: 'string',
    role: 'json',
    read: true,
    write: false,
    def: '[]'
}, () => {
    on({id: aliveIds, change: 'ne'}, async (obj) => {
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
    "instance": "Geburtstage",
    "offlineSince": "08.07.2022 00:00:16"
  },
  {
    "instance": "Deutsche Feiertage",
    "offlineSince": "08.07.2022 00:00:09"
  },
  {
    "instance": "iCal Kalender",
    "offlineSince": "08.07.2022 20:30:11"
  }
]
```
