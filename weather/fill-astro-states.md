# Fill astro states

## Script

```javascript
// v0.3
const suncalc = require('suncalc2');

const prefix = 'javascript.0';

async function fillAstroStates() {
    try {
        const systemConfig = await getObjectAsync('system.config');
        const times = suncalc.getTimes(new Date(), systemConfig.common.latitude, systemConfig.common.longitude);

        for (var t in times) {
            const h = times[t].getHours();
            const m = times[t].getMinutes();

            const timeFormatted = `${h < 10 ? '0' + h : h}:${m < 10 ? '0' + m : m}`;
            const objId = `${prefix}.suncalc.times.${t}`;

            if (!await existsObjectAsync(objId)) {
                await createStateAsync(objId, timeFormatted, { name: `Astro ${t}`, type: 'string', role: 'value' });
            } else {
                await setStateAsync(objId, { val: timeFormatted, ack: true });
            }
        }
    } catch (err) {
        console.error(err);
    }
}

// Refresh every day at 00:01
schedule('1 0 * * *', fillAstroStates);

// Run immediately after script start
fillAstroStates();
```

[![ioBroker Master Kurs](https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png)](https://haus-automatisierung.com/iobroker-kurs/?refid=iobroker-scripts)
