# Add object id to enum

## Script

```javascript
// v0.2
function addToEnum(enumId, objId) {
    if (existsObject(objId)) {
        if (existsObject(enumId)) {
            const enumObj = getObject(enumId);

            if (enumObj.type === 'enum') {
                if (!enumObj.common.members.includes(objId)) {
                    enumObj.common.members.push(objId);
                    setObject(enumId, enumObj);
                }

                return true;
            } else {
                throw new Error(`Target object with ID ${enumId} is not an enum`);
            }
        } else {
            throw new Error(`Enum with ID ${enumId} does not exist`);
        }
    } else {
        throw new Error(`Object with ID ${objId} does not exist`);
    }
}

// Test
addToEnum('enum.rooms.living_room', '0_userdata.0.mydate');
```

## Script (smarter?)

```javascript
// v0.2
function addToEnum(enumId, objId) {
    if (existsObject(objId)) {
        if (existsObject(enumId)) {
            const enumObj = getObject(enumId);

            if (enumObj.type === 'enum') {
                if (!enumObj.common.members.includes(objId)) {
                    extendObject(enumId, {
                        common: {
                            members: Array.prototype.concat(enumObj.common.members, [objId])
                        }
                    });
                }

                return true;
            } else {
                throw new Error(`Target object with ID ${enumId} is not an enum`);
            }
        } else {
            throw new Error(`Enum with ID ${enumId} does not exist`);
        }
    } else {
        throw new Error(`Object with ID ${objId} does not exist`);
    }
}

// Test
addToEnum('enum.rooms.living_room', '0_userdata.0.mydate');
```

[![ioBroker Master Kurs](https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png?2024)](https://haus-automatisierung.com/iobroker-kurs/?refid=iobroker-scripts)
