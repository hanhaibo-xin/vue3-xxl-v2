<template>
  <div class="container">
    <div v-show="flag" class="welcome">
      <img class="title-img" src="@/assets/images/text.png" />
      <div class="high-score-welcome">最高分：{{ highScore }}分</div>
      <el-button class="change-btn" type="primary" @click="handleStart"
        >开始游戏</el-button
      >
    </div>
    <div class="score-panel">
      <div class="current-score">当前分数：{{ score }}分</div>
      <div class="high-score-display">最高分：{{ highScore }}分</div>
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
      <el-button class="change-btn" type="primary" @click="handleOver"
        >结束游戏</el-button
      >
    </div>
  </div>
</template>
<script lang="ts" setup>
import { onMounted, reactive, ref, toRefs } from "vue"
import Stage from "@/utils/stage"
import { ElMessageBox } from "element-plus"
import type { Action } from "element-plus"
const flag = ref(true)
const highScore = ref(0)

const loadHighScore = () => {
  const saved = localStorage.getItem("xxl_high_score")
  if (saved) {
    highScore.value = parseInt(saved, 10)
  }
}

const saveHighScore = (score: number) => {
  if (score > highScore.value) {
    highScore.value = score
    localStorage.setItem("xxl_high_score", score.toString())
    return true
  }
  return false
}

onMounted(() => {
  loadHighScore()
})

let width: number = document.documentElement.clientWidth
const games = reactive(new Stage(7, 7, (width - 20) / 7))
const { data, score } = toRefs(games)

const handleStart = () => {
  flag.value = false
  games.gameLoop(true)
}
const handleClick = (item: any) => {
  games.click(item)
}
const handleRestart = () => {
  ElMessageBox.confirm("确定要重新开始游戏吗？当前进度将会丢失。", "提示", {
    confirmButtonText: "确定",
    cancelButtonText: "取消",
    type: "warning",
  })
    .then(() => {
      games.score = 0
      games.data.forEach((item: any) => {
        item.active = false
      })
      games.isSelect = false
      games.isHandle = false
      games.getMatrix()
      games.init(true)
    })
    .catch(() => {})
}
const handleOver = () => {
  const isNewRecord = saveHighScore(score.value)
  const message = isNewRecord
    ? `恭喜！新纪录！\n当前成绩：${score.value}分`
    : `当前成绩：${score.value}分\n最高分：${highScore.value}分`
  ElMessageBox.alert(message, "雪糕消消大作战", {
    confirmButtonText: "确定",
    callback: (action: Action) => {
      flag.value = true
    },
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
      z-index: 10;
      border: 3px solid #ff69b4;
      box-shadow: 0 0 0 2px #fff, 0 0 0 5px #ff1493, 0 0 15px rgba(255, 105, 180, 0.6);
      transform: scale(1.08);
      animation: pulse 0.6s ease-in-out infinite alternate;
    }
    @keyframes pulse {
      0% {
        transform: scale(1.05);
      }
      100% {
        transform: scale(1.1);
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
    display: flex;
    flex-direction: column;
    align-items: center;
    .title-img {
      margin-top: px2vw(150);
      width: 100%;
      transform: rotate(-10deg);
    }
    .high-score-welcome {
      margin-top: px2vw(30);
      font-weight: bold;
      font-size: px2vw(24);
      color: #fff;
      text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
    }
  }
  .score-panel {
    position: absolute;
    top: px2vw(20);
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: px2vw(5);
    .current-score {
      font-weight: bold;
      font-size: px2vw(24);
      color: #fff;
      text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
    }
    .high-score-display {
      font-weight: bold;
      font-size: px2vw(18);
      color: #ffd700;
      text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.5);
    }
  }
  .btn-group {
    position: absolute;
    bottom: px2vw(50);
    display: flex;
    gap: px2vw(20);
    .restart-btn {
      position: relative;
      left: auto;
      transform: none;
    }
    .change-btn {
      position: relative;
      left: auto;
      transform: none;
    }
  }
}
</style>
