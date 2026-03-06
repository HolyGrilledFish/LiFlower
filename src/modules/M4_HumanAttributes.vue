<!--
  模块编号：M4
  模块名称：属性（人类）
  显示模式：human-prep
  功能：6属性点分配（人类）- 当可用属性点>0时自动显示
-->
<template>
  <div class="module-human-attributes">
    <div class="module-header">
      <h3 class="module-title">属性点分配</h3>
      <span class="points-info">
        可用属性点：<span class="points-highlight">{{ remainingPoints }}</span> / {{ totalAttributePoints }}
      </span>
    </div>
    <!-- 属性点分配器 -->
    <AttributeAllocator
      :attribute-points="totalAttributePoints"
      :attribute-limit="attributeLimit"
      :attributes="characterStore.attributes"
      :attribute-data="attributeData"
      :show-divider="false"
      :modifier-rules="modifierRules"
      @update:attributes="handleAttributesUpdate"
    />
  </div>
</template>

<script setup>
import { computed, watch } from 'vue'
import { useCharacterStore } from '@/stores/character'
import { useModuleOutputsStore } from '@/stores/moduleOutputs'
import { useAutoOutput } from '@/composables/useModuleOutput'
import AttributeAllocator from '@/components/AttributeAllocator.vue'

// 导入属性数据
import attributeDataJson from '@/data/基本属性.json'
import backgroundData from '@/data/人类起源.json'

// 使用 store
const characterStore = useCharacterStore()
const outputsStore = useModuleOutputsStore()

// 属性数据
const attributeData = attributeDataJson.attributeSystem

// 人类属性点限制
const attributeLimit = 5

// 总技能点数
const totalSkillPoints = 15

// 计算已用技能点
const usedSkillPoints = computed(() => {
  return Object.values(characterStore.skills).reduce((sum, val) => sum + val, 0)
})

// 计算剩余可用技能点
const remainingSkillPoints = computed(() => {
  return totalSkillPoints - usedSkillPoints.value
})

// ==================== 出身影响计算 ====================
// 获取当前出身数据
const currentBackground = computed(() => {
  const bgId = characterStore.humanBackground
  if (!bgId) return null
  return backgroundData.find(item => item.id.toString() === bgId.toString())
})

// 是否为"无意义一代"
const isSuperfluousGeneration = computed(() => {
  return currentBackground.value?.id === 1
})

// ==================== 从 M1 读取特质（检测忒修斯之船）====================
const m1Traits = computed(() => {
  return outputsStore.getModuleOutput('M1').traits || ''
})

// 是否为"忒修斯之船"特质（ID=29）- 最高优先级
const isTheseusShip = computed(() => {
  return m1Traits.value.includes('29')
})

// ==================== 从 M5 读取数据 ====================
const m5Data = computed(() => outputsStore.getModuleOutput('M5'))

// 人性当前值（ID=16）
const humanityValue = computed(() => m5Data.value.humanityValue || 0)

// 人性负值转化的额外点数
const humanityBonus = computed(() => {
  const val = humanityValue.value
  return val < 0 ? Math.abs(val) : 0
})

// ==================== 可用属性点计算 ====================
// 规则优先级：
// 1. 忒修斯之船（ID=29）：可用属性点 = 15 + 人性负值（每-1人性=+1属性）
// 2. 非"无意义一代"：可用属性点 = 0
// 3. "无意义一代"：可用属性点 = min(5, M5剩余技能点)

const totalAttributePoints = computed(() => {
  // 优先级1：忒修斯之船
  // 基础15点 + 人性负值转化（每-1人性=+1属性点）
  if (isTheseusShip.value) {
    return 15 + humanityBonus.value
  }

  // 优先级2：非无意义一代
  if (!isSuperfluousGeneration.value) {
    return 0
  }

  // 优先级3：无意义一代规则
  const m5Used = m5Data.value.usedSkillPoints || 0
  const m5Remaining = m5Data.value.remainingSkillPoints ?? 15
  const maxAllowedByTotal = 15 - m5Used
  const baseAvailable = Math.min(5, m5Remaining)
  return Math.min(baseAvailable, maxAllowedByTotal)
})

// 计算已分配属性点
const allocatedPoints = computed(() => {
  return Object.values(characterStore.attributes).reduce((sum, val) => sum + val, 0)
})

// 计算剩余可用属性点
const remainingPoints = computed(() => {
  return totalAttributePoints.value - allocatedPoints.value
})

// 总消耗检查：已分配属性点超过可用上限时，清空属性点
watch([allocatedPoints, totalAttributePoints], ([allocated, available]) => {
  if (allocated > available) {
    // 清空属性点选择
    characterStore.updateAttributes({
      structure: 0,
      strength: 0,
      athletics: 0,
      compute: 0,
      information: 0,
      power: 0
    })
  }
})

// 处理属性更新
const handleAttributesUpdate = (newAttrs) => {
  const newAttrPoints = Object.values(newAttrs).reduce((sum, val) => sum + val, 0)

  // 检查是否超过当前可用上限
  if (newAttrPoints > totalAttributePoints.value) {
    return
  }

  characterStore.updateAttributes(newAttrs)
}

// 调整值规则（人类暂无）
const modifierRules = []

// ==================== 模块显示控制 ====================
// 定义模块的显示条件：当可用属性点 > 0 时显示
const shouldShow = computed(() => {
  return totalAttributePoints.value > 0
})

// 暴露给父组件（StandardModule）
defineExpose({
  shouldShow
})

// ==================== 模块数据输出 ====================
useAutoOutput({
  totalAttributePoints,
  remainingPoints,
  humanityValue,
  humanityBonus,
  isTheseusShip,
  isSuperfluousGeneration,
  shouldShow
})
</script>

<style lang="scss" scoped>
$cyber-cyan: #00f3ff;

.module-human-attributes {
  width: 100%;

  .module-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 16px;
    border-bottom: 1px solid rgba(0, 243, 255, 0.2);
    padding-bottom: 8px;
  }

  .module-title {
    color: $cyber-cyan;
    margin: 0;
    font-size: 16px;
  }

  .points-info {
    color: rgba(255, 255, 255, 0.8);
    font-size: 13px;
    font-family: "Courier New", "Consolas", monospace;
  }

  .points-highlight {
    color: $cyber-cyan;
    font-weight: 700;
  }

  .humanity-bonus {
    color: #bc13fe;
    font-size: 11px;
    margin-left: 8px;
  }
}
</style>
