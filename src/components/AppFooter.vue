<template>
  <div class="footer">
    © 2025
    <a href="https://github.com/biantailab" target="_blank" rel="noopener noreferrer">BianTai LAB</a>
    <br>
    Powered by 
    <a href="https://github.com/reymond-group/smilesDrawer" target="_blank" rel="noopener noreferrer">smilesDrawer</a>
  </div>
</template>

<script>
export default { name: 'AppFooter' };
</script>

<style scoped>
.footer {
  text-align: center;
  position: fixed;
  bottom: 10px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 16px;
  color: #ffffff;
  opacity: 0.95;
  white-space: nowrap;
}
.footer a {
  color: #e3769b;
  text-decoration: none;
  border-bottom: 0;
  transition: border-bottom-color 300ms ease,
    border-bottom-width 300ms ease,
    border-bottom-style 300ms ease;
}
.footer a:hover {
  border-bottom: 1px dashed #e3769b;
}
</style>
