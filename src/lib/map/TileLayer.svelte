<script lang="ts">
    import * as L from 'leaflet';
    import {
      getContext,
      onDestroy,
    } from 'svelte';
  
  
    export let url: string;
    export let options: object = {};
  
    const map = getContext<() => L.Map>('map')();
    let tileLayer = L.tileLayer( url, options).addTo(map);

  
    onDestroy(() => {
        tileLayer.remove();
    });
  
  </script>
  
  <slot />
  