<template>
  <view class="chat-container">
    <!-- 聊天记录 -->
    <scroll-view class="chat-history" scroll-y="true" :scroll-top="scrollTop">
      <view v-for="(message, index) in messages" :key="index" class="message-row"
        :class="{'is-me': message.role === 'user'}">
        <image class="avatar" :src="message.role === 'user' ? '/static/user-avatar.jpg' : '/static/ai-avatar.png'">
        </image>
        <view class="message-content">
          <!-- 文本消息 -->
          <view v-if="message.type === 'text'" class="text-bubble">
            {{ message.content }}
          </view>
          <!-- 图片消息 -->
          <view v-if="message.type === 'image'" class="image-bubble">
            <image :src="message.content" mode="widthFix" @tap="previewImage(message.content)"></image>
          </view>
        </view>
      </view>
      <!-- 加载中提示 -->
      <view v-if="isLoading" class="message-row is-ai">
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
      <input v-model="inputValue" type="text" placeholder="有什么问题尽管问我" @confirm="sendMessage" />
      <view class="plus-icon" @tap="chooseImage">+</view>
      <button @tap="sendMessage" :disabled="isLoading || (!inputValue && !tempImage)" class="send-btn">发送</button>
    </view>
  </view>
</template>

<script>
  // 引入crypto-js库用于HMAC-SHA256加密
  import CryptoJS from 'crypto-js';

  export default {
    data() {
      return {
        // 在这里填入你的讯飞API密钥信息
        APPID: '51a9a205',
        APIKey: 'a5d922ed7e47402b479c555b206dc767',
        APISecret: 'NzI5YWUwMzM0OTllMWNjMzcxOGJmN2Uz',

        messages: [{
          role: 'ai',
          type: 'text',
          content: 'Hi, 我是火调小助手。有什么可以帮到您？'
        }],
        inputValue: '',
        tempImage: null,
        isLoading: false,
        scrollTop: 0,
        ws: null,
      };
    },
    methods: {
      // ===================================================================
      //  核心修正：重写 chooseImage 方法，增加健壮性
      // ===================================================================
      chooseImage() {
        uni.chooseImage({
          count: 1,
          sourceType: ['album', 'camera'],
          success: (res) => {
            const tempFilePath = res.tempFilePaths[0];

            // --- 优化点 1: 立即显示图片 ---
            // 无论后续Base64转换是否成功，都先把用户选择的图片展示出来
            this.messages.push({
              role: 'user',
              type: 'image',
              content: tempFilePath
            });
            this.scrollToBottom();

            // --- 优化点 2: 异步处理Base64并添加try...catch ---
            this.processImage(tempFilePath);
          },
          fail: (err) => {
            console.error('选择图片失败:', err);
            uni.showToast({
              title: '选择图片失败',
              icon: 'none'
            });
          }
        });
      },

      async processImage(filePath) {
        try {
          uni.showLoading({
            title: '图片处理中...'
          });

          // 尝试将图片文件转为Base64
          const base64 = await this.imageToBase64(filePath);

          // 存储转换后的结果
          this.tempImage = {
            path: filePath,
            base64: base64
          };

          // --- 优化点 3: 成功后再填充默认问题 ---
          // 只有在图片处理成功后，才填充输入框，引导用户提问
          if (!this.inputValue) {
            this.inputValue = "请描述一下这张图片的内容";
          }

          uni.hideLoading();

        } catch (error) {
          // 如果转换失败，给出明确提示
          console.error('图片转Base64失败:', error);
          uni.hideLoading();
          uni.showToast({
            title: '图片处理失败，请重试',
            icon: 'none'
          });
          // 清理掉临时的图片数据，防止用户直接发送
          this.tempImage = null;
          // 也可以在这里向聊天记录中追加一条错误消息
          this.messages.push({
            role: 'ai',
            type: 'text',
            content: `图片处理失败: ${error.errMsg || '未知错误'}`
          });
          this.scrollToBottom();
        }
      },

      // 图片转Base64 (保持不变，但其调用方已增加错误处理)
      // 图片转Base64 (兼容H5、小程序和App的多端实现)
      imageToBase64(filePath) {
        return new Promise(async (resolve, reject) => {
          // #ifdef H5
          // H5环境的实现：使用FileReader
          try {
            // uni.chooseImage在H5端返回的是一个blob URL，我们需要先fetch它
            const response = await fetch(filePath);
            const blob = await response.blob();
            const reader = new FileReader();
            reader.onload = (e) => {
              // e.target.result 的格式是 "data:image/jpeg;base64,xxxxxx..."
              // 我们需要去掉前缀，只保留Base64部分
              const base64 = e.target.result.split(',')[1];
              resolve(base64);
            };
            reader.onerror = (e) => {
              console.error("FileReader error:", e);
              reject(new Error('图片读取失败'));
            };
            reader.readAsDataURL(blob);
          } catch (error) {
            console.error("Fetch blob error:", error);
            reject(error);
          }
          // #endif

          // #ifndef H5
          // 非H5环境（小程序、App等）的实现：使用getFileSystemManager
          uni.getFileSystemManager().readFile({
            filePath: filePath,
            encoding: 'base64',
            success: (res) => {
              resolve(res.data);
            },
            fail: (err) => {
              console.error("getFileSystemManager error:", err);
              reject(err);
            }
          });
          // #endif
        });
      },

      // ===================================================================
      //  以下为其他方法，基本保持不变
      // ===================================================================

      sendMessage() {
        if (this.isLoading || (!this.inputValue && !this.tempImage)) {
          // 如果没有文字也没有图片，或者正在加载，则不发送
          if (!this.inputValue && !this.tempImage) {
            uni.showToast({
              title: '请输入问题或上传图片',
              icon: 'none'
            });
          }
          return;
        }

        if (this.inputValue) {
          this.messages.push({
            role: 'user',
            type: 'text',
            content: this.inputValue
          });
        }

        this.isLoading = true;
        this.scrollToBottom();

        const params = {
          image: this.tempImage ? this.tempImage.base64 : null,
          question: this.inputValue || "这张图片是什么？"
        };

        this.inputValue = '';
        this.tempImage = null;

        this.callSparkAPI(params);
      },

      async callSparkAPI({
        image,
        question
      }) {
        const url = await this.getAuthorizationUrl();

        const aiResponse = {
          role: 'ai',
          type: 'text',
          content: ''
        };
        this.messages.push(aiResponse);
        this.scrollToBottom();

        this.ws = uni.connectSocket({
          url: url,
          success: () => {},
          fail: (err) => {
            console.error('WebSocket连接失败', err);
            this.isLoading = false;
            aiResponse.content = '连接服务器失败，请稍后再试。';
          }
        });

        this.ws.onOpen(() => {
          const requestPayload = this.buildRequestPayload(image, question);
          this.ws.send({
            data: JSON.stringify(requestPayload),
            fail: (err) => {
              this.isLoading = false;
              aiResponse.content = '发送请求失败。';
            }
          });
        });

        this.ws.onMessage((res) => {
          const data = JSON.parse(res.data);

          if (data.payload && data.payload.choices) {
            aiResponse.content += data.payload.choices.text[0].content;
            this.scrollToBottom();
          }

          if (data.header.status === 2) {
            this.isLoading = false;
            this.ws.close();
          }

          if (data.header.code !== 0) {
            aiResponse.content = `出错了: ${data.header.message}`;
            this.isLoading = false;
            this.ws.close();
          }
        });

        this.ws.onError((err) => {
          this.isLoading = false;
          aiResponse.content = '通信发生错误。';
        });

        this.ws.onClose(() => {
          this.isLoading = false;
        });
      },

      buildRequestPayload(imageBase64, question) {
        const textPayload = [];
        if (imageBase64) {
          textPayload.push({
            "role": "user",
            "content": imageBase64,
            "content_type": "image"
          });
        }
        textPayload.push({
          "role": "user",
          "content": question,
          "content_type": "text"
        });
        return {
          "header": {
            "app_id": this.APPID
          },
          "parameter": {
            "chat": {
              "domain": "image",
              "temperature": 0.5,
              "top_k": 4,
              "max_tokens": 2048
            }
          },
          "payload": {
            "message": {
              "text": textPayload
            }
          }
        };
      },

      getAuthorizationUrl() {
        return new Promise((resolve, reject) => {
          const host = "spark-api.cn-huabei-1.xf-yun.com";
          const path = "/v2.1/image";
          const date = new Date().toGMTString();
          const signatureOrigin = `host: ${host}\ndate: ${date}\nGET ${path} HTTP/1.1`;
          const signature = CryptoJS.HmacSHA256(signatureOrigin, this.APISecret).toString(CryptoJS.enc.Base64);
          const authorizationOrigin =
            `api_key="${this.APIKey}", algorithm="hmac-sha256", headers="host date request-line", signature="${signature}"`;
          const authorization = btoa(authorizationOrigin);
          const url =
            `wss://${host}${path}?authorization=${authorization}&date=${encodeURIComponent(date)}&host=${host}`;
          resolve(url);
        });
      },

      scrollToBottom() {
        this.$nextTick(() => {
          // uni-app的滚动API在不同端可能需要微调，这是比较通用的方式
          const lastMessageIndex = this.messages.length - 1;
          if (lastMessageIndex < 0) return;
          this.scrollTop = lastMessageIndex * 1000; // 一个足够大的值，确保能滚到底部
        });
      },

      previewImage(url) {
        uni.previewImage({
          urls: [url]
        });
      }
    },
    onReady() {
      // 头像文件路径确认
      // 确保/static/user-avatar.png 和 /static/ai-avatar.png 存在
    }
  };
