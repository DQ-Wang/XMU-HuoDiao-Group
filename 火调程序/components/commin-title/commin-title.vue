<template>
  <view class="title">
    {{props.title}}
  </view>

</template>

<script setup>
  const props = defineProps({
    title: {
      type: String,
      required: true
    }
  })
</script>

<style lang="scss" scoped>
  .title {
    padding-left: 30rpx;
    font-weight: bold;
  }
</style>