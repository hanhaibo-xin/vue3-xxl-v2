<template>
  <div class="container">
    <div v-show="flag" class="welcome">
      <img class="title-img" src="@/assets/images/text.png" />
      <div class="high-score-welcome">历史最高分：{{ highScore }}分</div>
      <el-button class="change-btn" type="primary" @click="handleStart"
        >开始游戏</el-button
      >
    </div>
    <div class="score-container">
      <div class="score">当前：{{ score }}分</div>
      <div class="high-score">最高：{{ highScore }}分</div>
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
    <div class="game-buttons" v-show="!flag">
      <el-button class="restart-btn" type="warning" @click="handleRestart"
        >重新开始</el-button
      >
      <el-button class="end-btn" type="primary" @click="handleOver"
        >结束游戏</el-button
      >
    </div>
    <el-button class="change-btn" type="primary" v-show="flag" @click="handleStart"
      >开始游戏</el-button
    >
  </div>
</template>
<script lang="ts" setup>
import { onMounted, ref, computed } from "vue"
import Stage from "@/utils/stage"
import { ElMessageBox } from "element-plus"
import type { Action } from "element-plus"
const flag = ref(true)
const highScore = ref(0)

let width: number = document.documentElement.clientWidth
const games = ref<Stage>(new Stage(7, 7, (width - 20) / 7))
const data = computed(() => games.value.data)
const score = computed(() => games.value.score)

onMounted(() => {
  const stored = localStorage.getItem("iceCreamHighScore")
  if (stored) {
    highScore.value = parseInt(stored)
  }
})

// 开始游戏
const handleStart = () => {
  flag.value = false
  games.value.gameLoop(true)
}
// 选择方块
const handleClick = (item: any) => {
  games.value.click(item)
}
// 更新最高分
const updateHighScore = () => {
  if (score.value > highScore.value) {
    highScore.value = score.value
    localStorage.setItem("iceCreamHighScore", score.value.toString())
  }
}
// 重新开始
const handleRestart = () => {
  ElMessageBox.confirm("确定要重新开始游戏吗？", "提示", {
    confirmButtonText: "确定",
    cancelButtonText: "取消",
    type: "warning",
  })
    .then(() => {
      width = document.documentElement.clientWidth
      games.value = new Stage(7, 7, (width - 20) / 7)
      games.value.gameLoop(true)
    })
    .catch(() => {
      // 用户取消
    })
}
// 结束游戏
const handleOver = () => {
  updateHighScore()
  ElMessageBox.alert(`当前成绩：${score.value}分`, "雪糕消消大作战", {
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
      z-index: 100 !important;
      transform: scale(1.1) !important;
      box-shadow: 0 0 0 3px #ff69b4, 0 0 0 6px #ff1493;
      border-radius: 8px;
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
  .score-container {
    position: absolute;
    top: px2vw(30);
    left: 0;
    right: 0;
    display: flex;
    justify-content: space-around;
    padding: 0 px2vw(20);
  }
  .score,
  .high-score {
    font-weight: bold;
    font-size: px2vw(24);
    color: #fff;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
  }
  .high-score-welcome {
    margin-top: px2vw(20);
    font-weight: bold;
    font-size: px2vw(20);
    color: #fff;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
  }
  .game-buttons {
    position: absolute;
    bottom: px2vw(50);
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: px2vw(20);
  }
  .change-btn {
    position: absolute;
    bottom: px2vw(50);
    left: 50%;
    transform: translateX(-50%);
  }
}
</style>
