# History as mail (with csv attachment)

## Requirements

- [history adapter](https://github.com/ioBroker/ioBroker.history)
- [eMail adapter](https://github.com/iobroker-community-adapters/ioBroker.email)

## Script

```javascript
// v0.1
const objId = 'yr.0.forecastHourly.0h.air_temperature';
const mailFrom = 'iobroker@mydomain.com';
const mailTo = 'yourmail@mydomain.com';

const csvDateFormat = 'DD.MM.YYYY hh:mm:ss';

const fileName = `${objId}.csv`;
const filePath = `/opt/iobroker/iobroker-data/${fileName}`;

var fs = require('fs');

// Send every monday @ 00:00
schedule('0 0 * * 1', async () => {
    const end = new Date().getTime();
    const start = end - (60 * 60 * 24 * 7 * 1000); // last 7 days

    getHistory('history.0', {
        id: objId,
        start: start,
        end: end,
        aggregate: 'none',
        timeout: 2000
    }, (err, result) => {
        if (err) {
            console.error(`Unable to get history of ${objId}: ${err}`);
        } else if (result) {
            let csvData = ['timestamp;value'];

            for (var i = 0; i < result.length; i++) {
                csvData.push(`${formatDate(result[i].ts, csvDateFormat)};${result[i].val}`);
            }

            fs.writeFileSync(filePath, csvData.join("\n"));

            sendTo('email.0', {
                from: mailFrom,
                to: mailTo,
                subject: `History-Daten von ${objId}`,
                text: `Hier sind die History-Daten von ${formatDate(start, csvDateFormat)} bis ${formatDate(end, csvDateFormat)}`,
                attachments: [
                    {
                        path: filePath
                    }
                ]
            }, (err, result) => {
                if (err) {
                    console.error(`Unable to send email: ${err}`);
                }
                fs.unlinkSync(filePath);
            });
        }
    });
});
```

[![ioBroker Master Kurs](https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png?2024)](https://haus-automatisierung.com/iobroker-kurs/?refid=iobroker-scripts)
