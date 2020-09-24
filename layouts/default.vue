<template>
  <v-app ref="conversationRef" style="overflow:scroll">
    <nuxt />
  </v-app>
</template>

<script>
export default {
  data() {
    return {}
  },
    updated() {
    console.log(this.$refs.conversationRef)
    this.$refs.conversationRef.scrollTop = this.$refs.conversationRef.scrollHeight
    console.log(
      this.$refs.conversationRef.scrollTop,
      this.$refs.conversationRef.scrollHeight
    )
  },
}
</script>
