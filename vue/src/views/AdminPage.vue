<template>
  <div class="admin-page">
    <h1 class="admin-page__title">Создать уровень</h1>

    <form class="admin-page__form" @submit.prevent="onSave">
      <div class="admin-page__field">
        <label class="admin-page__label">Название уровня</label>
        <input
            class="admin-page__input"
            v-model="form.name"
            type="text"
            placeholder="Например, Level 1"
        >
      </div>

      <div class="admin-page__field admin-page__field--mt-12">
        <label class="admin-page__label">Картинка 1</label>
        <input
            class="admin-page__file"
            type="file"
            accept="image/*"
            @change="onFileChange($event, 'image1')"
        >
      </div>

      <div class="admin-page__field admin-page__field--mt-12">
        <label class="admin-page__label">Картинка 2</label>
        <input
            class="admin-page__file"
            type="file"
            accept="image/*"
            @change="onFileChange($event, 'image2')"
        >
      </div>

      <div class="admin-page__field admin-page__field--mt-16 admin-page__controls">
        <label class="admin-page__label">Радиус круга (px, в координатах исходника):</label>
        <input
            class="admin-page__number"
            type="number"
            v-model.number="radius"
            min="5"
            max="200"
        >
        <label class="admin-page__radio">
          <input type="radio" value="add" v-model="mode"> Добавить
        </label>
        <label class="admin-page__radio">
          <input type="radio" value="remove" v-model="mode"> Удалить
        </label>
      </div>

      <div class="admin-page__editors">
        <div class="admin-page__editor" v-if="form.image1">
          <img
              ref="img"
              :src="form.image1"
              alt="img1"
              @load="onImgLoad"
          >
          <svg
              v-if="imgBox.width && imgBox.height"
              class="admin-page__overlay"
              :viewBox="`0 0 ${imgBox.width} ${imgBox.height}`"
              :width="imgBox.width"
              :height="imgBox.height"
              @click="onOverlayClick"
          >
            <circle
                v-for="(s, i) in form.spots"
                :key="i"
                :cx="toScreenX(s.x)"
                :cy="toScreenY(s.y)"
                :r="toScreenR(s.r)"
                fill="rgba(0,0,0,0.0)"
                stroke="red"
                stroke-width="2"
            />
          </svg>
        </div>

        <div class="admin-page__editor admin-page__editor--secondary" v-if="form.image2">
          <img :src="form.image2" alt="img2">
        </div>
      </div>

      <div v-if="form.spots.length" class="admin-page__field admin-page__field--mt-12">
        <strong>Отмечено отличий: {{ form.spots.length }}</strong>
        <ul class="admin-page__list">
          <li class="admin-page__list-item" v-for="(s, i) in form.spots" :key="i">
            #{{ i + 1 }} (x: {{ s.x }}, y: {{ s.y }}, r: {{ s.r }})
            <button type="button" class="admin-page__btn-delete" @click="removeSpot(i)">Удалить</button>
          </li>
        </ul>
      </div>

      <button type="submit" class="admin-page__btn-submit">Сохранить уровень</button>
    </form>

    <p v-if="message" class="admin-page__message admin-page__message--success">{{ message }}</p>
    <p v-if="error" class="admin-page__message admin-page__message--error">{{ error }}</p>
  </div>
</template>

<script>
import { saveLevel } from '@/utils/storage'
import { uid } from '@/utils/id'

