# Instagram followers

## Script

```javascript
// v0.1
const username = 'haus_automation';
const followerCountObjId = '0_userdata.0.web.instagramFollowers';

function refreshInstagramFollowers() {
    httpGet(
        `https://i.instagram.com/api/v1/users/web_profile_info/?username=${username}`,
        {
            headers: {
                'User-Agent': 'Instagram 76.0.0.15.395 Android (24/7.0; 640dpi; 1440x2560; samsung; SM-G930F; herolte; samsungexynos8890; en_US; 138226743)',
            }
        },
        (err, response) => {
            if (!err) {
                try {
                    const data = JSON.parse(response.data);
                    const followerCount = data.data.user.edge_followed_by.count;

                    setState(followerCountObjId, { val: followerCount, ack: true });
                } catch (err) {
                    console.error(err);
                }
            }
        }
    );
}

schedule('0 */6 * * *', refreshInstagramFollowers);

createState(followerCountObjId, { type: 'number', role: 'value', read: true, write: false }, refreshInstagramFollowers);
```

[![ioBroker Master Kurs](https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png?2024)](https://haus-automatisierung.com/iobroker-kurs/?refid=iobroker-scripts)
