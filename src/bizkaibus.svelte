<script>
  import stops from './gtfs_20221124/stops.csv'
  import trips from './gtfs_20221124/trips.csv'
  import routes from './gtfs_20221124/routes.csv'
  // import stop_times from './gtfs_20221124/stop_times.csv'
  import vehiculos from './gtfs_20221124/vehiculos-gtfs-realtime-2022-12-21T111849.070Z.json'
  import { getContext, onMount } from "svelte";

  import 'leaflet/dist/leaflet.css';
  // import {Map, TileLayer, Marker, Popup} from 'svelte-map-leaflet'
  import {LeafletMap, TileLayer, Marker, Icon, Popup, Tooltip} from 'svelte-leafletjs';

  // import GtfsRealtimeBindings from "gtfs-realtime-bindings";
  // import fetch from "node-fetch";

  import gtfsRealtime from 'gtfs-realtime';

  const config = {
    url: 'http://api.bart.gov/gtfsrt/tripupdate.aspx',
    output: '/path/to/save/file.json',
    header: ['Authorization: bearer 12345'],
  };


  const mapOptions = { center: [43.259925,-2.926869], zoom: 12};
  // const tileUrl = "http://{s}.tile.openstreetmap.fr/hot/{z}/{x}/{y}.png";
  const tileUrl = "https://cartodb-basemaps-{s}.global.ssl.fastly.net/light_all/{z}/{x}/{y}.png";

  const iconOptions = {
    iconUrl: './Bus-logo.svg',
    iconSize: [24, 24],
    iconAnchor: [20, 41],
    popupAnchor: [1, -34],
    tooltipAnchor: [16. -28],
  };

  gtfsRealtime(config)
    .then(() => {
      console.log('Import Successful');
    })
    .catch((err) => {
      console.error(err);
    });

  onMount(async () => {
    // try {
    //   const response = await fetch(
    //     "https://baliabideak.bizkaia.eus/Bizkaibus/GTFSRealTime/FeedVehiculos.pb"
    //   )
    //   if (!res.ok) {
    //     const error = new Error(`${res.url}: ${res.status} ${res.statusText}`);
    //     error.response = res;
    //     throw error;
    //     process.exit(1);
    //   }
    //   const buffer = await response.arrayBuffer();
    //   const feed = GtfsRealtimeBindings.transit_realtime.FeedMessage.decode(
    //     new Uint8Array(buffer)
    //   );
    //   console.log(feed);
    //   feed.entity.forEach((entity) => {
    //     if (entity.tripUpdate) {
    //       console.log("entity:", entity.tripUpdate);
    //     }
    //   });
    // }
    // catch (error) {
    //   console.log(error);
    //   process.exit(1);
    // }
  })

  function getTripDestination(id) {
    const tripInfo = trips.find(el => el.trip_id == id);
    // console.log(tripInfo)
    if (tripInfo !== undefined) return tripInfo.trip_headsign
    else return "Not Found in Database"
  }
  function getStopData(id, item) {
    const stopInfo = stops.find(el => el.stop_id == id);
    if (stopInfo !== undefined && stopInfo[item] !== undefined) {
      return stopInfo[item]
    }
    else {
      console.log(id)
      return "Not Found in Database"
    }
  }
  function getRouteData(id, item) {
    const routeInfo = routes.find(el => el.route_id == id);
    if (routeInfo !== undefined && routeInfo[item] !== undefined) {
      return routeInfo[item]
    }
    else {
      console.log(id)
      return "Not Found in Database"
    }
  }
    
</script>

<div class="map">
  <LeafletMap options={mapOptions}>
    <TileLayer url={tileUrl}></TileLayer>
    {#each vehiculos.entity as v}
    <Marker latLng={[v.vehicle.position.latitude, v.vehicle.position.longitude]}>
      <Icon options={iconOptions}/>
      <Popup>
        <table>
          <tr>
            <td class="item">Línea</td>
            <td>{v.vehicle.trip.tripId.substr(4,5)} / {getRouteData(v.vehicle.trip.tripId.substr(5,4), 'route_long_name')}</td>
          </tr>
          <tr>
            <td class="item">Destino</td>
            <td>{getTripDestination(v.vehicle.trip.tripId)}</td>
          </tr>
          <tr>
            <td class="item">Última parada</td>
            <td>{getStopData(v.vehicle.stopId, 'stop_name')}</td>
          </tr>
        </table>
        <small>ID: {v.vehicle.trip.tripId}</small>
        <a href={getRouteData(v.vehicle.trip.tripId.substr(5,4), 'route_url')} target="_blank" rel="nofollow noopener noreferrer">Info</a>
      </Popup>
    </Marker>
    {/each}
  </LeafletMap>
</div>

<style>
  .map{
    width: 100vw;
    height: 640px;
  }
  .item {
    color: #D20A11;
  }
</style>
