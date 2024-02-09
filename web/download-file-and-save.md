# Download file and save to files tab

## Script

```javascript
  // v0.1
const axios = require('axios').default;

const url = 'https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png';
const fileName = 'myFile.png';

axios({
    method: 'get',
    url,
    responseType: 'arraybuffer',
}).then((response) => {
    if (response.status === 200) {
        writeFile('0_userdata.0', fileName, response.data, (error) => {
            if (!error) {
                console.log('saved file');
            }
        });
    }
});
```

[![ioBroker Master Kurs](https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png?2024)](https://haus-automatisierung.com/iobroker-kurs/?refid=iobroker-scripts)
