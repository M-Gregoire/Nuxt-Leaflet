# Nuxt-Leaflet

A Nuxt module already exists for using Leaflet with Nuxt ([nuxt-leaflet](https://github.com/schlunsen/nuxt-leaflet)) ~~but it doesn't expose `L` from Leaflet, which makes the use of plugins really difficult~~, which now exports `$L` (See https://github.com/M-Gregoire/Nuxt-Leaflet/issues/2).


Moveover, Leaflet cannot be used (cleanly) with Nuxt by default because of ssr. The best way I found was creating a Nuxt plugin (Thanks [Christian Nwamba](https://medium.com/@codebeast_/why-your-third-party-plugin-dont-work-in-nuxt-and-how-to-fix-it-d1a8caadf422)!)

## Steps

### Vue2Leaflet
Install [Vue2Leaflet](https://github.com/KoRiGaN/Vue2Leaflet) :
`$ npm install vue2-leaflet leaflet --save`

### Nuxt plugin

Create a Nuxt Plugin in `src/plugins` :

``` javascript
// src/plugins/vue-leaflet.js
import Vue from 'vue'
import { LMap, LTileLayer, LMarker } from 'vue2-leaflet'
const VueLeaflet = {
  install(Vue, options) {
    Vue.component('l-map', LMap)
    Vue.component('l-marker', LMarker)
    Vue.component('l-tile-layer', LTileLayer)
  }
};
Vue.use(VueLeaflet);
export default VueLeaflet;
```

Of course, you can add any other component from Leaflet that you need using `Vue.component`, such as `L-Rectangle`.  

Add the plugin to `nuxt.config.js` and disable ssr :

``` javascript
// nuxt.config.js
  plugins: [
    {src: '~/plugins/vue-leaflet', ssr: false },
  ],

```

### CSS

Finally, because we are using a Nuxt plugin, the CSS won't work properly. You can either :

* Add it in each page you are using Leaflet :
`<link rel="stylesheet" href="https://unpkg.com/leaflet@1.2.0/dist/leaflet.css">`

* Load it globally using a CDN :

``` javascript
// nuxt.config.js
  head: {
    link: [
      { rel: 'stylesheet', href: 'https://unpkg.com/leaflet@1.2.0/dist/leaflet.css' },
    ],
  },
```

* Load it locally, you will also need to serve the image locally also :

``` javascript

// nuxt.config.js
  css: [
    "~assets/css/leaflet/leaflet.css",
  ],
```

* Load it in the Leaflet plugin (Thank you [xpuu](https://github.com/M-Gregoire/Nuxt-Leaflet/issues/1)):

``` javascript
import 'leaflet/dist/leaflet.css'
```

### Use it !

You can now access `L` in your vue template !  
_Example :_ (Don't forget to replace `your.mapbox.access.token`) !

``` vue
<template>
	<div id="map-wrap"></div>
</template>
<script>
	export default {
		name: 'leaflet-map',
		mounted() {
			var mymap = L.map('map-wrap').setView([51.505, -0.09], 13)
			L.tileLayer('https://api.tiles.mapbox.com/v4/{id}/{z}/{x}/{y}.png?access_token={accessToken}', {
    attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
    maxZoom: 18,
    id: 'mapbox.streets',
    accessToken: 'your.mapbox.access.token'
}).addTo(mymap);
		}
	}
</script>
```

### Add methods to plugin

I did this in order to being able to use [PruneCluster](https://github.com/SINTEF-9012/PruneCluster) with Nuxt.  

Add your import in the plugin and set a property for the methods you wan't to use :  
_Example :_
``` javascript
import Vue from 'vue'

import Vue2Leaflet from 'vue2-leaflet'

import { PruneCluster, PruneClusterForLeaflet } from 'exports-loader?PruneCluster,PruneClusterForLeaflet!prunecluster/dist/PruneCluster.js'

const VueLeaflet = {

  install(Vue, options) {
    Vue.component('l-map', Vue2Leaflet.LMap)
    Vue.component('l-marker', Vue2Leaflet.LMarker)
    Vue.component('l-tile-layer', Vue2Leaflet.LTileLayer)
    Vue.prototype.PruneCluster = PruneCluster;
    Vue.prototype.PruneClusterForLeaflet = PruneClusterForLeaflet;
  }
};


Vue.use(VueLeaflet);

export default VueLeaflet;
```

In your vue template, you can call your methods by prepending them with `this.`
_Example :_
``` vue
<template>
	<div id="map-wrap"></div>
</template>
<script>
	export default {
		name: 'leaflet-map',
		mounted() {
			var map = L.map("map-wrap", {
				attributionControl: false,
				zoomControl: false
			}).setView(L.latLng(-37.79, 175.27), 12);
			L.tileLayer('http://{s}.tile.osm.org/{z}/{x}/{y}.png', {
				detectRetina: true,
				maxNativeZoom: 17
			}).addTo(map);

			var pruneCluster = new this.PruneClusterForLeaflet();
			var marker = new this.PruneCluster.Marker(59.8717, 11.1909);
			pruneCluster.RegisterMarker(marker);
			map.addLayer(pruneCluster);
		},
	}
</script>
```

## Donation

This project helped you ? You can buy me a cup of coffee  
[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=EWHGT3M9899J6)
