<template>
  <div />
</template>

<script>
import Vue from 'vue';
import propsBinder from '../utils/propsBinder.js';
import findRealParent from '../utils/findRealParent.js';
import { optionsMerger } from '../utils/optionsUtils.js';
import GridLayer from '../mixins/GridLayer.js';
import Options from '../mixins/Options.js';

export default {
  name: 'LGridLayer',
  mixins: [GridLayer, Options],

  props: {
    tileComponent: {
      type: Object,
      custom: true,
      required: true
    }
  },

  data () {
    return {
      tileComponents: {}
    };
  },

  computed: {
    TileConstructor () {
      return Vue.extend(this.tileComponent);
    }
  },

  mounted () {
    const GLayer = L.GridLayer.extend({});
    const options = optionsMerger(this.gridLayerOptions, this);
    this.mapObject = new GLayer(options);
    L.DomEvent.on(this.mapObject, this.$listeners);
    this.mapObject.on('tileunload', this.onUnload, this);
    propsBinder(this, this.mapObject, this.$options.props);
    this.mapObject.createTile = this.createTile;
    this.parentContainer = findRealParent(this.$parent);
    this.parentContainer.addLayer(this, !this.visible);
  },
  beforeDestroy () {
    this.parentContainer.removeLayer(this.mapObject);
    this.mapObject.off('tileunload', this.onUnload);
    this.mapObject = null;
  },

  methods: {
    createTile (coords) {
      let div = L.DomUtil.create('div');
      let dummy = L.DomUtil.create('div');
      div.appendChild(dummy);

      const tileInstance = new this.TileConstructor({
        el: dummy,
        parent: this,
        propsData: {
          coords: coords
        }
      });

      const key = this.mapObject._tileCoordsToKey(coords);
      this.tileComponents[key] = tileInstance;

      return div;
    },

    onUnload (e) {
      const key = this.mapObject._tileCoordsToKey(e.coords);
      if (typeof this.tileComponents[key] !== 'undefined') {
        this.tileComponents[key].$destroy();
        this.tileComponents[key].$el.remove();
        delete this.tileComponents[key];
      }
    },

    setTileComponent (newVal) {
      this.mapObject.redraw();
    }
  }
};
</script>
