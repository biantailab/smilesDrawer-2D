<template>
  <div class="smiles-input">
    <div class="row">
      <input
        :value="smiles"
        @input="$emit('update:smiles', $event.target.value)"
        type="text"
        placeholder="SMILES"
        class="text"
      />
      <button class="btn" type="button" @click="$emit('clear')">Clear</button>
    </div>

    <div class="row">
      <label for="size">size:</label>
      <input
        id="size"
        type="number"
        min="100"
        max="700"
        step="100"
        :value="size"
        @input="$emit('update:size', toNumber($event.target.value))"
        @wheel.prevent="onSizeWheel"
        class="number"
      />

      <label><input type="radio" name="fmt" value="png" :checked="format==='png'" @change="$emit('update:format', 'png')" /> png</label>
      <label><input type="radio" name="fmt" value="jpg" :checked="format==='jpg'" @change="$emit('update:format', 'jpg')" /> jpg</label>
    </div>
    <div class="actions">
      <button class="btn" type="button" :disabled="!canDownload" @click="downloadCurrent">Download</button>
    </div>
  </div>
  
</template>

<script>
export default {
  name: 'SmilesInput',
  props: {
    smiles: { type: String, default: '' },
    size: { type: Number, default: 300 },
    format: { type: String, default: 'png' }
  },
  computed: {
    canDownload() {
      const hasInput = (this.smiles || '').trim().length > 0;
      if (!hasInput) return false;
      void this.size;
      void this.format;
      const canvas = typeof document !== 'undefined' ? document.getElementById('canvas') : null;
      return !!(canvas && canvas.width > 0 && canvas.height > 0);
    }
  },
  methods: {
    toNumber(v) {
      const n = Number(v);
      if (!Number.isFinite(n)) return 300;
      return Math.max(100, Math.min(1000, n));
    },
    onSizeWheel(e) {
      const el = e.currentTarget || e.target;
      const step = Number(el && el.step) || 100;
      const min = Number(el && el.min) || 100;
      const max = Number(el && el.max) || 700;
      const direction = e.deltaY < 0 ? 1 : -1;
      const current = Number(this.size) || 300;
      let next = current + direction * step;
      next = Math.max(min, Math.min(max, next));
      if (next !== current) {
        this.$emit('update:size', next);
      }
    },
    buildBaseName() {
      const raw = (this.smiles || '').trim();
      const w = Math.max(100, Math.min(1000, Number(this.size) || 300));
      const base = raw ? raw : 'molecule';
      return `${base}_${w}`;
    },
    downloadBlob(filename, blob) {
      const a = document.createElement('a');
      const url = URL.createObjectURL(blob);
      a.href = url;
      a.download = filename;
      document.body.appendChild(a);
      a.click();
      setTimeout(() => {
        a.remove();
        URL.revokeObjectURL(url);
      }, 2000);
    },
    downloadDataURL(filename, dataURL) {
      const a = document.createElement('a');
      a.href = dataURL;
      a.download = filename;
      document.body.appendChild(a);
      a.click();
      setTimeout(() => a.remove(), 0);
    },
    downloadCurrent() {
      const canvas = document.getElementById('canvas');
      if (!canvas) return;

      const fmt = this.format;
      const baseName = this.buildBaseName();

      if (fmt === 'png') {
        const dataURL = canvas.toDataURL('image/png');
        this.downloadDataURL(baseName + '.png', dataURL);
        return;
      }

      if (fmt === 'jpg') {
        const off = document.createElement('canvas');
        const srcW = canvas.width;
        const srcH = canvas.height;
        off.width = srcW;
        off.height = srcH;
        const octx = off.getContext('2d');
        octx.imageSmoothingEnabled = true;
        octx.fillStyle = '#ffffff';
        octx.fillRect(0, 0, srcW, srcH);
        octx.drawImage(canvas, 0, 0, srcW, srcH);
        off.toBlob((blob) => {
          if (blob) {
            this.downloadBlob(baseName + '.jpg', blob);
          } else {
            const dataURL = off.toDataURL('image/jpeg', 0.95);
            this.downloadDataURL(baseName + '.jpg', dataURL);
          }
        }, 'image/jpeg', 0.95);
      }
    }
  }
};
</script>

<style scoped>
.row { margin: 4px 0; display: flex; align-items: center; gap: 8px; justify-content: center; }

.text,
.number {
  font-size: 16px;
  padding: 4px 8px 4px 8px;
  background: transparent;
  color: #ffffff;
  border: none;
  border-bottom: 2px solid rgba(255, 255, 255, 0.9);
  outline: none;
  transition: border-color 0.2s ease;
}

.number { width: 50px; color-scheme: dark; -moz-appearance: textfield; }
</style>
