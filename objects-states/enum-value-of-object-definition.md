# Get enum value of object definition

## Script

```javascript
function getStateOptions(objId) {
    const obj = getObject(objId);

    let states = obj.common?.states;
    if (states) {
        if (typeof states === 'string' && states[0] === '{') {
            try {
                states = JSON.parse(states);
            } catch (ex) {
                console.error(`Cannot parse states: ${states}`);
                states = null;
            }
        } else if (typeof states === 'string') {
            const parts = states.split(';');
            states = {};
            for (let p = 0; p < parts.length; p++) {
                const s = parts[p].split(':');
                states[s[0]] = s[1];
            }
        } else if (Array.isArray(states)) {
            const result = {};
            if (obj.common.type === 'number') {
                states.forEach((value, key) => result[key] = value);
            } else
            if (obj.common.type === 'string') {
                states.forEach(value => result[value] = value);
            } else if (obj.common.type === 'boolean') {
                result['false'] = states[0];
                result['true'] = states[1];
            }

            return result;
        }
    }

    return states;
}

function getStateOptionByValue(objId) {
    if (existsObject(objId)) {
        const value = getState(objId).val;
        const stateOptions = getStateOptions(objId);

        if (stateOptions && stateOptions?.[value]) {
            return stateOptions[value];
        } else {
            return value;
        }
    } else {
        console.error(`Object with ID "${objId}" doesn't exist!`);
        return undefined;
    }
}
```

## Example

### Object definition

```json
{
  "common": {
    "type": "number",
    "name": "States Test",
    "role": "state",
    "states": {
      "0": "test 0",
      "1": "test 1",
      "2": "test 2"
    }
  },
  "native": {},
  "type": "state",
  "_id": "0_userdata.0.test",
  "acl": {
    "object": 1636,
    "state": 1636,
    "owner": "system.user.admin",
    "ownerGroup": "system.group.administrator"
  },
  "from": "system.adapter.admin.0",
  "user": "system.user.admin",
  "ts": 1664261344435
}
```

### Script

```javascript
setState('0_userdata.0.test', 2, true);
console.log(getStateOptionByValue('0_userdata.0.test')); // Ouput: "test 2"
```

[![ioBroker Master Kurs](https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png)](https://haus-automatisierung.com/iobroker-kurs/?refid=iobroker-scripts)
