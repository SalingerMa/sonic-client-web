<script setup>
/*
 *  Copyright (C) [SonicCloudOrg] Sonic Project
 *
 *  Licensed under the Apache License, Version 2.0 (the "License");
 *  you may not use this file except in compliance with the License.
 *  You may obtain a copy of the License at
 *
 *         http://www.apache.org/licenses/LICENSE-2.0
 *
 *  Unless required by applicable law or agreed to in writing, software
 *  distributed under the License is distributed on an "AS IS" BASIS,
 *  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 *  See the License for the specific language governing permissions and
 *  limitations under the License.
 *
 */
import { nextTick, onBeforeUnmount, onMounted, ref } from 'vue';
import { ElMessage } from 'element-plus';
import { useI18n } from 'vue-i18n';
import IOSRemote from './IOSRemote.vue';

const { t: $t } = useI18n();

// let isPress = false;
let mouseMoveTime = 0;
const startPosition = { x: 0, y: 0 };
let parentNode = null;

const isPress = ref(false);
const _layoutSplitInfo = window.localStorage.getItem('layoutSplitInfo');
const _tabPosition = window.localStorage.getItem('tabPosition');
const tabPosition = ref(_tabPosition || 'left'); // left,top
// 分屏默认值
const layoutSplitInfo = ref(
  _layoutSplitInfo
    ? JSON.parse(_layoutSplitInfo)
    : {
        left: 33,
        last_left: 33,
        top: 316,
        last_top: 316,
      }
);
const canvasRectInfo = ref(
  tabPosition.value == 'left'
    ? { width: '100%', height: 'auto' }
    : {
        width: 'auto',
        height: `${layoutSplitInfo.value.top}px`,
      }
);
const swithLayout = () => {
  // console.log('swithLayout!!', tabPosition.value);
  if (tabPosition.value == 'left') {
    // 变竖屏
    tabPosition.value = 'top';
    canvasRectInfo.value = {
      width: 'auto',
      height: `${layoutSplitInfo.value.top}px`,
    };
  } else {
    // 变横屏
    tabPosition.value = 'left';
    canvasRectInfo.value = { width: '100%', height: 'auto' };
  }
  window.localStorage.setItem('tabPosition', tabPosition.value);
  ElMessage.success({
    message: $t('indexIOSTS.successText'),
  });
};
const lineMouseup = (event) => {
  // console.log('lineMouseup', event.clientX, event.clientY);
  isPress.value = false;
  if (tabPosition.value == 'left') {
    layoutSplitInfo.value.last_left = layoutSplitInfo.value.left;
  } else if (tabPosition.value == 'top') {
    layoutSplitInfo.value.last_top = layoutSplitInfo.value.top;
  }
  saveLastSplitObj();
};
const lineMouseleave = (event) => {
  if (!isPress.value) {
    return;
  }
  // console.log('lineMouseleave', event.clientX, event.clientY);
  // 由于左右滑动有可能误触发该事件，所以只处理上下滑动的情况
  if (tabPosition.value == 'top') {
    isPress.value = false;
    layoutSplitInfo.value.last_top = layoutSplitInfo.value.top;
  }
  saveLastSplitObj();
};
const lineMousedown = (event) => {
  // console.log('lineMousedown', event.clientX, event.clientY);
  event.preventDefault();
  // event.stopPropagation()
  isPress.value = true;
  startPosition.x = event.clientX;
  startPosition.y = event.clientY;
  parentNode = event.target.parentNode; // 记录分割线的父组件，防止移动的时候变化
};
const lineMousemove = (event) => {
  event.preventDefault();
  // event.stopPropagation()
  if (isPress.value) {
    // console.log('lineMousemove', event.clientX, event.clientY);
    if (mouseMoveTime < 2) {
      mouseMoveTime++;
    } else {
      if (tabPosition.value == 'left') {
        // 水平移动
        const deltaX = event.clientX - startPosition.x;
        // console.log('deltaX', deltaX);
        handleSplit(deltaX);
      } else if (tabPosition.value == 'top') {
        // 垂直移动
        const deltaY = event.clientY - startPosition.y;
        // console.log('deltaY', deltaY);
        handleSplit(deltaY);
      }
      mouseMoveTime = 0;
    }
  }
};
const handleSplit = (variate) => {
  if (tabPosition.value == 'left') {
    const rect = parentNode.getBoundingClientRect();
    const percent = ((variate * 100) / rect.width).toFixed(2);
    // 边界处理
    if (
      (percent < 0 && layoutSplitInfo.value.left <= 20) ||
      (percent >= 0 && layoutSplitInfo.value.left >= 70)
    ) {
      return;
    }

    layoutSplitInfo.value.left =
      layoutSplitInfo.value.last_left + Number(percent);
    // console.log('variate', percent, layoutSplitInfo.value.left);
  } else if (tabPosition.value == 'top') {
    const rect = parentNode.getBoundingClientRect();
    const percent = variate;
    const canvas = document.getElementById('iosCap');
    const rectCanvas = canvas.getBoundingClientRect();

    // 边界处理
    if (
      (percent < 0 && layoutSplitInfo.value.top <= 300) ||
      (percent >= 0 && rectCanvas.width >= rect.width - 100)
    ) {
      return;
    }

    layoutSplitInfo.value.top =
      layoutSplitInfo.value.last_top + Number(percent);
    // 直接应用
    canvasRectInfo.value.height = `${layoutSplitInfo.value.top}px`;
    // console.log('variate', percent, layoutSplitInfo.value.top);
  }
};
const saveLastSplitObj = () => {
  // 储存最后比例
  window.localStorage.setItem(
    'layoutSplitInfo',
    JSON.stringify(layoutSplitInfo.value)
  );
};
</script>

<template>
  <div class="index">
    <el-tooltip
      :enterable="false"
      effect="dark"
      :content="$t('indexIOSTS.contentText')"
      placement="left"
      :offset="15"
    >
      <div
        :class="['button', tabPosition == 'left' ? 'button_top' : '']"
        @click="swithLayout"
      />
    </el-tooltip>
    <IOSRemote
      :tab-position="tabPosition"
      :canvas-rect-info="canvasRectInfo"
      :layout-split-info="layoutSplitInfo"
      :line-mouseup="lineMouseup"
      :line-mousemove="lineMousemove"
      :line-mousedown="lineMousedown"
      :line-mouseleave="lineMouseleave"
      :is-split-pressing="isPress"
    />
  </div>
</template>

<style scoped lang="less">
.index {
  /*background: #999;*/
  position: relative;

  .button {
    position: absolute;
    right: 10px;
    top: 0px;
    width: 30px;
    height: 30px;
    background: url('@/assets/img/left.png') no-repeat center;
    background-size: 70% 70%;
    cursor: pointer;

    &:focus {
      outline: 0;
    }

    &_top {
      background-image: url('@/assets/img/top.png');
    }
  }
}
</style>
