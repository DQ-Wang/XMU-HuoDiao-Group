<template>
  <view class="select-dialog-container">
    <view class="title animate__animated animate__fadeInDown">选择对话类型</view>

    <view class="options animate__animated animate__fadeInDown">
      <view class="option-card" @click="goTo('graphic')">
        <!-- <image src="@/static/icons/graphic.png" mode="aspectFit" class="icon" /> -->
        <text class="label">图文对话</text>
      </view>

      <view class="option-card" @click="goTo('text')">
        <!-- <image src="@/static/icons/text.png" mode="aspectFit" class="icon" /> -->
        <text class="label">文本对话</text>
      </view>
    </view>
  </view>
</template>

<script setup>
  const goTo = (type) => {
    if (type === 'graphic') {
      uni.navigateTo({
        url: '/pages/talk/talk'
      })
    } else if (type === 'text') {
      uni.navigateTo({
        url: '/pages/talk/textTalk'
      })
    }
  }
</script>

<style lang="scss" scoped>
  @import "../../static/css/animate.min.css";

  .select-dialog-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 100rpx 40rpx;
    background-color: #f7f8fa;
    height: 100vh;

    .title {
      font-size: 36rpx;
      font-weight: 600;
      margin-bottom: 60rpx;
      color: #1e1e1e;
    }

    .options {
      display: flex;
      flex-direction: column;

      gap: 40rpx;
      width: 100%;
    }

    .option-card {

      border-radius: 16rpx; //边框圆角
      margin: 20rpx; //外边距
      padding: 30rpx; //内边距
      box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
      margin-bottom: 5rpx;
      background-color: #f05454;
      display: flex;
      justify-content: center;
      align-items: center;
      transition: transform 0.2s;

      &:active {
        transform: scale(0.97);
      }

      .icon {
        width: 60rpx;
        height: 60rpx;
        margin-right: 30rpx;
      }

      .label {
        font-size: 36rpx;
        color: #fff;
        font-weight: bolder;

      }
    }
  }
</style>