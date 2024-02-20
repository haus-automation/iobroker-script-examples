# Inline image to telegram

## Requirements

- [telegram adapter](https://github.com/iobroker-community-adapters/ioBroker.telegram)

## Script

```javascript
// v0.1
const fs = require('node:fs');

function sendBase64Img(str) {
    const imgPath = '/opt/iobroker/iobroker-data/telegram.tmp.png';
    const data = str.split(',')[1];

    fs.writeFileSync(imgPath, Buffer.from(data, 'base64'));

    const stats = fs.statSync(imgPath);
    console.log(`Saved tmp file to ${imgPath} - size: ${stats.size}`);

    sendTo('telegram.0', 'send', {
        user: 'klein0r',
        text: imgPath,
        caption: 'Just an image',
    });
}

// Test

const base64Img = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAIAAAAlC+aJAAAABGdBTUEAALGPC/xhBQAAAAlwSFlzAAAOwgAADsIBFShKgAAAABh0RVh0U29mdHdhcmUAcGFpbnQubmV0IDQuMS4xYyqcSwAAB8xJREFUaEPtmflzVFUWx+ePsYQk3e/1ls7rTu/pdHrvFivOsAlBE2IQUAajjiw6iqhYoowgg1gzLKXEsSQ6YhDDMk4CcTQJhWAgCdmgZlyKfZOZX+bz+nY63Z2gI4F5UtVV3wrd75177vmec89ym1+d70vemUic74vxFwJ8ukMR5y8E+OdORJrGHR0BFQUCWqNAQGsUCGiNAgGtUSCgNQoEtEaBgNYoENAaBQJao0BAaxQIaI0CAa1x6wmc601cOgnilwcSgA98Fc+zxTJPLvQjhnD822Ox3o7wsbbQSHf0PA+HkpcGcuQnxG2JwJKH7fdPt86ZYZ07s2zmr61LF9r3Nwcu9ufIpHkOJI63R9avcdfPU35TbY1HLdGw+d6kpWZW2cpGR8sOv8rkR2ncegJ4NBgw6yVJkiXZIBcVS+GQuXlrhYiDwLne+NWhBJ7eusE7e7q1wmc0mWW9LJfogaSTJINRttsM98TNyx8r79obvFEMwW0hEA6aZQNGSEaTXKKTcKpKYNSRqu8HEsOdkU2vuKsqTVOLJdhiMcImswEYTUCWDOpaS6nc8KDS2Ro8e0L9Qfdc7l7gthCIhCwZg3R6ORaxfLDNn3Wg4xdPJra/4VOtL9JjrrCbJZKsxoG4ibW8whE6nbRkga2nPaz+KD0uCBoQ4Dwc/SxcO1e5awrWC0Nl5EtLDT6vqcJnstmMfE1pgJUEJUupgSw6eyJ2IXcvoAGBa8PJN9d6K/0mnZS2HmD0Qw8of3jBvWGN53dLyhNRC1lBQHiF5G8X2rtaQ2eOj/23QAYaEPj36WTjYrtsVM89AnwotRoeechGGSWzLw8mrp9K7n638r57S80WeVrS0rytgqRHLWvH57EGBP5z+p5F9TYSVAhw4h0O456/BPIcvH9n4KVnnEOd0eyH46ENgYXzleKSdAJwTsrLjZSpMz05boYPMcH3YtWNoM0RerTBptNLhpQAaUqO1tWUnTocvTKkLs9Wlfl8I2hA4IeR5CvPuVxOI75HADFgsxkaahXEvjkauz6S7r4Tdq48aEDgymDiQHNgenXplCLSQHQutWiSDNMSlmVLy9/+o/frtvDVoeTV4eRPBkEDAhf64t9+HVu9wmmxyKKSCiBfrFO7cjBgokxt3+jrag0iT10SCyeEBgQ4GLSCQy3BxfU2BOhTmWYsaMBhSpFesRkW1yvv/anixMEwnVssFBqyoQEBQG25NpQ4+HFwQa2tTFEraUY+A4YIAkIrePpxx5H9oTNiFvol9AEBTKFt9R4KP/eU0+sxWa1wUDtDCmM01FFCku+fae1sDZ1NldQ8DpoRAMIU6n33vhBjs8ulDtWpVTk0UMVkXjun7MiB0OVBOOR0Bu0JAEblb45Gj/09vGmtJxm3yEY1DbIJAL0svbXO888jsYv9vxgC2WAVrsU+xuamt3ycGXEfQgNMAI1v3mxrR0vw2nDOQi1zAEnKS/Ydha+0MCaIL1uDq5Y50YAeEQfCYrcbP9hWQR/MyIObJ8BmPwwnx7fMnySgmp4aqr/viZ/ujqIhezlAM27+Yk+wrkbhLAkCzB1cUps2+65PngAm0ibp+bve8XNwU9EfA4amCRjVvSHAlXLnlrErJcWHJdj32mr3a6td/R0RenNmOYAPrIY6I0wcJbp0BHAHZHa8WXF9+GYJCD+pQe+Nd+4NbX7VE42YX1jp7P88gsOwCfCWATjg504si1mNsZnbye6mSl6xnHztaQu/v8W/oE7BJrvd8Ooq10h3hDGOm1paSX+CcYhpYukiOxd8QQCPMDtNNgJsj2Namirr5ik0mqISyeM2vvGym4cc3O+OxSBDJWEy460x5baiEv2MautXf0sH6vjB0IpGB/cv1qaui5LbaVj3ootL5kh3lEOVqkixgS8iWzd43W7iOJoDBnXgY+q++RzAhf/6KvrOJp9wBtujl+LAXMnosnOL/8Pt/ueXO7idYLdIgFQbkpj+1bE+FcCWHX7iI344QYDyAlWsrJ5m5T75ybuVn31Y1bS5gnkbPdATMgA9c2ZaD35cNakqRNP59L0AN0DsSzXO9OnEGowA6tFPmS6es6vPZ1q/xkPcRbW5NBB/9imn04lo+kY2auKoEhjxctQFQP1qVH8vItSnuqOT6gMX+uMjXdE1v3emlI4RyMOoWUyX+kcabJwrNXPSSuKnD8caF9kYciRD+nwLAuMh3rIXEauZbT28b3KdWCTxuT7SILpqmQPVeAXtYps84OC7p+prZpUd2Bm4dHJsV6Fk8MvIutUur8d4dxEqsDV/uQDKOUWc0ppZ1n/sCd6CWUgsJh1x6vaN3hnVpfRLDCUjmRwpOHyYWqw+cToMzzzp6Gip+r4nhnzerhQcIsmJX/6Yw+c1crNhfhZKMnp4guMTUfP6l9xde9Vf5sbrAT+PABAqKOrUira/Vv35de/Kx8sbHlTq5iq1c5X5NcqjC2xrV7mat/qZ43F8quzmaBAQ1xQGh11v+ze+7FnRWF7/gELnAuh5eL7t6SccDD/73g9AlbZA75tQz88mkAGWUdHwSl9H5PPdwfaPqto+qmLE794XhBu3QWHihLsCnrOWFkY7x7snDoW54rTvqmrfFUQPbY4gw58t8maQPNw8AUBeYgTuEfdXgSuDSdGz/neMV8Jn1eua/Crxf0aBgLboS/wXkZ9EqXSS7ZwAAAAASUVORK5CYII=';
sendBase64Img(base64Img);

```

[![ioBroker Master Kurs](https://haus-automatisierung.com/images/ads/ioBroker-Kurs.png?2024)](https://haus-automatisierung.com/iobroker-kurs/?refid=iobroker-scripts)
