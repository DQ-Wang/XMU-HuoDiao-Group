<template>
  <view class="chat-container">
    <!-- 聊天记录 -->
    <scroll-view class="chat-history" scroll-y="true" :scroll-top="scrollTop">
      <view v-for="(message, index) in messages" :key="index" class="message-row"
        :class="{'is-me': message.role === 'user'}">
        <!-- 头像：你需要将user-avatar.png和ai-avatar.png放到项目的/static目录下 -->
        <image class="avatar" :src="message.role === 'user' ? '/static/user-avatar.jpg' : '/static/ai-avatar.png'">
        </image>
        <view class="message-content">
          <!-- 文本消息 -->
          <view class="text-bubble">
            {{ message.content }}
          </view>
        </view>
      </view>
      <!-- 加载中提示 -->
      <view v-if="isLoading" class="message-row">
        <image class="avatar" src="/static/ai-avatar.png"></image>
        <view class="message-content">
          <view class="text-bubble loading">
            <view class="dot"></view>
            <view class="dot"></view>
            <view class="dot"></view>
          </view>
        </view>
      </view>
    </scroll-view>

    <!-- 底部输入区域 -->
    <view class="input-area">
      <input v-model="inputValue" type="text" placeholder="请输入您的问题..." @confirm="sendMessage" />
      <button @tap="sendMessage" :disabled="isLoading || !inputValue.trim()" class="send-btn">发送</button>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        // ===================================================================
        //  重要：请在这里配置你的API信息
        // ===================================================================
        // 1. 替换为你的 APIPassword (在讯飞控制台获取)
        apiPassword: 'xLhKLFdhzrjlspPoHYGg:TYKSMgJXZmeOlPrUawah',

        // 2. 这是星火X1的HTTP接口地址，通常无需修改
        apiUrl: 'https://spark-api-open.xf-yun.com/v2/chat/completions',
        // ===================================================================

        messages: [
          // 角色 assistant 代表AI, user 代表用户
          {
            role: 'assistant',
            content: 'Hi, 我是火调小助手。有什么可以帮到您？'
          }
        ],
        inputValue: '',
        isLoading: false,
        scrollTop: 0,
      };
    },
    methods: {
      sendMessage() {
        // 检查输入是否为空或正在加载中
        if (this.isLoading || !this.inputValue.trim()) {
          return;
        }

        // 1. 将用户输入添加到消息列表
        const userMessage = {
          role: 'user',
          content: this.inputValue.trim()
        };
        this.messages.push(userMessage);

        // 清空输入框并滚动到底部
        this.inputValue = '';
        this.scrollToBottom();

        // 2. 设置加载状态
        this.isLoading = true;

        // 3. 准备API请求所需的消息历史
        //    根据API文档，需要传递包含上下文的messages数组
        const apiMessages = this.messages.map(msg => {
          return {
            role: msg.role,
            content: msg.content
          };
        });

        // 4. 调用HTTP API
        uni.request({
          url: this.apiUrl,
          method: 'POST',
          header: {
            'Content-Type': 'application/json',
            // 根据文档，鉴权方式为 Bearer Token
            'Authorization': `Bearer ${this.apiPassword}`
          },
          data: {
            model: 'x1',
            messages: apiMessages,
            stream: false, // 使用非流式，简化处理
            // user: 'user-id-123' // 可选：用于区分用户的唯一ID
          },
          success: (res) => {
            console.log('API响应成功:', res.data);
            let replyContent = '';

            // 检查API返回码是否为0 (成功)
            if (res.data.code === 0 && res.data.choices && res.data.choices.length > 0) {
              replyContent = res.data.choices[0].message.content;
            } else {
              // 如果API返回错误，显示错误信息
              replyContent = `请求出错: ${res.data.message || '未知错误'}`;
            }

            // 将AI的回复添加到消息列表
            this.messages.push({
              role: 'assistant',
              content: replyContent
            });
          },
          fail: (err) => {
            // 处理网络请求本身的失败
            console.error('网络请求失败:', err);
            this.messages.push({
              role: 'assistant',
              content: '网络连接失败，请检查网络或稍后再试。'
            });
          },
          complete: () => {
            // 无论成功或失败，都结束加载状态
            this.isLoading = false;
            this.scrollToBottom();
          }
        });
      },

      scrollToBottom() {
        // 使用nextTick确保DOM更新后再执行滚动
        this.$nextTick(() => {
          // 通过设置一个很大的值来确保滚动到底部
          this.scrollTop = this.messages.length * 1000;
        });
      }
    }
  };
</script>

<style lang="scss" scoped>
  .chat-container {
    display: flex;
    flex-direction: column;
    height: 100vh;
    background: linear-gradient(to bottom right, #f7f9fc, #e8ecf3);
  }

  .chat-history {
    flex: 1;
    overflow-y: auto;
    padding: 30rpx 24rpx;
    box-sizing: border-box;
  }

  .message-row {
    display: flex;
    margin-bottom: 40rpx;

    .avatar {
      width: 80rpx;
      height: 80rpx;
      border-radius: 50%;
      flex-shrink: 0;
      box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.08);
    }

    .message-content {
      max-width: calc(100% - 160rpx);
      margin-left: 20rpx;
      display: flex;
      flex-direction: column;
      gap: 8rpx;
    }

    .text-bubble {
      background-color: #ffffff;
      padding: 24rpx;
      border-radius: 20rpx;
      word-break: break-word;
      line-height: 1.6;
      font-size: 28rpx;
      box-shadow: 0 4rpx 10rpx rgba(0, 0, 0, 0.05);
      transition: all 0.2s ease-in-out;
    }

    &.is-me {
      flex-direction: row-reverse;

      .message-content {
        margin-left: 0;
        margin-right: 20rpx;
        align-items: flex-end;
      }

      .text-bubble {
        background-color: #f05454;
        color: #fff;
        border-bottom-right-radius: 4rpx;
      }
    }
  }

  .input-area {
    display: flex;
    align-items: center;
    padding: 24rpx;
    background-color: #ffffff;
    border-top: 1rpx solid #dcdfe6;
    box-shadow: 0 -2rpx 8rpx rgba(0, 0, 0, 0.03);

    input {
      flex: 1;
      height: 70rpx;
      background-color: #f4f5f7;
      border-radius: 35rpx;
      padding: 0 30rpx;
      margin-right: 20rpx;
      font-size: 28rpx;
      color: #333;
      border: none;
      outline: none;
    }

    .send-btn {
      width: 120rpx;
      height: 70rpx;
      line-height: 70rpx;
      font-size: 28rpx;
      background-color: #f05454;
      color: #fff;
      border-radius: 35rpx;
      text-align: center;
      transition: background-color 0.3s ease;

      &[disabled] {
        background-color: #d3bcbc;
      }

      &:active {
        background-color: #f05454;
      }
    }
  }

  // 加载中动画（省略号）
  .loading .dot {
    display: inline-block;
    width: 12rpx;
    height: 12rpx;
    background-color: #aaa;
    border-radius: 50%;
    margin: 0 6rpx;
    animation: dot-blink 1.4s infinite both;
  }

  .loading .dot:nth-child(2) {
    animation-delay: 0.2s;
  }

  .loading .dot:nth-child(3) {
    animation-delay: 0.4s;
  }

  @keyframes dot-blink {

    0%,
    80%,
    100% {
      transform: scale(0);
    }

    40% {
      transform: scale(1.0);
    }
  }
</style>