# Republish the current value via interval

## Script

```javascript
// v0.1
const repeatObjId = 'mqtt.0.home.test';

const topic = getObject(repeatObjId)?.native?.topic;
let repeatValue = getState(repeatObjId).val;

on({ id: repeatObjId, ack: true }, (obj) => {
    repeatValue = obj.state.val;
});

setInterval(() => {
    sendTo('mqtt.0', 'sendMessage2Client', { topic, message: repeatValue });
}, 1000);
```

[![ioBroker Master Kurs](https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png?2024)](https://haus-automatisierung.com/iobroker-kurs/?refid=iobroker-scripts)
