<template>
  <view class="news-detail">
    <!-- 新闻标题 -->
    <view class="title">长约百米货船突发大火，现场浓烟滚滚！</view>

    <!-- 发布时间 -->

    <view class="date">2025年08月11日 15:46 广东</view>





    <image class="cover" src='/static/火灾事件/新闻04/01.png' mode="widthFix" />
    <image class="cover" src='/static/火灾事件/新闻04/02.png' mode="widthFix" />
    <image class="cover" src='/static/火灾事件/新闻04/03.png' mode="widthFix" />
    <image class="cover" src='/static/火灾事件/新闻04/04.png' mode="widthFix" />
    <image class="cover" src='/static/火灾事件/新闻04/05.png' mode="widthFix" />
    <image class="cover" src='/static/火灾事件/新闻04/06.png' mode="widthFix" />


    <view class="date">转自：广东消防</view>
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