</script>

<style lang="scss" scoped>
  .chat-container {
    display: flex;
    flex-direction: column;
    height: 100vh;
    background-color: #f2f2f2;
    font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
  }

  .chat-history {
    flex: 1;
    overflow-y: auto;
    padding: 30rpx;
    box-sizing: border-box;
    background: #f9f9f9;
  }

  .message-row {
    display: flex;
    align-items: flex-end;
    margin-bottom: 30rpx;

    .avatar {
      width: 80rpx;
      height: 80rpx;
      border-radius: 50%;
      box-shadow: 0 0 8rpx rgba(0, 0, 0, 0.1);
      flex-shrink: 0;
    }

    .message-content {
      max-width: calc(100% - 160rpx);
      display: flex;
      flex-direction: column;
      margin-left: 20rpx;
    }

    .image-bubble {
      max-width: 100%;
      border-radius: 12rpx;
      overflow: hidden;
      box-shadow: 0 2rpx 6rpx rgba(0, 0, 0, 0.1);
    }

    .image-bubble image {
      min-width: 200rpx;
      width: 100%;
      display: block;
    }

    .text-bubble {
      background-color: #ffffff;
      padding: 22rpx 26rpx;
      border-radius: 20rpx;
      font-size: 28rpx;
      line-height: 1.6;
      word-break: break-word;
      color: #333;
      box-shadow: 0 2rpx 6rpx rgba(0, 0, 0, 0.05);
    }

    &.is-me {
      flex-direction: row-reverse;

      .message-content {
        margin-right: 20rpx;
        margin-left: 0;
        align-items: flex-end;
      }

      .text-bubble {
        background-color: #f05454;
        color: #fff;
        box-shadow: 0 2rpx 6rpx rgba(240, 84, 84, 0.3);
      }
    }
  }

  .input-area {
    display: flex;
    align-items: center;
    padding: 20rpx 24rpx;
    background-color: #ffffff;
    border-top: 1rpx solid #ddd;

    input {
      flex: 1;
      height: 70rpx;
      background-color: #f0f0f0;
      border-radius: 35rpx;
      padding: 0 30rpx;
      margin-right: 20rpx;
      font-size: 28rpx;
      border: none;
    }

    .plus-icon {
      width: 50rpx;
      height: 50rpx;
      font-size: 44rpx;
      line-height: 50rpx;
      text-align: center;
      color: #999;
      margin-right: 20rpx;
    }

    .send-btn {
      width: 120rpx;
      height: 70rpx;
      line-height: 70rpx;
      font-size: 28rpx;
      background-color: #409EFF;
      color: white;
      text-align: center;
      border-radius: 35rpx;
      box-shadow: 0 2rpx 6rpx rgba(64, 158, 255, 0.3);

      &[disabled] {
        background-color: #a0cfff;
        box-shadow: none;
      }
    }
  }

  // 加载中的省略号动画
  .loading .dot {
    display: inline-block;
    width: 10rpx;
    height: 10rpx;
    background-color: #888;
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