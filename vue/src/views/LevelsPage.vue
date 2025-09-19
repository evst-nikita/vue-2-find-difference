<template>
  <div class="levels-page">
    <h1 class="levels-page__title">Список уровней</h1>

    <div class="levels-page__empty" v-if="levels.length === 0">
      <p>
        Пока нет уровней. Зайдите в
        <router-link to="/admin">админку</router-link>
        и создайте первый.
      </p>
    </div>

    <div class="levels-page__list" v-else>
      <div
          v-for="level in levels"
          :key="level.id"
          class="levels-page__card"
      >
        <h3 class="levels-page__card-title">{{ level.name }}</h3>
        <img
            v-if="level.image1"
            :src="level.image1"
            alt="preview"
            class="levels-page__preview"
        >
        <div class="levels-page__actions">
          <router-link :to="`/play/${level.id}`" class="levels-page__link">
            Играть
          </router-link>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { getLevels } from '@/utils/storage'

export default {
  name: 'LevelsPage',
  data() {
    return {
      levels: []
    }
  },
  mounted() {
    this.levels = getLevels()
  }
}
</script>

<style lang="less" scoped>
.levels-page {
  padding: 16px;
}

.levels-page__title {
  margin: 0 0 12px;
}

.levels-page__empty {
  color: #555;
}

.levels-page__list {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
}

.levels-page__card {
  border: 1px solid #ccc;
  padding: 8px;
  width: 200px;
}

.levels-page__card-title {
  margin: 0 0 8px;
  font-size: 16px;
}

.levels-page__preview {
  max-width: 100%;
  height: auto;
  display: block;
}

.levels-page__actions {
  margin-top: 8px;
  display: flex;
  gap: 8px;
}

.levels-page__link {
  text-decoration: none;
}
</style>
