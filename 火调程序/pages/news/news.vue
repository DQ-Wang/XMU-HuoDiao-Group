<template>
  <view class="homelayout">
    <view class="banner ">
      <swiper circular indicator-color="rgba(255,255,255,0.5)" indicator-active-color="#fff" autoplay>
        <swiper-item v-for="item in photos" :key="item">
          <image :src="item" mode="aspectFill"></image>
        </swiper-item>
      </swiper>
    </view>


    <view class="notice">
      <view class="left">
        <uni-icons type="sound-filled" size="20" color="#f05454"></uni-icons>
        <view class="text">公告</view>

      </view>
      <view class="center">
        <swiper vertical autoplay interval="2000" duration="600" circular>
          <swiper-item v-for="(item,index) in warning" :key="index">{{item}}</swiper-item>
        </swiper>
      </view>
      <view class="right">
        <uni-icons type="right" size="16" color="#666"></uni-icons>
      </view>
    </view>

    <view class="select">
      <commin-title title="火灾事件"></commin-title>
      <view class="content">
        <scroll-view scroll-x="true">
          <view class="box" v-for="(item,index) in photos02" :key="index">
            <image :src="item" mode="aspectFill" @click="gotodetail(index)"></image>

          </view>
        </scroll-view>
      </view>
    </view>

  </view>
</template>

<script setup>
  import {
    ref
  } from 'vue';

  const warning = ref([
    '近期高温，请注意用电安全',
    '宿舍消防检查即将开始',
    '请勿使用违规电器',
    '勿在宿舍内私拉乱接电线',
    '保持逃生通道畅通，不要堵塞走廊',
    '请勿遮挡烟雾报警器',
    '禁止使用大功率电器，如电磁炉、电热毯',
    '发现安全隐患请及时上报宿管或辅导员',
    '请勿在宿舍内吸烟',
    '使用插排时勿超负荷接电',
    '切勿使用“三无”电器产品',
    '寝室无人时请关闭所有电源',
    '定期检查电器插头是否松动、发热',
    '防火防盗，注意宿舍财产安全',
    '宿舍内禁止使用明火设备',
    '保持宿舍整洁，避免可燃物堆积',
    '勿随意拆改宿舍电路设施',
    '如遇火情，切勿慌张，立即疏散并报警',
    '参与消防演练，掌握基本逃生技能',
    '注意人身安全，晚归请结伴而行'
  ])

  const photos = ref([
    // '../../static/火调实地调查照片/采访03.jpg',
    // '../../static/火调实地调查照片/采访04.jpg',
    // '../../static/火调实地调查照片/观察器材02.jpg',
    // '../../static/火调实地调查照片/观察器材03.jpg',
    // '../../static/火调实地调查照片/合照02.jpg',
    // '../../static/火调实地调查照片/合照06.jpg',
    '../../static/应知应会PPT图片/01.png',
    '../../static/应知应会PPT图片/02.png',
    '../../static/应知应会PPT图片/03.png',
    '../../static/应知应会PPT图片/04.png',
    '../../static/应知应会PPT图片/05.png',
    '../../static/应知应会PPT图片/06.png',
    '../../static/应知应会PPT图片/07.png',
    '../../static/应知应会PPT图片/08.png'
  ])


  const photos02 = ref(['../../static/火灾事件/新闻01/cover.png', "/static/火灾事件/新闻02/cover.jpg"


  ])

  const gotodetail = (index) => {
    uni.navigateTo({
      url: `/pages/news/news_detail${index}`
    })
  }
</script>

<style lang="scss" scoped>
  @import url('../../static/css/wholeCss.css');
  @import '../../static/css/animate.min.css';

  .homelayout {
    .banner {
      width: 750rpx;
      padding: 30rpx 0;

      swiper {
        width: 750rpx;
        height: 375rpx; //量好高度图片才能还原大小

        &-item {
          width: 100%;
          height: 100%;
          padding: 0 30rpx;
          box-sizing: border-box;


          image {
            height: 100%;
            width: 100%;
            border-radius: 10rpx;

          }
        }
      }
    }

    .notice {
      display: flex;
      width: 690rpx;
      height: 80rpx;
      line-height: 80rpx; //疑问：为什么在父级元素中指定行高就能让子块的文字处于同一行，而且行高和高度参数一致
      background: #f9f9f9;
      margin: 0 auto;
      border-radius: 80rpx; //tips:参数上限是高度和宽度的较小值，当与较小值相同时，呈胶囊状！！！


      // border: 1px solid red;
      .left {
        display: flex;
        width: 140rpx;
        // border: 1px solid red;
        justify-content: center;
        align-items: center;

        .text {
          color: #d81e06;
          font-size: 28rpx;
          font-weight: 600;
        }

      }

      .center {
        flex: 1;

        // border: 1px solid red;
        swiper {
          height: 100%; //perfect！由于swiper有默认的高度，让他继承父级高度可以增加视觉美感

          &-item {
            height: 100%;
            font-size: 30rpx;
            color: #666;
            overflow: hidden;
            white-space: nowrap;
            text-overflow: ellipsis; //文字太多时的解决办法，加引号
          }
        }



      }

      .right {
        width: 20rpx;
        display: flex;
        justify-content: center;
        align-items: center;

      }
    }

    .select {
      padding-top: 50rpx;

      .content {
        width: 720rpx;
        margin-top: 30rpx;
        margin-left: 30rpx;

        scroll-view {
          white-space: nowrap;

          .box {
            width: 200rpx;
            height: 430rpx;
            display: inline-block;
            margin-right: 15rpx;

            image {
              width: 100%;
              height: 100%;
              border-radius: 10rpx;
            }


          }

          .box:last-child {
            margin-right: 30rpx; //伪类要与其本来的类同级，这里单独修改box类最后一个元素，使其左右margin对称
          }
        }
      }
    }
  }
</style>