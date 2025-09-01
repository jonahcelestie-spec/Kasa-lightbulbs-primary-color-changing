# Kasa-lightbulbs-primary-color-changing
Can  make your lights change colors based on the primary color of you computer ( need a pc that can run termanil)
YOU NEED NODE AND PYTHON 

1: go in termainal and run `npm install tplink-smarthome-api screenshot-desktop get-image-colors`

2: go on notepad and paste in this code

```

const { Client } = require('tplink-smarthome-api');
const screenshot = require('screenshot-desktop');
const getColors = require('get-image-colors');

const client = new Client();
const bulbIPs = ['192.168.0.xxx', '192.168.0.xxx']; (your kasa ip you can use double or just use one)

function rgbToHSB([r, g, b]) {
  r /= 255; g /= 255; b /= 255;
  const max = Math.max(r, g, b), min = Math.min(r, g, b);
  const delta = max - min;

  let hue = 0;
  if (delta) {
    if (max === r) hue = ((g - b) / delta) % 6;
    else if (max === g) hue = (b - r) / delta + 2;
    else hue = (r - g) / delta + 4;
    hue = (hue * 60 + 360) % 360;
  }

  const saturation = max ? (delta / max) * 100 : 0;
  const brightness = max * 100;

  return {
    hue: Math.round(hue),
    saturation: Math.round(saturation),
    brightness: Math.round(brightness)
  };
}

async function getDominantColor() {
  const imgBuffer = await screenshot({ format: 'png' });
  const [color] = await getColors(imgBuffer, 'image/png');
  return color.rgb();
}

async function setupBulbs() {
  const setupTasks = bulbIPs.map(async ip => {
    try {
      const device = await client.getDevice({ host: ip });
      if (!(await device.getPowerState())) await device.setPowerState(true);
      console.log(`✅ Connected to bulb at ${ip}`);
      return device;
    } catch (err) {
      console.error(`❌ Failed to connect to bulb at ${ip}:`, err.message);
      return null;
    }
  });

  const devices = await Promise.all(setupTasks);
  return devices.filter(Boolean);
}

async function syncLoop(devices) {
  console.log('🚀 KL135 Sync Started — Updating every 250ms');
  setInterval(async () => {
    try {
      const rgb = await getDominantColor();
      const hsb = rgbToHSB(rgb);

      await Promise.all(devices.map(device =>
        device.lighting.setLightState({ color_temp: 0, ...hsb })
      ));
    } catch (err) {
      console.error('❌ Sync error:', err.message);
    }
  }, 250);
}

(async () => {
  const devices = await setupBulbs();
  if (devices.length) await syncLoop(devices);
})();
```
3: now click save as and on the same type something like `syncbutdual.js`

4: go in termainl and run this `node syncbutdual.js` (replace syncbutdual.js with the path to the js 

Bounus ig:this was based of https://github.com/jrudio/smart-light-disco/tree/master code probs to them ig
I use KL135 dk if it will work for others kasa lights
