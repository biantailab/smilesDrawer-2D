<template>
  <div class="preview">
    <div class="error" v-if="error">{{ error }}</div>
    <canvas id="canvas" ref="canvas" :width="size" :height="size"></canvas>
  </div>
</template>

<script>

export default {
  name: 'SmilesPreview',
  props: {
    smiles: { type: String, required: true },
    size: { type: Number, default: 300 },
    format: { type: String, default: 'png' }
  },
  data() {
    return {
      canDownload: false,
      error: '',
      parsedTree: null,
      redrawTimerId: null
    };
  },
  watch: {
    smiles: 'parseSmiles',
    size: 'scheduleRedraw',
    format: 'scheduleRedraw'
  },
  mounted() {
    this.parseSmiles();
  },
  methods: {
    scheduleRedraw() {
      if (this.redrawTimerId) clearTimeout(this.redrawTimerId);
      this.redrawTimerId = setTimeout(() => {
        this.redrawTimerId = null;
        this.$nextTick(() => this.redraw());
      }, 120);
    },
    clearCanvasBackground(ctx, w, h) {
      ctx.clearRect(0, 0, w, h);
      if (this.format === 'jpg') {
        ctx.fillStyle = '#ffffff';
        ctx.fillRect(0, 0, w, h);
      }
    },
    setCanvasSize(w, h) {
      const canvas = this.$refs.canvas;
      if (!canvas) return;
      canvas.width = w;
      canvas.height = h;
      canvas.style.width = w + 'px';
      canvas.style.height = h + 'px';
    },
    parseSmiles() {
      const canvas = this.$refs.canvas;
      if (!canvas) return;

      this.error = '';
      this.canDownload = false;
      this.parsedTree = null;
      const w = Math.max(100, Math.min(1000, Number(this.size) || 300));
      const h = w;
      this.setCanvasSize(w, h);

      const ctx = canvas.getContext('2d');
      this.clearCanvasBackground(ctx, w, h);

      window.SmilesDrawer.parse(
        (this.smiles || '').trim(),
        (tree) => {
          this.parsedTree = tree;
          this.$nextTick(() => this.redraw());
        },
        () => {
          this.canDownload = false;
          this.error = 'Failed to get molecular structure';
        }
      );
    },
    redraw() {
      const canvas = this.$refs.canvas;
      if (!canvas) return;

      const w = Math.max(100, Math.min(1000, Number(this.size) || 300));
      const h = w;
      this.setCanvasSize(w, h);

      const opts = {
        width: w,
        height: h,
        bondLength: 18,
        bondThickness: 0.7,
        padding: 24
      };

      const ctx = canvas.getContext('2d');
      this.clearCanvasBackground(ctx, w, h);

      if (!this.parsedTree) {
        this.canDownload = false;
        return;
      }

      const drawer = new window.SmilesDrawer.Drawer(opts);
      drawer.draw(this.parsedTree, 'canvas', 'light', false);
      this.canDownload = true;
    }
  }
};
</script>

<style scoped>
.preview {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
}
canvas { display: block; margin: 0 auto; background: transparent; }
.error { color: red; text-align: center; }
</style>
