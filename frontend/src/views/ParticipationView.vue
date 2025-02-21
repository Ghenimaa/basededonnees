<script setup>
import FormulaireParticipation from "@/components/FormulaireParticipation.vue";
</script>

<template>
  <div class="participation">
    <FormulaireParticipation />
  </div>
</template>

<style scoped>
@media (min-width: 1024px) {
  .participation {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>
