<script setup>
  import Flow from "@/components/Flow.vue"
  import Rose from "@/components/Rose.vue"
  import FlowMother from "@/components/FlowMother.vue"
  const params = new URLSearchParams(location.search)
  const key = params.get('key')
  let com = key == '1' ?  Rose : Flow

  if(key == "mother"){
    com = FlowMother
  }
</script>

<template>
  <div class="page">
    <component :is="com"></component>
  </div>
</template>

<style scoped>
.page {
  background: radial-gradient(circle at center, #ff9a9e, #fad0c4);
  background: radial-gradient(circle at center, #80CBC4, #004D40);
  background: radial-gradient(farthest-corner, #f7d998, #004D40);
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  outline: none;
}
</style>
