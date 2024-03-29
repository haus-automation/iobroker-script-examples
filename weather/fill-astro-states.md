# Fill astro states

Since JavaScript-Adapter version `>= 7.3.0` the states for the **current day** are stored in `variables.astro.*`.

## Script

```javascript
// v0.5
const suncalc = require('suncalc2');

const prefix = '0_userdata.0.suncalc';
const daysFuture = 1; // 0 = today, 1 = tomorrow, ...

async function fillAstroStates() {
    try {
        const d = new Date();
        if (daysFuture > 0) {
            d.setDate(d.getDate() + daysFuture);
        }

        console.log(`Refreshing astro states for ${d.toISOString()}`);

        const systemConfig = await getObjectAsync('system.config');
        const times = suncalc.getTimes(d, systemConfig.common.latitude, systemConfig.common.longitude);

        for (const [name, time] of Object.entries(times)) {
            const h = time.getHours();
            const m = time.getMinutes();

            const timeFormatted = `${String(h).padStart(2, '0')}:${String(m).padStart(2, '0')}`;
            const objId = `${prefix}.${name}`;

            if (!await existsObjectAsync(objId)) {
                await createStateAsync(objId, timeFormatted, { name: `Astro ${name}`, type: 'string', role: 'value' });
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

[![ioBroker Master Kurs](https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png?2024)](https://haus-automatisierung.com/iobroker-kurs/?refid=iobroker-scripts)
