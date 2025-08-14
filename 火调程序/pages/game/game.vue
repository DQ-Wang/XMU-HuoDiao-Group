<template>
  <div class="game-container">
    <!-- 故事选择界面 -->
    <div v-if="!activeStoryKey" class="story-selection-container">
      <div class="selection-header">
        <h1 class="selection-title">请选择你的故事</h1>
        <p class="selection-subtitle">每个选择都将影响结局...</p>
      </div>
      <div class="story-list">
        <div v-for="(story, key) in allStories" :key="key" class="story-card" @click="selectStory(key)"
          :style="{ backgroundImage: `url(${story.coverImage || story.start.image})` }">
          <div class="story-card-overlay"></div>
          <div class="story-card-content">
            <h3>{{ story.title }}</h3>
            <p class="story-desc">{{ story.description }}</p>
            <button class="select-button">开始故事</button>
          </div>
        </div>
      </div>
    </div>

    <!-- 游戏主界面 -->
    <div v-else class="game-play-container">
      <div class="image-container">
        <img v-for="(img, index) in imageStack" :key="img.id" :src="img.src" class="game-image"
          :class="{ 'visible': index === activeImageIndex }" @error="imageLoadError" />
      </div>

      <!-- 【优化】返回按钮容器 -->
      <div id="back-button-container" v-if="canGoBack" @click="goBack" title="返回上一步">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor">
          <path
            d="M12.5,8C9.85,8,7.45,9,5.6,10.6L2,7V16H11L7.38,12.38C8.77,11.22,10.54,10.5,12.5,10.5C16.04,10.5,19.05,12.81,20.1,16L22.47,15.22C21.08,11.03,17.15,8,12.5,8Z" />
        </svg>
      </div>

      <div class="ui-overlay">
        <div class="gradient-overlay"></div>

        <div class="story-container">
          <template v-if="!ending">
            <p class="story-text">{{ currentStoryText }}</p>
          </template>
          <template v-else>
            <div class="ending-container">
              <h2 class="ending-title">{{ endingTitle }}</h2>
              <p class="ending-text">{{ endingText }}</p>
              <div class="ending-buttons">
                <button @click="restartGame" class="restart-button">重新开始本章</button>
                <button @click="backToSelection" class="back-button">返回主菜单</button>
              </div>
            </div>
          </template>
        </div>

        <div v-if="!ending && showChoices" class="choices-container">
          <button v-for="(choice, index) in currentChoices" :key="index" @click="selectOption(choice.action)"
            class="choice-button">
            <span>{{ choice.text }}</span>
          </button>
        </div>

        <div v-if="isTransitioning" class="transition-container">
          <p class="transition-text">...</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        activeStoryKey: null,
        storyPoint: 'start',
        ending: null,
        showChoices: false,
        isTransitioning: false,
        imageStack: [{
          id: 0,
          src: ''
        }, {
          id: 1,
          src: ''
        }],
        activeImageIndex: 0,

        // --- 【新增】历史记录堆栈 ---
        historyStack: [],

        allStories: {
          femaleLine: {
            title: '卷发棒的“清晨陷阱”',
            start: {
              image: '/static/游戏图片/female_scene_start.png',
              text: '周日深夜，国光园区女生宿舍。为了准备明天早八重要的小组Pre，你定了清晨5点半的闹钟，打算在室友们醒来前悄悄使用卷发棒打理发型。',
              choices: [{
                text: '放弃卷发，安全第一',
                action: 'end_good'
              }, {
                text: '追求完美，偷偷使用',
                action: 'branch_B'
              }],
            },
            branch_B: {
              image: '/static/游戏图片/female_scene_use_iron.png',
              text: '实在想要完美效果。清晨困意浓重的你摸黑拿出卷发棒插上电。极度的疲倦让你竟然握着卷发棒、靠在椅背上又沉沉睡去！',
              transitionTo: 'branch_fire'
            },
            branch_fire: {
              image: '/static/游戏图片/female_scene_fire.png',
              text: '通着电、发烫的卷发棒直接贴在了资料和纸巾上...你被一股强烈的焦糊味呛醒，睁眼便看到书桌上的资料正冒着黑烟！',
              choices: [{
                text: '迅速拔电源，呼救并用灭火器扑救',
                action: 'end_medium'
              }, {
                text: '手忙脚乱，用手或袖子去拍打',
                action: 'end_bad'
              }, {
                text: '被吓傻，看着火苗不知所措',
                action: 'end_bad'
              }]
            },
            end_good: {
              image: '/static/游戏图片/female_ending_good.png',
              title: '睿智之选，清晨无忧',
              text: '你明智地规避了风险，安全地打理了头发，Pre顺利，心情踏实。后来听闻其他楼的火险通报，你更庆幸自己的选择，成为倡导安全美妆的“明白人”。'
            },
            end_medium: {
              image: '/static/游戏图片/female_ending_medium.png',
              title: '清晨惊魂，代价惨痛',
              text: '火警惊动整栋楼，书桌部分烧毁。校方给予严厉批评与赔偿，并强制你完成消防课程和安全巡查。事件促使你深刻悔悟，最终从规则挑战者转变为安全守护者。'
            },
            end_bad: {
              image: '/static/游戏图片/female_ending_bad.png',
              title: '逆光前行，淬火之证',
              text: '处置失误引发严重火情，你受到留校察看、大额赔偿和120小时消防服务的处分。经历磨砺后，你最终以宿舍楼义务安全员的身份完成了救赎，将教训转化为守护他人的力量。'
            }
          },
          // ... 此处应包含您其他的四个故事 ...
          maleLine: {
            title: '插座狂魔的插线板陷阱',
            start: {
              image: '/static/游戏图片/male_scene_start.png',
              text: '深夜，为冲刺Pre，你书桌上的旧台灯闪烁不断。你图方便，把床下一个老旧破损的插线板扯过来，压在书本下，接上台灯继续工作。',
              choices: [{
                text: '意识到问题，立刻整改换新',
                action: 'end_good'
              }, {
                text: '嫌麻烦，能用就行',
                action: 'branch_B'
              }]
            },
            branch_B: {
              image: '/static/游戏图片/male_scene_danger.png',
              text: '你只是稍微挪开书本，根本没解开缠绕的线。深夜，高负载下的老旧插线板，在被压住的部分内部短路，突然“嘭”的一声迸出电火花！',
              transitionTo: 'branch_fire'
            },
            branch_fire: {
              image: '/static/游戏图片/male_scene_fire.png',
              text: '火花瞬间点燃了压在电线上的干燥草稿纸和书本！你的第一反应是？',
              choices: [{
                text: '保持冷静，大喊呼救并用灭火器',
                action: 'end_medium'
              }, {
                text: '惊慌失措，用手扑打或保护电脑',
                action: 'end_bad'
              }, {
                text: '吓懵了，下意识用水泼',
                action: 'end_bad'
              }]
            },
            end_good: {
              image: '/static/游戏图片/male_ending_good.png',
              title: '稳字当先，领袖担当',
              text: '你消除了隐患，平安度过。几天后宿舍卫检，你因规范用电被表扬。你牵头成立了“用电安全自查小分队”，成为楼栋“安全之星”。'
            },
            end_medium: {
              image: '/static/游戏图片/male_ending_medium.png',
              title: '灼痕烙印，男儿担当',
              text: '你及时控制了火势，但宿舍仍一片狼藉，并因此被处分。学校全面清查老旧设备，看着那块焦痕，你深深理解了什么叫“细节决定安危”。'
            },
            end_bad: {
              image: '/static/游戏图片/male_ending_bad.png',
              title: '烬夜明灯',
              text: '失控大火将宿舍焚为灰烬。校方给予记过处分和公益劳动赔偿。一年后，已成为安全委员的你，在新生安全教育会上举起烧毁的残骸，沉声道：“这堆废铁，成了我生命中的一盏灯。”'
            }
          },
          kitchen: {
            title: '宿舍厨房的代价',
            start: {
              image: '/static/游戏图片/kitchen_scene_start.png',
              text: '周五傍晚，舍友小李正用1200瓦的电热锅煮火锅，发烫的插排上挤满插头。他笑着说：“就煮10分钟，宿管刚查过。”',
              choices: [{
                text: '坚决阻止，拔掉插头',
                action: 'branch_A'
              }, {
                text: '帮忙照看，提醒时间',
                action: 'branch_BC'
              }, {
                text: '漠不关心，戴上耳机',
                action: 'branch_BC'
              }],
            },
            branch_A: {
              image: '/static/游戏图片/kitchen_scene_unplug.png',
              text: '小李不情愿地关了锅。半小时后，刺鼻的塑料味惊醒了你，未拔的插排持续发热，磨损的电线短路引燃了纸巾盒！',
              choices: [{
                text: '拉电闸，湿毛巾闷灭',
                action: 'end_good_4_1'
              }, {
                text: '直接冲出宿舍',
                action: 'end_medium_4_2'
              }],
            },
            branch_BC: {
              image: '/static/游戏图片/kitchen_scene_overflow.png',
              text: '十分钟后，滚烫的汤汁溢出溅到加热丝，爆出火花点燃了油腻的课本，火苗窜向窗帘！浓烟瞬间涌来。',
              choices: [{
                text: '湿毛巾捂鼻，走疏散通道',
                action: 'end_medium_4_3'
              }, {
                text: '惊慌中冲向电梯',
                action: 'end_bad_4_4'
              }],
            },
            end_good_4_1: {
              image: '/static/游戏图片/kitchen_ending_good.png',
              title: '临危不乱，化险为夷',
              text: '湿毛巾闷灭小火，只留下焦黑的纸盒。宿管和辅导员赶来，虽通报批评，但也因你们正确的“先断电、再扑救”而口头赞扬。'
            },
            end_medium_4_2: {
              image: '/static/游戏图片/kitchen_ending_medium_1.png',
              title: '逃生之后，满目疮痍',
              text: '你们跑下楼才报警，小火已引燃床铺。消防员赶到时，书桌和床铺全毁。学校给了你们记大过的处分和高额赔偿。'
            },
            end_medium_4_3: {
              image: '/static/游戏图片/kitchen_ending_medium_2.png',
              title: '教科书式逃生',
              text: '你们报警清晰、湿毛巾捂鼻逃生，消防车很快赶到。宿舍虽一片狼藉，但无人受伤。学校勒令你们休学一周，去消防大队学习，深刻反思。'
            },
            end_bad_4_4: {
              image: '/static/游戏图片/kitchen_ending_bad.png',
              title: '致命捷径',
              text: '你们冲进电梯，但火灾中断了电源，被困在浓烟中。消防员撬门救出时，你们已吸入大量浓烟。“别让捷径变成绝路”，消防员的话成了你们的终身警钟。'
            }
          },
          battery: {
            title: '电池惊魂',
            start: {
              image: '/static/游戏图片/battery_scene_start.png',
              text: '深夜，你看见舍友小李把有磕碰的电动车锂电池塞在墙角充电，旁边就是零食箱。你劝说他，他却说：“新电池，就一晚，我定闹钟。”',
              choices: [{
                text: '坚决上报宿管处理',
                action: 'end_good'
              }, {
                text: '妥协容忍，帮忙移开杂物',
                action: 'branch_B'
              }, {
                text: '帮忙优化，整理电线散热',
                action: 'branch_B'
              }],
            },
            branch_B: {
              image: '/static/游戏图片/battery_scene_sleep.png',
              text: '小李睡得太沉，按掉了闹钟。凌晨2点，电池过充开始发烫，冒出黑烟。一股奇怪的“臭鸡蛋味”把你呛醒！',
              transitionTo: 'branch_fire'
            },
            branch_fire: {
              image: '/static/游戏图片/battery_scene_fire.png',
              text: '墙角正冒着浓烟，热浪扑面而来！“小李！醒醒！电池着火了！”你的应急措施是？',
              choices: [{
                text: '灭火器、关电闸、报警',
                action: 'end_medium'
              }, {
                text: '把发烫的电池扔出去',
                action: 'end_bad'
              }, {
                text: '打开宿舍门往外跑',
                action: 'end_bad'
              }],
            },
            end_good: {
              image: '/static/游戏图片/battery_ending_good.png',
              title: '较真的“守护者”',
              text: '宿管及时处理了隐患。几天后，在消防讲座上看到电池热失控的恐怖演示，小李向你道歉。你们和好如初，还一起成了宿舍安全监督员。'
            },
            end_medium: {
              image: '/static/游戏图片/battery_ending_medium.png',
              title: '及时的代价',
              text: '灭火器压制了火势，但你们仍被处分。在消防队，看着烧毁的电池，你们才明白，最好的办法是永远别把电池拿进宿舍。'
            },
            end_bad: {
              image: '/static/游戏图片/battery_ending_bad.png',
              title: '致命的选择',
              text: '移动电池导致爆炸，开门放烟堵塞楼道。你的错误选择造成了更严重的后果，不仅宿舍被毁，还危及邻里。惨痛的教训让你铭记终身。'
            }
          },
          cigarette: {
            title: '烟头与锁死的门',
            start: {
              image: '/static/游戏图片/cigarette_scene_start.png',
              text: '舍友小李在寝室抽烟，未掐灭的烟头就在桌上，桌底是易燃的废纸盒，门后堆满行李箱堵住通道。你劝说他，他却不以为然。',
              choices: [{
                text: '强制清理隐患',
                action: 'end_good'
              }, {
                text: '只是口头提醒',
                action: 'branch_B'
              }, {
                text: '视而不见，独善其身',
                action: 'branch_B'
              }],
            },
            branch_B: {
              image: '/static/游戏图片/cigarette_scene_fire.png',
              text: '深夜，未灭的烟头滚落，引燃了纸盒。你被浓烟呛醒，却发现唯一的门被行李箱堵死，根本无法逃离！',
              choices: [{
                text: '退守阳台，堵门缝报警',
                action: 'end_medium'
              }, {
                text: '用凳子猛砸行李箱',
                action: 'end_bad'
              }],
            },
            end_good: {
              image: '/static/游戏图片/cigarette_ending_good.png',
              title: '无患则安',
              text: '你强硬地清理了隐患。一周后，你们因“门口畅通、无杂物”被辅导员点名表扬，成为“无隐患宿舍”的榜样。'
            },
            end_medium: {
              image: '/static/游戏图片/cigarette_ending_medium.png',
              title: '阳台上的生机',
              text: '消防员从阳台救下了你们。小李因违规被记大过，你也因未坚决制止被警告。但你们正确的自救方式，是这次不幸中的万幸。'
            },
            end_bad: {
              image: '/static/游戏图片/cigarette_ending_bad.png',
              title: '堵死的生路',
              text: '砸门导致火势因空气涌入而变大，有毒浓烟将你们呛晕。最终虽被救出，但都因灼伤和中毒住院。侥幸心理，最终让你们付出了惨痛的代价。'
            }
          }
        },
      };
    },
    computed: {
      activeStoryData() {
        return this.activeStoryKey ? this.allStories[this.activeStoryKey] : {};
      },
      currentStoryNode() {
        return this.activeStoryKey ? this.activeStoryData[this.storyPoint] : {};
      },
      currentStoryText() {
        return this.ending ? '' : this.currentStoryNode.text;
      },
      currentChoices() {
        return (this.ending || this.isTransitioning) ? [] : this.currentStoryNode.choices || [];
      },
      endingTitle() {
        return this.ending ? this.activeStoryData[this.ending]?.title : '';
      },
      endingText() {
        return this.ending ? this.activeStoryData[this.ending]?.text : '';
      },

      // --- 【新增】判断是否可以返回 ---
      canGoBack() {
        return this.historyStack.length > 0;
      }
    },
    watch: {
      storyPoint(newPoint, oldPoint) {
        if (newPoint && newPoint !== 'end' && this.activeStoryData[newPoint]?.image) {
          // 只有在非返回操作时才记录历史
          if (this.historyStack[this.historyStack.length - 1] !== oldPoint) {
            // This check is a bit tricky, a flag might be better. Let's manage history in methods.
          }
          this.updateImage(this.activeStoryData[newPoint].image);
        }
      },
      ending(newEnding) {
        if (newEnding && this.activeStoryData[newEnding]?.image) {
          this.updateImage(this.activeStoryData[newEnding].image);
        }
      }
    },
    methods: {
      updateImage(newSrc) {
        /* ... 此方法保持不变 ... */
        const inactiveIndex = 1 - this.activeImageIndex;
        const inactiveImage = this.imageStack[inactiveIndex];
        const loader = new Image();
        loader.src = newSrc;
        loader.onload = () => {
          inactiveImage.src = newSrc;
          this.activeImageIndex = inactiveIndex;
        };
        // loader.onerror = () => { inactiveImage.src = '/static/游戏图片/default_error.jpg'; this.activeImageIndex = inactiveIndex; };
      },
      selectStory(key) {
        this.activeStoryKey = key;
        this.historyStack = []; // 清空历史
        this.restartGame(); // 重用restart逻辑来初始化
      },
      selectOption(action) {
        // 【修改】在做出选择时，记录当前步骤到历史
        this.historyStack.push({
          point: this.storyPoint,
          ending: this.ending
        });

        this.showChoices = false;
        if (action.startsWith('end_')) {
          this.ending = action;
          this.storyPoint = 'end';
        } else {
          this.storyPoint = action;
          const nextNode = this.currentStoryNode;
          if (nextNode.transitionTo) {
            this.isTransitioning = true;
            setTimeout(() => {
              this.isTransitioning = false;
              this.storyPoint = nextNode.transitionTo;
              this.displayChoicesWithDelay();
            }, 4000);
          } else {
            this.displayChoicesWithDelay();
          }
        }
      },
      // --- 【新增】返回上一步的方法 ---
      goBack() {
        if (!this.canGoBack) return;

        const lastState = this.historyStack.pop();
        this.showChoices = false;

        this.ending = lastState.ending;
        this.storyPoint = lastState.point;

        this.$nextTick(() => {
          this.displayChoicesWithDelay();
        });
      },

      restartGame() {
        this.historyStack = []; // 清空历史
        this.ending = null;
        this.storyPoint = 'start';
        this.showChoices = false;
        this.updateImage(this.activeStoryData.start.image); // 手动更新初始图片
        this.displayChoicesWithDelay();
      },
      backToSelection() {
        this.activeStoryKey = null;
        this.historyStack = [];
      },
      displayChoicesWithDelay() {
        /* ... 此方法保持不变 ... */
        if (this.currentStoryNode.choices) {
          setTimeout(() => {
            if (!this.ending) this.showChoices = true;
          }, 1500); // 缩短延迟，节奏更快
        }
      },
      imageLoadError(event) {
        /* ... 此方法保持不变 ... */
        console.error("图片加载失败:", event.target.src);
        // event.target.src = '/static/游戏图片/default_error.jpg';
      }
    },
  };
