<template>
    <div :style="{'grid-area': area}" class="main"><slot></slot></div>
</template>

<script>
    
export default 
{
  data() { return { } },
  props: ['area']
};

</script>

<style scoped></style>