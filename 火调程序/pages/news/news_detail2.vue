<template>
  <view class="news-detail">
    <!-- 新闻标题 -->
    <view class="title">【科普小贴士】消防安全知识大全</view>

    <!-- 发布时间 -->

    <view class="date">2025年 08月05日 15:58 辽宁</view>





    <image class="cover" src='/static/火灾事件/新闻03/mp.weixin.qq.com_s_BmewtFr0iiFFm3ETV-NluA(iPhone 14 Pro Max).png'
      mode="widthFix" />


    <view class="date">转自：V观阜新公众号</view>
  </view>

</template>

<script setup>

</script>

<style scoped lang="scss">
  .news-detail {
    padding: 20rpx;

    .title {
      font-size: 36rpx;
      font-weight: bold;
      margin-bottom: 10rpx;
    }

    .title02 {
      font-size: 36rpx;
      font-weight: bold;
      margin: 0 auto;
      margin-bottom: 10rpx;
      text-align: center;
      background: #f05454;
      color: #fff;
      border-radius: 10rpx;
      width: 30%;
    }

    .title03 {
      font-size: 32rpx;
      font-weight: bold;
      margin: 10rpx auto;
      margin-bottom: 10rpx;
      text-align: center;
      color: #f05454;
      width: 10%;
    }

    .date {
      margin-top: 40rpx;
      font-size: 20rpx;
      color: #999;
      margin-bottom: 20rpx;
    }

    .cover {
      width: 100%;
      border-radius: 12rpx;
      margin-top: 20rpx;
      margin-bottom: 20rpx;
    }

    .content {
      padding-top: 20rpx;
      margin-bottom: 20rpx;
      font-size: 28rpx;
      line-height: 1.8;
      color: #333;
      // text-align: center;

      p {
        margin-bottom: 20rpx;
      }
    }
  }
</style>