</script>

<style scoped>
  /* --- 全局设计变量 --- */
  :root {
    --bg-color: #e6f7fa;
    --text-color: #1a3c4e;
    --accent-color: #2ec4b6;
    --accent-gradient: linear-gradient(135deg, #2ec4b6 0%, #53d6ff 100%);
    --glass-bg: rgba(255, 255, 255, 0.7);
    --border-color: rgba(46, 196, 182, 0.15);
    --font-display: 'Sarpanch', 'Noto Sans SC', sans-serif;
    --font-body: 'Noto Sans SC', sans-serif;
  }

  /* 故事选择界面美化 */
  .story-selection-container {
    background: var(--accent-gradient);
    min-height: 100vh;
    padding: 7vh 0 5vh 0;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-start;
    animation: fadeIn 1.2s;
  }

  .selection-header {
    margin-bottom: 40px;
  }

  .selection-title {
    font-family: var(--font-display);
    font-weight: 700;
    font-size: 2.8em;
    color: #55007f;
    letter-spacing: 2px;
    text-shadow: 0 4px 24px rgba(255, 0, 0, 0.2);
  }

  .selection-subtitle {
    color: #55007f;
    font-size: 1.2em;
    margin-top: 10px;
    letter-spacing: 1px;
  }

  .story-list {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(290px, 1fr));
    gap: 32px;
    width: 100%;
    max-width: 1200px;
    margin: 0 auto;
  }

  .story-card {
    position: relative;
    /* 移除半透明磨砂背景 */
    background: #fff;
    /* 纯白或可用 var(--bg-color) */
    border-radius: 22px;
    overflow: hidden;
    box-shadow: 0 8px 32px rgba(46, 196, 182, 0.12), 0 2px 8px rgba(0, 0, 0, 0.06);
    cursor: pointer;
    transition: transform 0.25s, box-shadow 0.25s;
    min-height: 370px;
    display: flex;
    flex-direction: column;
    justify-content: flex-end;
    border: 2px solid var(--border-color);
    background-size: cover;
    background-position: center;
  }

  .story-card-overlay {
    position: absolute;
    inset: 0;
    background: linear-gradient(0deg, rgba(46, 196, 182, 0.28) 70%, rgba(255, 255, 255, 0.05) 100%);
    pointer-events: none;
  }

  .story-card-content {
    position: relative;
    z-index: 2;
    padding: 32px 24px 28px 24px;
    background: transparent;
    /* 取消内容区的磨砂和半透明 */
    border-radius: 0 0 20px 20px;
    box-shadow: none;
    backdrop-filter: none;
    display: flex;
    flex-direction: column;
    align-items: center;
    /* 新增：让内容居中 */
  }

  .story-card h3 {
    font-family: var(--font-display);
    font-size: 1.35em;
    color: var(--accent-color);
    margin-bottom: 10px;
    font-weight: 700;
    letter-spacing: 1px;
    text-align: center;
    /* 新增：标题居中 */
    width: 100%;
    /* 新增：宽度撑满父容器 */
  }

  .story-desc {
    color: var(--text-color);
    font-size: 1em;
    margin-bottom: 22px;
    min-height: 48px;
    line-height: 1.6;
  }

  .select-button {
    background: var(--accent-gradient);
    color: #fff;
    font-family: var(--font-display);
    font-size: 1.08em;
    font-weight: 600;
    border: none;
    border-radius: 8px;
    padding: 12px 32px;
    box-shadow: 0 2px 12px rgba(46, 196, 182, 0.13);
    cursor: pointer;
    transition: background 0.2s, transform 0.2s, box-shadow 0.2s;
    outline: none;
  }

  .select-button:hover {
    background: linear-gradient(135deg, #53d6ff 0%, #2ec4b6 100%);
    color: #fff;
    transform: translateY(-2px) scale(1.04);
    box-shadow: 0 6px 24px rgba(46, 196, 182, 0.18);
  }

  .story-card:hover {
    transform: translateY(-8px) scale(1.03);
    box-shadow: 0 16px 48px rgba(46, 196, 182, 0.22), 0 2px 8px rgba(0, 0, 0, 0.08);
    border-color: #2ec4b6;
  }

  @media (max-width: 768px) {
    .story-list {
      grid-template-columns: 1fr;
      gap: 22px;
    }

    .story-card-content {
      padding: 22px 14px 18px 14px;
    }

    .story-card {
      min-height: 260px;
    }

    .selection-title {
      font-size: 2em;
    }
  }

  /* --- 游戏主界面 --- */
  .game-play-container {
    max-width: 900px;
    margin: auto;
    position: relative;
    width: 100%;
    height: 100vh;
    display: flex;
    flex-direction: column;
    overflow: hidden;
    background: #e6f7fa;
  }

  .image-container {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: #000;
    z-index: 1;
  }

  .game-image {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    opacity: 0;
    transition: opacity 1.2s;
  }

  .game-image.visible {
    opacity: 1;
  }

  /* 优化渐变蒙层为蓝绿色调，仅底部渐变 */
  .gradient-overlay {
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 45%;
    background: linear-gradient(to top, rgba(46, 196, 182, 0.92) 0%, rgba(46, 196, 182, 0.25) 60%, transparent 100%);
    pointer-events: none;
    z-index: 2;
  }

  /* 浮层整体居底 */
  .ui-overlay {
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    z-index: 3;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding-bottom: 5vh;
  }

  /* 卡片化文本区 */
  .story-container {
    background: #fff;
    border-radius: 18px 18px 0 0;
    box-shadow: 0 4px 24px rgba(46, 196, 182, 0.13);
    padding: 28px 32px 18px 32px;
    margin-bottom: 0;
    width: 90%;
    max-width: 600px;
    min-height: 80px;
    color: var(--text-color);
    text-align: left;
    animation: fadeInText 0.8s;
  }

  @keyframes fadeInText {
    from {
      opacity: 0;
      transform: translateY(30px);
    }

    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  .story-text {
    font-size: 1.18em;
    line-height: 1.8;
    margin: 0;
    text-shadow: none;
  }

  /* 卡片化选项区 */
  .choices-container {
    background: transparent;
    border-radius: 0;
    box-shadow: none;
    width: auto;
    max-width: none;
    display: flex;
    flex-direction: row;
    gap: 18px;
    /* 缩小间距以适应更多按钮 */
    justify-content: center;
    align-items: flex-end;
    padding: 32px 0 0 0;
    margin-bottom: 0;
    animation: slideInUp 0.8s;
  }

  @keyframes slideInUp {
    from {
      opacity: 0;
      transform: translateY(30px);
    }

    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  .choice-button {
    width: 120px;
    height: 60px;
    min-width: 90px;
    min-height: 48px;
    max-width: 180px;
    max-height: 80px;
    border-radius: 14px;
    /* 方形圆角 */
    background: var(--accent-gradient);
    color: #fff;
    font-size: 1.08em;
    font-family: var(--font-display);
    font-weight: 700;
    border: none;
    box-shadow: 0 2px 12px rgba(46, 196, 182, 0.13);
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: background 0.18s, color 0.18s, transform 0.18s, box-shadow 0.18s;
    outline: none;
    text-align: center;
    letter-spacing: 1px;
    white-space: normal;
    opacity: 1;
    margin: 0;
    padding: 0 10px;
    line-height: 1.4;
    word-break: break-all;
  }

  .choice-button:active,
  .choice-button.clicked,
  .choice-button:disabled {
    background: #b2f0e6 !important;
    color: #aaa !important;
    transform: scale(0.97);
    cursor: not-allowed;
    opacity: 0.7;
  }

  .choice-button:hover:not(:disabled) {
    background: linear-gradient(135deg, #53d6ff 0%, #2ec4b6 100%);
    color: #fff;
    transform: scale(1.08);
    box-shadow: 0 6px 24px rgba(46, 196, 182, 0.18);
  }

  /* 选项文字自动换行且居中 */
  .choice-button span,
  .choice-button {
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    text-align: center;
    word-break: break-all;
    padding: 0 2px;
  }

  /* 返回按钮更突出 */
  #back-button-container {
    position: absolute;
    top: 24px;
    left: 24px;
    z-index: 10;
    width: 48px;
    height: 48px;
    background: rgba(46, 196, 182, 0.7);
    border-radius: 50%;
    display: flex;
    justify-content: center;
    align-items: center;
    cursor: pointer;
    transition: all 0.3s;
    box-shadow: 0 2px 12px rgba(46, 196, 182, 0.18);
  }

  #back-button-container:hover {
    background: #2ec4b6;
    transform: scale(1.1);
  }

  #back-button-container svg {
    width: 28px;
    height: 28px;
    color: #fff;
    transition: color 0.3s;
  }

  #back-button-container:hover svg {
    color: #000;
  }

  /* 结局页面按钮高亮 */
  .ending-buttons .restart-button,
  .ending-buttons .back-button {
    font-size: 1.15em;
    font-weight: 700;
    box-shadow: 0 0 18px 2px #53d6ff55;
    animation: pulse 1.5s infinite alternate;
  }

  @keyframes pulse {
    from {
      box-shadow: 0 0 18px 2px #53d6ff55;
    }

    to {
      box-shadow: 0 0 32px 8px #2ec4b655;
    }
  }

  @media (max-width: 768px) {

    .story-container,
    .choices-container {
      padding: 18px 8px;
      width: 98%;
    }

    #back-button-container {
      top: 10px;
      left: 10px;
      width: 40px;
      height: 40px;
    }
  }
</style>