export default {
  name: 'AdminPage',
  data() {
    return {
      form: {
        id: uid(),
        name: '',
        image1: null,
        image2: null,
        spots: []
      },
      message: '',
      error: '',
      radius: 24,
      mode: 'add', // 'add' | 'remove'
      imgNatural: { w: 0, h: 0 },
      imgBox: { width: 0, height: 0 }
    }
  },
  methods: {
    onFileChange(e, field) {
      this.error = ''
      const file = e.target.files && e.target.files[0]
      if (!file) return

      const MAX_MB = 5
      if (file.size > MAX_MB * 1024 * 1024) {
        this.error = `Файл слишком большой (> ${MAX_MB} MB)`
        e.target.value = ''
        return
      }

      const reader = new FileReader()
      reader.onload = () => {
        this.form[field] = reader.result
      }
      reader.onerror = () => {
        this.error = 'Не удалось прочитать файл'
      }
      reader.readAsDataURL(file)
    },
    async onSave() {
      this.message = ''
      this.error = ''

      if (!this.form.name.trim()) {
        this.error = 'Введите название уровня'
        return
      }
      if (!this.form.image1 || !this.form.image2) {
        this.error = 'Нужно загрузить обе картинки'
        return
      }

      try {
        saveLevel(this.form)

        this.message = 'Уровень сохранён'
        const oldName = this.form.name
        this.form = {
          id: uid(),
          name: oldName + ' (копия)',
          image1: null,
          image2: null,
          spots: []
        }
      } catch (e) {
        this.error = 'Ошибка при сохранении уровня'
        console.error(e)
      }
    },
    onImgLoad() {
      const imgEl = this.$refs.img
      this.imgNatural.w = imgEl.naturalWidth
      this.imgNatural.h = imgEl.naturalHeight
      this.imgBox.width = imgEl.clientWidth
      this.imgBox.height = imgEl.clientHeight
    },
    toImageCoords(screenX, screenY) {
      const scaleX = this.imgNatural.w / this.imgBox.width
      const scaleY = this.imgNatural.h / this.imgBox.height
      return {
        x: Math.round(screenX * scaleX),
        y: Math.round(screenY * scaleY)
      }
    },
    toScreenX(imgX) {
      const scaleX = this.imgBox.width / this.imgNatural.w
      return imgX * scaleX
    },
    toScreenY(imgY) {
      const scaleY = this.imgBox.height / this.imgNatural.h
      return imgY * scaleY
    },
    toScreenR(imgR) {
      const scale = this.imgBox.width / this.imgNatural.w
      return imgR * scale
    },
    onOverlayClick(e) {
      const rect = e.currentTarget.getBoundingClientRect()
      const sx = e.clientX - rect.left
      const sy = e.clientY - rect.top

      if (this.mode === 'add') {
        const { x, y } = this.toImageCoords(sx, sy)
        this.form.spots.push({ x, y, r: this.radius })
      } else {
        const { x, y } = this.toImageCoords(sx, sy)
        const idx = this.findSpotIndexUnderPoint(x, y)
        if (idx !== -1) this.form.spots.splice(idx, 1)
      }
    },
    findSpotIndexUnderPoint(x, y) {
      let best = -1
      let bestDist = Infinity
      for (let i = 0; i < this.form.spots.length; i++) {
        const s = this.form.spots[i]
        const dx = s.x - x
        const dy = s.y - y
        const dist = Math.sqrt(dx * dx + dy * dy)
        if (dist <= s.r && dist < bestDist) {
          best = i
          bestDist = dist
        }
      }
      return best
    },
    removeSpot(i) {
      this.form.spots.splice(i, 1)
    }
  },
}
</script>

<style lang="less" scoped>
.admin-page {
  max-width: 900px;
  margin: 0 auto;
  padding: 16px;
}

.admin-page__title {
  margin: 0 0 12px;
}

.admin-page__form {
  display: block;
}

.admin-page__field {
  display: flex;
  flex-direction: column;
}

.admin-page__field--mt-12 {
  margin-top: 12px;
}

.admin-page__field--mt-16 {
  margin-top: 16px;
}

.admin-page__label {
  margin-bottom: 6px;
}

.admin-page__input,
.admin-page__file,
.admin-page__number {
  width: 240px;
}

.admin-page__controls {
  align-items: center;
}

.admin-page__radio {
  margin-left: 12px;
}

.admin-page__editors {
  display: flex;
  gap: 20px;
  margin-top: 12px;
}

.admin-page__editor {
  position: relative;
  display: inline-block;
  border: 1px solid #ccc;
}

.admin-page__editor img {
  display: block;
  max-width: 100%;
  height: auto;
}

.admin-page__editor--secondary {
  margin-left: 20px;
}

.admin-page__overlay {
  position: absolute;
  left: 0;
  top: 0;
  pointer-events: auto;
  cursor: crosshair;
}

.admin-page__list {
  margin: 8px 0 0;
  padding-left: 18px;
}

.admin-page__list-item {
  margin-top: 4px;
}

.admin-page__btn-submit {
  margin-top: 16px;
}

.admin-page__btn-delete {
  margin-left: 8px;
}

.admin-page__message {
  margin-top: 10px;
}

.admin-page__message--success {
  color: green;
}

.admin-page__message--error {
  color: crimson;
}
</style>
