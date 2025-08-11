<template>
  <div class="app">
    <div class="controls">
      <h1>smilesDrawer 2D</h1>
      <SmilesInput
        v-model:smiles="smiles"
        v-model:size="size"
        v-model:format="format"
        @clear="onClear"
      />
    </div>

    <div class="preview-wrap">
      <SmilesPreview
        :smiles="effectiveSmiles"
        :size="size"
        :format="format"
      />
    </div>

    <AppFooter />
  </div>
</template>

<script>
import SmilesInput from './components/SmilesInput.vue';
import SmilesPreview from './components/SmilesPreview.vue';
import AppFooter from './components/AppFooter.vue';

export default {
  name: 'App',
  components: { SmilesInput, SmilesPreview, AppFooter },
  data() {
    return {
      smiles: '',
      size: 300,
      format: 'png',
      defaultSmiles: 'C(C1=CC=CC=C1)[Ti](CC1=CC=CC=C1)(CC1=CC=CC=C1)CC1=CC=CC=C1'
    };
  },
  computed: {
    effectiveSmiles() {
      return (this.smiles || '').trim() || this.defaultSmiles;
    }
  },
  methods: {
    onClear() {
      this.smiles = '';
    }
  }
};
</script>

<style>
html, body, #app { height: 100%; margin: 0; }
.app {
  min-height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  background: #437568;
}
.controls {
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 10;
  text-align: center;
  background: #213a34;
  padding: 10px 10px;
  border-radius: 8px;
  font-size: 20px;
  color: #ffffff;
}
.controls h1 {
  margin: 0 0 4px 0;
  font-size: 20px;
}
.preview-wrap {
  text-align: center;
  width: 100%;
  padding-top: 100px;
}
</style>


