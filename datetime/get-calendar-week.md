# Get current calendar week

## Script

```javascript
// v0.1
const objIdWeekNumber = 'javascript.0.variables.calendarWeek';

function getWeekNumber() {
    const now = new Date();
    const d = new Date(Date.UTC(now.getFullYear(), now.getMonth(), now.getDate()));
    const dayNum = d.getUTCDay() || 7;
    d.setUTCDate(d.getUTCDate() + 4 - dayNum);

    const yearStart = new Date(Date.UTC(d.getUTCFullYear(), 0, 1));

    return Math.ceil(((d.getTime() - yearStart.getTime()) / 86400000 + 1) / 7);
}

async function updateWeekNumber() {
    const weekNumber = getWeekNumber();

    if (!await existsObjectAsync(objIdWeekNumber)) {
        await createStateAsync(objIdWeekNumber, weekNumber, { name: 'Current week number', type: 'number', role: 'value', read: true, write: false });
    } else {
        await setStateAsync(objIdWeekNumber, { val: weekNumber, ack: true });
    }
}

// Refresh every monday
schedule('0 0 * * 1', updateWeekNumber);

// Run immediately after script start
updateWeekNumber();
```

[![ioBroker Master Kurs](https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png?2024)](https://haus-automatisierung.com/iobroker-kurs/?refid=iobroker-scripts)
