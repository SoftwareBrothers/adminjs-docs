---
description: '@adminjs/leaflet'
---

# Leaflet Maps

The Leaflet feature (`@adminjs/leaflet`) integrates Leaflet ([https://leafletjs.com/](https://leafletjs.com/)) into AdminJS allowing you to visualize your maps and markers.

## Installation

```bash
$ yarn add @adminjs/leaflet # or: npm install @adminjs/leaflet
```

## Usage

`@adminjs/leaflet` package exports two features that you should use based on your use case.

* `leafletSingleMarkerMapFeature` should be used when your resource is the `Marker` itself, and you only want to display a single marker on the rendered map. This should be used when, for example, you'd like to change a specific marker's location without displaying all your markers on a map.

<figure><img src="../../.gitbook/assets/Screenshot 2023-01-26 at 12.15.30.png" alt=""><figcaption><p>leafletSingleMarkerMapFeature</p></figcaption></figure>

* `leafletMultipleMarkersMapFeature` should be used when your resource is a collection of `Marker`s. An example could be a `Map` resource with `Marker` being a separate resource. This feature will allow you to manage all associated markers.

{% hint style="warning" %}
`leafletMultipleMarkersMapFeature` will not display the map when creating a new `Map` record due to dynamic management of markers. Whenever you drag, edit, create or delete a `Marker`, the change takes effect immediately which relies on your `Map` to had been created before.  If you want to manage your `Map` markers, create the `Map` first and you can manage them via `edit` action.
{% endhint %}

<figure><img src="../../.gitbook/assets/Screenshot 2023-01-26 at 12.14.54.png" alt=""><figcaption><p>leafletMultipleMarkersMapFeature</p></figcaption></figure>

The usage tutorial will be split into two sections related to the feature you're using but there are some common steps you must take to get the features working.

First of all, you have to import `getLeafletDist` from `@adminjs/leaflet`. Leaflet library has it's own CSS which is required for the maps to display properly but AdminJS core does not let you import CSS files directly into React components (as of version 6). As a workaround, `@adminjs/leaflet` exports `getLeafletDist` utility function which resolves a path to Leaflet dist files in your `node_modules`. These dist files have to be exposed by your server framework. The example below shows how you can do it with `express`.

```typescript
import leafletFeatures, { getLeafletDist } from '@adminjs/leaflet';
import AdminJS, { ComponentLoader } from 'adminjs';
import express from 'express';

// ...

const app = express();
// Use express.static to serve public files
app.use(express.static(getLeafletDist()));

// ...

const componentLoader = new ComponentLoader();
const admin = new AdminJS({
  componentLoader,
  assets: {
    // Tell AdminJS that leaflet.css is available under <server url>/leaflet.css
    styles: ['/leaflet.css'],
  },
});
```

Alternatively, you can provide a CDN link to `leaflet.css` in `styles`.

In the example above, we also instantiated a `ComponentLoader`. Take note of that, as it will be used by Leaflet features later.

### leafletSingleMarkerMapFeature

The `AdminJS` configuration described above should be extended with your `Marker` resource.

The feature goes inside `features` in your resource specification. Take a look at the example below which includes comments on how you can configure the feature.

```typescript
import leafletFeatures, { getLeafletDist } from '@adminjs/leaflet';
// other imports
import Marker from './marker.entity.js';

// Other code - remember to set up the common configuration!

const admin = new AdminJS({
  resources: [
    {
      resource: Marker,
      features: [
        leafletFeatures.leafletSingleMarkerMapFeature({
          /* You must provide your "componentLoader" instance for the feature
          to add it's components. */
          componentLoader,
          /* You must provide "paths" for the feature to know which fields
          contain your marker's coordinates or where to display the map in the UI. */
          paths: {
            /* A property which should be used to display the map. It can be an entirely
            new property name, or you can use an actual field from your Marker's model. */
            mapProperty: 'location',
            /* "jsonProperty" is optional and should only be given if your Marker's latitude
            and longitude are stored in a single JSON field. In this example, "location"
            is of GeoJSON.Point type (https://geojson.org/). If your latitude and longitude
            are stored in separate fields, leave this option undefined. */
            jsonProperty: 'location',
            /* If your latitude has a separate field in your model, just use the field name.
            Example:
              latitudeProperty: 'latitude'
            If your latitude property is a part of a JSON structure (GeoJSON example from above)
            you must provide a flattened path under which the latitude should be saved in the JSON. */
            latitudeProperty: 'location.coordinates.0',
            /* Longitude configuration is the same as latitude's */
            longitudeProperty: 'location.coordinates.1',
            /* 'location.coordinates.0' combined with 'location.coordinates.1' will save the
            coordinates in the following format: { coordinates: [<lat>, <lng>] } */
          },
          /* "baseValue" should be left undefined unless your coordinates field is of JSON structure
          that needs additional constant attributes. The GeoJSON example described above requires
          the Point payload to be: { type: 'Point', coordinates: [<lat>, <lng>] }
          Providing only "paths.latitudeProperty" and "paths.longitudeProperty" will send the coordinates
          as: { coordinates: [<lat>, <lng>] } with `type: 'Point'` missing. "baseValue" can be used
          so that `type: 'Point'` is always added to the payload.
          Please note that "baseValue" is only required for JSON structures which require extra elements.
          If your latitude and longitude are stored in separate fields, please leave "baseValue" undefined. */
          baseValue: { type: 'Point', coordinates: [] },
          /* "mapProps" are passed to React Leaflet's MapContainer component. You can use them to
          change initial zoom, initial coordinates, max zoom, map bounds, disable scroll zoom, etc.
          Reference: https://react-leaflet.js.org/docs/v3/api-map/ */
          mapProps: undefined,
          /* "tileProps" are passed to React Leaflet's TileLayer component. You can use them
          to provide your custom tile URL, attribution, etc.
          Reference: https://react-leaflet.js.org/docs/v3/api-components/#tilelayer */
          tileProps: undefined,
        }),
      ],
    },
  ],
  componentLoader,
  assets: {
    styles: ['/leaflet.css'],
  },
});
```

### leafletMultipleMarkersMapFeature

The `AdminJS` configuration described above should be extended with your `MapEntity` resource.

{% hint style="info" %}
We use `MapEntity` instead of simply `Map` for entity name because `Map` is a reserved name in Javascript.
{% endhint %}

The feature goes inside `features` in your resource specification. Take a look at the example below which includes comments on how you can configure the feature.

```typescript
import leafletFeatures, { getLeafletDist } from '@adminjs/leaflet';
// other imports
import MapEntity from './map.entity.js';

// Other code - remember to set up the common configuration!

const admin = new AdminJS({
  resources: [
    {
      resource: MapEntity,
      features: [
        leafletFeatures.leafletMultipleMarkersMapFeature({
          /* You must provide your "componentLoader" instance for the feature
          to add it's components. */
          componentLoader,
          /* Since Map and Marker are in 1:M relation, a property to display the map
          will not be present by default and the feature has to create one.
          In this example, "mapProperty" creates a new "markers" field in your Map's "edit"
          and "show" views. */
          mapProperty: 'markers',
          /* "markerOptions" are required for the feature to know where to get and how to manage
          your markers. Please note that "edit", "new" and "list" actions have to be enabled in your
          Marker resource. */
          markerOptions: {
            /* Your marker's resource ID. This is usually either the model name or a table name
            of your Marker unless you'd changed it. */
            resourceId: 'Marker',
            /* The foreign key in your Marker model which associates it with currently managed Map */
            foreignKey: 'mapId',
            /* This configuration is exactly the same as "paths" configuration
            in "leafletSingleMarkerMapFeature" */
            paths: {
              /* A property which should be used to display the map. It can be an entirely
              new property name, or you can use an actual field from your Marker's model. */
              mapProperty: 'location',
              /* "jsonProperty" is optional and should only be given if your Marker's latitude
              and longitude are stored in a single JSON field. In this example, "location"
              is of GeoJSON.Point type (https://geojson.org/). If your latitude and longitude
              are stored in separate fields, leave this option undefined. */
              jsonProperty: 'location',
              /* If your latitude has a separate field in your model, just use the field name.
              Example:
                latitudeProperty: 'latitude'
              If your latitude property is a part of a JSON structure (GeoJSON example from above)
              you must provide a flattened path under which the latitude should be saved in the JSON. */
              latitudeProperty: 'location.coordinates.0',
              /* Longitude configuration is the same as latitude's */
              longitudeProperty: 'location.coordinates.1',
              /* 'location.coordinates.0' combined with 'location.coordinates.1' will save the
              coordinates in the following format: { coordinates: [<lat>, <lng>] } */
            },
          },
          /* "mapProps" are passed to React Leaflet's MapContainer component. You can use them to
          change initial zoom, initial coordinates, max zoom, map bounds, disable scroll zoom, etc.
          Reference: https://react-leaflet.js.org/docs/v3/api-map/ */
          mapProps: {
            /* In "leafletSingleMarkerMapFeature" the map is centered at your marker by default.
            In "leafletMultipleMarkersMapFeature" the markers are fetched asynchronously and the map
            is rendered before they're available so you must provide initial coordinates yourself.
            By default, if you don't provide "center", the map will be centered at London coordinates.
            
            Future versions of "@adminjs/leaflet" should have better handling of initial coordinates. */
            center: [52.237049, 21.017532],
          },
          /* "tileProps" are passed to React Leaflet's TileLayer component. You can use them
          to provide your custom tile URL, attribution, etc.
          Reference: https://react-leaflet.js.org/docs/v3/api-components/#tilelayer */
          tileProps: undefined,
        }),
      ],
    },
  ],
  componentLoader,
  assets: {
    styles: ['/leaflet.css'],
  },
});
```

{% hint style="info" %}
`leafletMultipleMarkersMapFeature` works best when combined with `leafletSingleMarkerMapFeature` in your `Marker` resource.
{% endhint %}

## Example

An example application can be found in [@adminjs/leaflet GitHub repository](https://github.com/SoftwareBrothers/adminjs-leaflet/tree/main/example-app).

### Installation

```bash
$ git clone https://github.com/SoftwareBrothers/adminjs-leaflet.git
$ cd adminjs-leaflet
$ yarn install
$ yarn build
$ cd example-app
$ yarn install
$ docker-compose up -d
$ yarn start
```

## Issues & Ideas

`@adminjs/leaflet` is still in its early development phase. If you have any ideas on how to improve it, add new features or if you find any bugs, please create an issue in its [GitHub repository](https://github.com/SoftwareBrothers/adminjs-leaflet/issues).
