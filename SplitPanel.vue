<script setup lang="ts">

import type {PropType} from 'vue'
import {ref} from "vue";

type Modes = "horizontal" | "vertical"

const props = defineProps({
  mode: {
    type: String as PropType<Modes>,
    default: "horizontal",
    required: false
  },
  triggerSize: {
    type: String,
    default: "5px",
    required: false,
    validator: (value: string) => value && value.indexOf("px") > -1
  },
  ratio: {
    type: Number,
    default: 0.5,
    validator: (value: number) => value > 0 && value < 1
  },
  max: {
    type: Number,
    default: 400 // mode : vertical 时使用
  }
})


/**
 *
 * @param ratio
 */
function initSize(ratio: number): string {
  if (props.mode === 'horizontal') return `calc( (100% - ${props.triggerSize}) * ${ratio} )`;
  return `calc( (${props.max}px - ${props.triggerSize}) * ${ratio} )`;
}

function getNumberValue(value: string): number {
  if (!value) return 0;
  const match = value.match(/\d+/);
  if (!match || !match.length) return 0;
  return Number(match[0]);
}

const startSize = ref<string>(initSize(props.ratio))
const endSize = ref<string>(initSize(1 - props.ratio))
const triggerValue = ref<number>(getNumberValue(props.triggerSize))

const splitPanelRef = ref<HTMLElement>()
const triggerRef = ref<HTMLElement>()
const startPanelRef = ref<HTMLElement>()
const endPanelRef = ref<HTMLElement>()

const oldX = ref<number>(0)
const oldY = ref<number>(0)
const isActive = ref<boolean>(false)

interface StartEndSize {
  start: string
  end: string
}

function computeStartEndSize(key: "width" | "height", diff: number): StartEndSize {
  const mainRect = splitPanelRef.value?.getBoundingClientRect();
  const startRect = startPanelRef.value?.getBoundingClientRect();

  const maxV = mainRect[key];

  let start = Math.round(startRect[key]);
  let end = maxV - start - triggerValue.value;

  start += diff;
  end -= diff;
  if (start <= 0) {
    start = 0;
    end = maxV - triggerValue.value;
  }
  if (end <= 0) {
    end = 0;
    start = maxV - triggerValue.value;
  }
  return {end: end + "px", start: start + "px"}
}

function handleHorizontalMove(diff: number) {
  const size = computeStartEndSize("width", diff);
  startSize.value = size.start;
  endSize.value = size.end;
}

function handleVerticalMove(diff: number) {
  const size = computeStartEndSize("height", diff);
  startSize.value = size.start;
  endSize.value = size.end;
}

function handleStart(event: MouseEvent) {
  isActive.value = true
  oldX.value = event.screenX;
  oldY.value = event.screenY;
  triggerRef.value?.addEventListener("mousemove", handleMove)
  triggerRef.value?.addEventListener("mouseup", handleEnd)
  triggerRef.value?.addEventListener("mouseout", handleEnd)
}

function handleMove(event: MouseEvent) {
  const diffX = event.screenX - oldX.value;
  const diffY = event.screenY - oldY.value;

  oldX.value = event.screenX;
  oldY.value = event.screenY;

  if (props.mode === 'horizontal') handleHorizontalMove(diffX);
  if (props.mode === 'vertical') handleVerticalMove(diffY);
}

function handleEnd() {
  triggerRef.value?.removeEventListener("mouseup", handleEnd);
  triggerRef.value?.removeEventListener("mousemove", handleMove);
  triggerRef.value?.removeEventListener("mouseout", handleEnd);

  isActive.value = false
  oldX.value = 0;
  oldY.value = 0;
}

</script>

<template>
  <div ref="splitPanelRef" class="split-panel" :class="{ ['split-panel-'+mode]:mode }">
    <div ref="startPanelRef" class="panel-start">
      <slot name="start">panel-start</slot>
    </div>
    <div ref="triggerRef" class="split-trigger" :class="{ 'split-trigger-active':isActive }"
         @mousedown.stop="handleStart">
      <slot name="trigger"/>
    </div>
    <div ref="endPanelRef" class="panel-end">
      <slot name="end">panel-end</slot>
    </div>
  </div>
</template>

<style scoped lang="scss">
.split-panel {

  .panel-start {
    background-color: purple;
  }

  .panel-end {
    background-color: firebrick;
  }

  .split-trigger {
    background-color: green;
    overflow: hidden;
    position: relative;
    background-color: rgba(255, 255, 255, 0.07);

    &.split-trigger-active::after {
      content: "";
      position: fixed;
      z-index: 1000;
      top: 0;
      left: 0;
      width: 100vw;
      height: 100vh;
      background-color: rgba(0, 0, 0, 0);
    }

    &:hover::before {
      content: "";
      display: block;
      background-color: deeppink;
    }
  }

  // 水平分割面板
  &.split-panel-horizontal {
    width: 100%;
    display: flex;
    flex-direction: row;

    .panel-start,
    .panel-end {
      overflow-y: hidden;
    }

    .panel-start {
      width: v-bind(startSize);
    }

    .panel-end {
      width: v-bind(endSize);
    }

    .split-trigger {
      width: v-bind(triggerSize);

      &:hover {
        background-color: rgba(46, 189, 0, 0.2);
        cursor: col-resize;

        &::before {
          width: 0;
          height: 100%;
          margin: 0 auto;
          border-right: 1px #CCCCCC dashed;
        }
      }
    }
  }

  // 垂直分割面板
  &.split-panel-vertical {

    .panel-start,
    .panel-end {
      overflow-y: auto;
    }

    .panel-start {
      height: v-bind(startSize);
    }

    .panel-end {
      height: v-bind(endSize);
    }

    .split-trigger {
      height: v-bind(triggerSize);

      &:hover {
        background-color: rgba(46, 189, 0, 0.2);
        cursor: row-resize;
        display: grid;
        align-content: center;

        &::before {
          width: 100%;
          height: 0;
          border-bottom: 1px #CCCCCC dashed;
        }
      }
    }
  }
}
</style>