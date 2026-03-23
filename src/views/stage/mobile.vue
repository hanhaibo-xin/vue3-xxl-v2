<template>
  <div class="container">
    <div v-show="flag" class="welcome">
      <img class="title-img" src="@/assets/images/text.png" />
      <div class="high-score-display">历史最高分: {{ highScore }}分</div>
      <el-button class="change-btn" type="primary" @click="handleStart"
        >开始游戏</el-button
      >
    </div>
    <div class="score-board">
      <div class="current-score">当前: {{ score }}分</div>
      <div class="high-score">最高: {{ highScore }}分</div>
    </div>
    <div class="stage">
      <div
        v-for="item in data"
        :style="{
          left: `${item.positionLeft}px`,
          top: `${item.positionTop}px`,
        }"
        :key="item.key"
        :class="[
          'square',
          `type${item.type}`,
          `scale${item.scale}`,
          { active: item.active },
        ]"
        @click="handleClick(item)"
      ></div>
    </div>
    <div class="btn-group">
      <el-button class="restart-btn" type="warning" @click="handleRestart"
        >重新开始</el-button
      >
      <el-button class="over-btn" type="primary" @click="handleOver"
        >结束游戏</el-button
      >
    </div>
  </div>
</template>
<script lang="ts" setup>
import { onMounted, reactive, ref, toRefs, watch } from "vue"
import Stage from "@/utils/stage"
import { ElMessageBox } from "element-plus"
import type { Action } from "element-plus"

const flag = ref(true)
const highScore = ref(0)
const HIGH_SCORE_KEY = 'iceCreamGameHighScore'

let width: number = document.documentElement.clientWidth
const games = reactive(new Stage(7, 7, (width - 20) / 7))
const { data, score } = toRefs(games)

// 从localStorage读取最高分
onMounted(() => {
  const saved = localStorage.getItem(HIGH_SCORE_KEY)
  if (saved) {
    highScore.value = parseInt(saved, 10) || 0
  }
})

// 监听分数变化，更新最高分
watch(score, (newScore) => {
  if (newScore > highScore.value) {
    highScore.value = newScore
    localStorage.setItem(HIGH_SCORE_KEY, newScore.toString())
  }
})

// 开始游戏
const handleStart = () => {
  flag.value = false
  games.gameLoop(true)
}
// 选择方块
const handleClick = (item: any) => {
  games.click(item)
}
// 结束游戏
const handleOver = () => {
  const isNewRecord = score.value > 0 && score.value >= highScore.value
  const message = isNewRecord
    ? `🎉 恭喜！你创造了新纪录！<br>当前成绩：${score.value}分`
    : `当前成绩：${score.value}分<br>历史最高分：${highScore.value}分`
  ElMessageBox.alert(message, "雪糕消消大作战", {
    confirmButtonText: "确定",
    dangerouslyUseHTMLString: true,
    callback: (action: Action) => {
      flag.value = true
    },
  })
}

// 重新开始游戏
const handleRestart = () => {
  ElMessageBox.confirm(
    '确定要重新开始游戏吗？当前分数将被清零。',
    '重新开始',
    {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning',
    }
  )
    .then(() => {
      // 重置游戏状态
      games.score = 0
      games.isSelect = false
      games.target1 = { active: false }
      games.target2 = {}
      games.isHandle = false
      // 重新生成棋盘
      games.init(true)
      games.gameLoop(true)
    })
    .catch(() => {
      // 用户取消，不做任何操作
    })
}
</script>
<style lang="scss" scoped>
@function px2vw($px) {
  @return calc($px / 375 * 100vw);
}
.container {
  display: flex;
  align-items: center;
  justify-content: center;
  width: px2vw(375);
  height: 100vh;
  background-color: #ccc;
  background-image: url("@/assets/images/bg.jpg");
  background-size: 100% 100%;
  .stage {
    position: relative;
    width: calc(100vw - 20px);
    height: calc(100vw - 20px);
    background-color: rgba(116, 183, 187, 0.7);
    border: 1px solid #fff;
    .square {
      position: absolute;
      width: px2vw(calc(355 / 7));
      height: px2vw(calc(355 / 7));
      border: 1px solid #fff;
      box-sizing: border-box;
      transition: 0.5s;
      background-repeat: no-repeat;
      background-size: 85% 85%;
      background-position: center;
    }
    .type0 {
      background-image: url("@/assets/images/0.png");
    }
    .type1 {
      background-image: url("@/assets/images/1.png");
    }
    .type2 {
      background-image: url("@/assets/images/2.png");
    }
    .type3 {
      background-image: url("@/assets/images/3.png");
    }
    .type4 {
      background-image: url("@/assets/images/4.png");
    }
    .type5 {
      background-image: url("@/assets/images/5.png");
    }
    .type6 {
      background-image: url("@/assets/images/6.png");
    }
    .active {
      background-color: transparent;
      border: 3px solid #ff69b4;
      box-shadow: 0 0 0 3px #ffb6c1, 0 0 15px rgba(255, 105, 180, 0.6);
      transform: scale(1.15);
      z-index: 10;
      animation: pulse 0.6s ease-in-out infinite alternate;
    }
    @keyframes pulse {
      from {
        transform: scale(1.1);
      }
      to {
        transform: scale(1.2);
      }
    }
    .scale0 {
      transform: scale(0);
      z-index: 0;
    }
    .scale1 {
      transform: scale(1);
      z-index: 1;
    }
  }
  .welcome {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-image: url("@/assets/images/bg.jpg");
    background-size: 100% 100%;
    z-index: 2;
    overflow: hidden;
    .title-img {
      margin-top: px2vw(200);
      width: 100%;
      transform: rotate(-10deg);
    }
  }
  .score-board {
    position: absolute;
    top: px2vw(20);
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: px2vw(5);
    .current-score {
      font-weight: bold;
      font-size: px2vw(28);
      color: #fff;
    }
    .high-score {
      font-size: px2vw(20);
      color: #ffd700;
      text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
    }
  }
  .high-score-display {
    margin-top: px2vw(30);
    font-size: px2vw(24);
    color: #ffd700;
    font-weight: bold;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
  }
  .btn-group {
    position: absolute;
    bottom: px2vw(50);
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: px2vw(15);
  }
}
</style>
