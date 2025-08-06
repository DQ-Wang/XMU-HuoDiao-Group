<template>
  <view class="news-detail">
    <!-- 新闻标题 -->
    <view class="title">新华应急丨5人死亡！广西梧州市一居民楼发生火灾</view>

    <!-- 发布时间 -->

    <view class="date">2025年 08月06日 17:09 北京</view>




    <!-- 正文内容 -->
    <view class="content">
      近日，广西梧州市龙圩区城南大道一居民楼发生火灾，造成5人死亡。据新华网消息，起火建筑为5层砖混结构的居民楼，总建筑面积300平方米，起火部位位于一层。该建筑东面为民房，南面为空地，西面为民房，北面为街道，过火面积约20平方米，燃烧物质为杂物等。目前，善后工作及起火原因调查正有序开展。
    </view>
    <image class="cover" src="/static/火灾事件/新闻02/cover.jpg" mode="widthFix" />
    <view class="content" v-html="news.content02"></view>
    <image class="cover" src="/static/火灾事件/新闻01/wps.png" mode="widthFix" />
    <view class="content" v-html="news.content03"></view>
    <view class="title02"> 相关案例</view>
    <view class="title03"> -1-</view>
    <view class="content" v-html="news.content04"></view>
    <image class="cover" src='/static/火灾事件/新闻01/wps (1).png' mode="widthFix" />
    <view class="title03"> -2-</view>
    <view class="content" v-html="news.content05"></view>
    <image class="cover" src='/static/火灾事件/新闻01/wps (2).png' mode="widthFix" />
    <view class="content" v-html="news.content06"></view>
    <view class="title02"> 安全知识</view>
    <view class="content" v-html="news.content07"></view>

    <view class="box" v-for="(item,index) in photos" :key="index">
      <image class="cover" :src="item" mode='widthFix'></image>
    </view>
    <view class="date">转自：中国消防公众号</view>
  </view>

</template>

<script setup>
  import {
    ref
  } from 'vue'

  // 静态新闻数据（可以后期做成根据 id 传值）
  const news = ref({

    content01: `
    <p>据南京医科大学保卫处通报</p>
    <p>4月14日22时46分</p>
    <p>南京医科大学江宁校区</p>
    <p>北苑01栋1楼</p>
    <p>电表计量控制柜发生故障</p>
    <p>触发烟感报警</p>
   
  `,
    content02: `
    <p>发现火情后</p>
    <p>学校相关部门及时组织学生疏散</p>
    <p>并联合消防部门应急处置</p>
    <p>火情被迅速扑灭</p>
    <p>过火面积约0.5平方米</p>
    <p>未造成任何人员伤亡</p>
    <p>目前，学校正在全力处理后续工作</p>
   
  `,
    content03: `
  <p>据人民网江苏消息</p>
  <p>起火的场所系宿舍楼</p>
  <p>学校火灾不可忽视</p>
  <p>近期已发生多起类似事故</p>
  `,
    content04: `
  <p>据鸡西市消防救援支队通报</p>
  <p>3月17日</p>
  <p>黑龙江工业学院公寓</p>
  <p>一寝室发生火情</p>
  <p>接警后</p>
  <p>辖区消防部门赶赴现场处置</p>
  <p>将明火及时扑灭</p>
  <p>此次火灾未造成人员伤亡</p>
  `,
    content05: `
  <p>据海南消防消息</p>
  <p>3月18日</p>
  <p>海口市龙华区一学校围墙外</p>
  <p>突然起火</p>
  <p>接警后</p>
  <p>多部门到场处置</p>

  `,
    content06: `
  <p>初查
</p>
  <p>施工人员在驾驶小型挖掘机</p>
  <p>清理断头树根时</p>
  <p>挖破地下燃气管道发生明火</p>
  <p>过火面积约3.7平方米</p>
  <p>事故未造成人员被困和受伤</p>
  `,
    content07: `
  <p>学校是师生们生活学习的场所
</p>
  <p>人员密集，安全隐患暗藏</p>
  <p>校园内消防安全不可忽视</p>
  <p>快来一起学习</p>
  <p>这些“必修”知识点</p>

  `,
  })

  const photos = ref([
    '../../static/火灾事件/新闻01/01.png',
    '../../static/火灾事件/新闻01/02.png',
    '../../static/火灾事件/新闻01/03.png',
    '../../static/火灾事件/新闻01/04.png',
    '../../static/火灾事件/新闻01/05.png',
    '../../static/火灾事件/新闻01/06.png',
    '../../static/火灾事件/新闻01/07.png',
    '../../static/火灾事件/新闻01/08.png',

  ])
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
      text-align: center;

      p {
        margin-bottom: 20rpx;
      }
    }
  }
</style>