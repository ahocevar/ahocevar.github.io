---
layout: post
title:  "Proj4js 2.2.x with OpenLayers 3"
date:   2014-07-10 14:10:42
categories: 
---

With pull request [openlayers/ol3#1228](https://github.com/openlayers/ol3/pull/1228), the API for registering coordinate transform functions has been simplified. Along with this change, support for [Proj4js](http://proj4js.org/) 2.2.x has been added to [OpenLayers 3](http://ol3js.org/) master. At the same time, transparent Proj4js 1.x integration is no longer available.

For users who previously used Proj4js 1.x and projection definitions from [http://spatialreference.org/](http://spatialreference.org/), the changes are fairly straightforward. Let's say you had Proj4js and a projection definition loaded in your html:

```html
<script
    src="http://cdnjs.cloudflare.com/ajax/libs/proj4js/1.1.0/proj4js-compressed.js">
</script>
<script src="http://spatialreference.org/ref/epsg/21781/proj4js/"></script>
```

To update your html, these two script tags would slightly change:

```html
<script src="http://cdnjs.cloudflare.com/ajax/libs/proj4js/2.2.1/proj4.js"></script>
<script src="http://epsg.io/21781.js"></script>
```

Note that [http://epsg.io/](http://epsg.io/) provides projection definitions for Proj4js 2.x, while [http://spatialreference.org/](http://spatialreference.org/) has definitions for Proj4js 1.x.

If you were previously relying on Proj4js to transform coordinates from WFS servers, you may need to set up SRS code aliases manually, because Proj4js 2.x does not do that any more:

```js
proj4.defs('urn:x-ogc:def:crs:EPSG:21781', proj4.defs('EPSG:21781'));
```

The above snippet makes the 'EPSG:21781' projection and transforms also available when the 'urn:x-ogc:def:crs:EPSG:21781' SRS identifier is used.

You can see that both the namespace and the way you provide projection definitions have changed in Proj4js 2.x. The old syntax was:

```js
Proj4js.defs['EPSG:4326'] =
    '+title=WGS 84 (long/lat) +proj=longlat +ellps=WGS84 +datum=WGS84 +units=degrees';
```

The new one is:

```js
Proj4js.defs(
  'EPSG:4326',
  '+title=WGS 84 (long/lat) +proj=longlat +ellps=WGS84 +datum=WGS84 +units=degrees'
);
```

If your application previously used `ol.proj.configureProj4jsProjection`, you also have to change your code a bit. Let's assume you had a code snippet like this:

```js
var projection = ol.proj.configureProj4jsProjection({
  code: 'EPSG:21781',
  extent: [485869.5728, 76443.1884, 837076.5648, 299941.7864]
});
```

When updating, you have to replace it with the following:

```js
var projection = ol.proj.get('EPSG:21781');
projection.setExtent([485869.5728, 76443.1884, 837076.5648, 299941.7864]);
```

For those who want to use custom tranform functions, OpenLayes 3 has a new [ol.proj.addCoordinateTransforms()](http://ol3js.org/en/master/apidoc/ol.proj.html#addCoordinateTransforms) function. The usage is shown in the [wms-custom-proj](https://github.com/openlayers/ol3/blob/v3.0.0-gamma.2/examples/wms-custom-proj.js#L30-L42) example.

