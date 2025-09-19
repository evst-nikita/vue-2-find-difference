<template>
  <div class="play-page">
    <div v-if="!level" class="play-page__empty">
      <h1 class="play-page__title">Уровень не найден</h1>
      <p class="play-page__text">Проверьте ссылку или вернитесь к списку уровней.</p>
    </div>

    <div v-else>
      <h1 class="play-page__title">{{ level.name }}</h1>

      <div class="play-page__game">
        <div class="play-page__image-wrap">
          <img
              ref="img"
              :src="level.image1"
              alt="img1"
              @load="onImgLoad"
          >
          <svg
              v-if="imgBox.width && imgBox.height"
              class="play-page__overlay"
              :viewBox="`0 0 ${imgBox.width} ${imgBox.height}`"
              :width="imgBox.width"
              :height="imgBox.height"
              @click="onClick"
          >
            <circle
                v-for="(s, i) in found"
                :key="i"
                :cx="toScreenX(s.x)"
                :cy="toScreenY(s.y)"
                :r="toScreenR(s.r)"
                fill="rgba(0,255,0,0.3)"
                stroke="green"
                stroke-width="2"
            />
          </svg>
        </div>

        <div class="play-page__image-wrap">
          <img :src="level.image2" alt="img2">
          <svg
              v-if="imgBox.width && imgBox.height"
              class="play-page__overlay"
              :viewBox="`0 0 ${imgBox.width} ${imgBox.height}`"
              :width="imgBox.width"
              :height="imgBox.height"
          >
            <circle
                v-for="(s, i) in found"
                :key="'right-' + i"
                :cx="toScreenX(s.x)"
                :cy="toScreenY(s.y)"
                :r="toScreenR(s.r)"
                fill="rgba(0,255,0,0.3)"
                stroke="green"
                stroke-width="2"
            />
          </svg>
        </div>
      </div>

      <div v-if="win" class="play-page__win">
        🎉 Победа! Все отличия найдены!
      </div>
    </div>
  </div>
</template>

<script>
import { getLevel } from '@/utils/storage'

export default {
  name: 'PlayPage',
  props: {
    levelId: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      level: null,
      imgNatural: { w: 0, h: 0 },
      imgBox: { width: 0, height: 0 },
      found: [],
      win: false
    }
  },
  methods: {
    onImgLoad() {
      const imgEl = this.$refs.img
      this.imgNatural.w = imgEl.naturalWidth
      this.imgNatural.h = imgEl.naturalHeight
      this.imgBox.width = imgEl.clientWidth
      this.imgBox.height = imgEl.clientHeight
    },
    toImageCoords(sx, sy) {
      return {
        x: Math.round(sx * (this.imgNatural.w / this.imgBox.width)),
        y: Math.round(sy * (this.imgNatural.h / this.imgBox.height))
      }
    },
    toScreenX(imgX) {
      return imgX * (this.imgBox.width / this.imgNatural.w)
    },
    toScreenY(imgY) {
      return imgY * (this.imgBox.height / this.imgNatural.h)
    },
    toScreenR(imgR) {
      return imgR * (this.imgBox.width / this.imgNatural.w)
    },
    onClick(e) {
      const rect = e.currentTarget.getBoundingClientRect()
      const sx = e.clientX - rect.left
      const sy = e.clientY - rect.top
      const { x, y } = this.toImageCoords(sx, sy)

      const idx = this.level.spots.findIndex(s => {
        const dx = s.x - x
        const dy = s.y - y
        return Math.sqrt(dx * dx + dy * dy) <= s.r
      })

      if (idx !== -1) {
        const spot = this.level.spots[idx]
        const alreadyFound = this.found.some(f => f.x === spot.x && f.y === spot.y)
        if (!alreadyFound) {
          this.found.push(spot)
          if (this.found.length === this.level.spots.length) {
            this.win = true
          }
        }
      }
    }
  },
  mounted() {
    this.level = getLevel(this.levelId)
  }
}
</script>

<style lang="less" scoped>
.play-page {
  padding: 16px;
}

.play-page__title {
  margin: 0 0 12px;
}

.play-page__empty .play-page__title {
  margin-bottom: 8px;
}

.play-page__text {
  margin: 0;
}

.play-page__game {
  display: flex;
  gap: 20px;
  margin-top: 20px;
}

.play-page__image-wrap {
  position: relative
}

.play-page__image-wrap img {
  display: block;
  max-width: 100%;
  height: auto;
}

.play-page__overlay {
  position: absolute;
  left: 0;
  top: 0;
  pointer-events: auto;
  cursor: crosshair;
}

.play-page__win {
  margin-top: 20px;
  font-size: 1.5em;
  color: green;
  font-weight: bold;
}
</style>
