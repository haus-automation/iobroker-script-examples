# Random joke from API to state

## Script

```javascript
// v0.1
const axios = require('axios').default;

const jokeObjId = '0_userdata.0.service.jokeOfTheDay';

async function update() {
    try {
        const response = await axios.get('https://witzapi.de/api/joke/?limit=1&language=de', {
            headers: { 'Content-Type': 'application/json' }
        });

        if (response.status == 200 && response.data.length > 0) {
            const jokes = response.data;
            setState(jokeObjId, { val: jokes[0].text, ack: true });
        }
    } catch (err) {
        console.error(err);
    }
}

createState(jokeObjId, { type: 'string', read: true, write: false }, () => {
    update(); // Refresh now
    schedule('0 1 * * *', update); // Refresh every day at 01:00
});
```

[![ioBroker Master Kurs](https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png)](https://haus-automatisierung.com/iobroker-kurs/?refid=iobroker-scripts)
