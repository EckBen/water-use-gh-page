<script lang="ts">
    import * as L from 'leaflet';
    import flush from 'just-flush';
    import {
      getContext,
      setContext,
      onDestroy,
      createEventDispatcher,
    } from 'svelte';
  
    const dispatch = createEventDispatcher();
  
    export let geojson: any;
    export let color: string;
    export let pane: string | undefined = undefined;
    export let fillColor: string | undefined = undefined;
    export let fillOpacity: number | undefined = undefined;
    export let weight: number | undefined = undefined;
  
    const geojsonMarkerOptions = {
    radius: 4,
    fillColor: 'blue',
    color:'blue',
    weight: 1,
    opacity: 0,
    fillOpacity: 0.8
};

    const container = getContext<() => L.LayerGroup>('layerGroup')();
    let layerPane = pane || getContext<string>('pane');
    console.log("GeoJson:",geojson);
    export let layer: L.GeoJSON = L.geoJSON(geojson)
      .on('mouseover', (e) => dispatch('mouseover', e))
      .on('mouseout', (e) => dispatch('mouseout', e))
      .on('click', (e) => dispatch('click', e))
      .addTo(container);
  
    setContext('layer', () => layer);
  
    onDestroy(() => {
      layer.remove();
    });
  
    $: layerStyle = flush({ radius: 4, color, fillColor, fillOpacity, weight });
    $: layer.setStyle(layerStyle);
  </script>
  
  <slot />
  