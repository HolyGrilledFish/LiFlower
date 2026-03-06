<!--
  模块编号：M6
  模块名称：专长（人类）
  显示模式：human-prep
  功能：2-6个专长槽，大脑电子化按钮
-->
<template>
  <div class="module-human-talents">
    <div class="header">
      <h3 class="module-title">专长</h3>
      <button
        :class="['cyber-button', { active: isCyberBrainActive }]"
        @click="toggleCyberBrain"
      >
        <span class="button-icon">{{ isCyberBrainActive ? '◉' : '○' }}</span>
        <span class="button-text">大脑电子化</span>
      </button>
    </div>

    <!-- 专长列表 -->
    <div class="talents-list">
      <TalentItem
        v-for="(talent, index) in talents"
        :key="index"
        v-model="talents[index]"
        :index="index"
        :show-delete-button="talents.length > 2"
        @delete="removeTalent(index)"
        @clear="clearTalent(index)"
      />
    </div>

    <!-- 新增专长按钮（在第二个专长下面，最多6个） -->
    <div v-if="showAddButton" class="add-button-row">
      <button class="add-talent-btn" @click="addTalent">
        <span class="add-icon">+</span>
        <span class="add-text">新增专长</span>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { useAutoOutput } from '@/composables/useModuleOutput'
import TalentItem from '@/components/TalentItem.vue'

// 大脑电子化状态
const isCyberBrainActive = ref(false)

// 切换状态
const toggleCyberBrain = () => {
  isCyberBrainActive.value = !isCyberBrainActive.value
}

// 创建空专长模板
const createEmptyTalent = () => ({
  name: '',
  skill: '',
  limitation: '',
  description: ''
})

// 专长数据（默认2个）
const talents = ref([
  createEmptyTalent(),
  createEmptyTalent()
])

// 是否显示新增按钮（最多6个）
const showAddButton = computed(() => talents.value.length < 6)

// 添加专长（在第二个位置后添加）
const addTalent = () => {
  if (talents.value.length < 6) {
    talents.value.push(createEmptyTalent())
  }
}

// 清空专长内容
const clearTalent = (index) => {
  talents.value[index] = createEmptyTalent()
}

// 移除专长（补位机制：后面的前移）
const removeTalent = (index) => {
  // 最少保留2个专长
  if (talents.value.length <= 2) {
    return
  }
  
  // 移除指定索引的专长
  talents.value.splice(index, 1)
}

// 构建输出数据
const talentsOutput = computed(() => {
  return talents.value.map((talent, index) => ({
    index: index + 1,
    name: talent.name,
    skill: talent.skill,
    skillName: getSkillName(talent.skill),
    limitation: talent.limitation,
    description: talent.description,
    isActive: !!talent.name
  })).filter(t => t.isActive)
})

// 获取技能名称
const getSkillName = (skillId) => {
  if (!skillId) return ''
  if (skillId === 'special') return '特殊对策'
  return skillId
}

// 数据输出
useAutoOutput({
  isCyberBrainActive,
  cyberBrainStatus: computed(() => isCyberBrainActive.value ? 'activated' : 'inactive'),
  talents: talentsOutput,
  activeTalentCount: computed(() => talentsOutput.value.length),
  totalTalentSlots: computed(() => talents.value.length)
})
</script>

<style lang="scss" scoped>
$cyber-cyan: #00f3ff;
$cyber-purple: #bc13fe;

.module-human-talents {
  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 16px;
  }

  .module-title {
    color: $cyber-cyan;
    margin: 0;
    font-size: 16px;
  }

  // 赛博朋克风格按钮
  .cyber-button {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 10px 16px;
    background: rgba(26, 26, 46, 0.8);
    border: 1px solid rgba(0, 243, 255, 0.3);
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.3s ease;
    font-family: "Courier New", "Consolas", monospace;

    .button-icon {
      font-size: 14px;
      color: rgba(255, 255, 255, 0.4);
      transition: all 0.3s ease;
    }

    .button-text {
      font-size: 13px;
      color: rgba(255, 255, 255, 0.7);
      transition: all 0.3s ease;
    }

    &:hover {
      background: rgba(0, 243, 255, 0.1);
      border-color: rgba(0, 243, 255, 0.5);
      box-shadow: 0 0 10px rgba(0, 243, 255, 0.2);

      .button-icon {
        color: rgba(0, 243, 255, 0.6);
      }

      .button-text {
        color: rgba(255, 255, 255, 0.9);
      }
    }

    &.active {
      background: linear-gradient(
        135deg,
        rgba(0, 243, 255, 0.2),
        rgba(188, 19, 254, 0.2)
      );
      border-color: $cyber-cyan;
      box-shadow: 0 0 15px rgba(0, 243, 255, 0.3), inset 0 0 10px rgba(0, 243, 255, 0.1);

      .button-icon {
        color: $cyber-cyan;
        text-shadow: 0 0 8px rgba(0, 243, 255, 0.8);
      }

      .button-text {
        color: $cyber-cyan;
        font-weight: 600;
      }

      &:active {
        transform: scale(0.98);
        box-shadow: 0 0 20px rgba(0, 243, 255, 0.4), inset 0 0 15px rgba(0, 243, 255, 0.2);
      }
    }
  }

  .talents-list {
    margin-top: 16px;
  }

  // 新增专长按钮
  .add-button-row {
    margin-top: 16px;
    padding-top: 8px;
    border-top: 1px dashed rgba(0, 243, 255, 0.2);
    display: flex;
    justify-content: center;
  }

  .add-talent-btn {
    display: flex;
    align-items: center;
    gap: 6px;
    padding: 8px 16px;
    background: rgba(0, 243, 255, 0.05);
    border: 1px dashed rgba(0, 243, 255, 0.4);
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.3s ease;
    font-family: "Courier New", "Consolas", monospace;

    .add-icon {
      color: $cyber-cyan;
      font-size: 16px;
      font-weight: bold;
    }

    .add-text {
      color: rgba(255, 255, 255, 0.7);
      font-size: 13px;
    }

    &:hover {
      background: rgba(0, 243, 255, 0.1);
      border-color: $cyber-cyan;
      box-shadow: 0 0 10px rgba(0, 243, 255, 0.2);

      .add-text {
        color: rgba(255, 255, 255, 0.9);
      }
    }

    &:active {
      transform: scale(0.98);
    }
  }
}
</style>
