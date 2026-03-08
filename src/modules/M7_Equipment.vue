<!--
  模块编号：M7
  模块名称：芯片（人形）
  显示模式：doll-prep
  功能：芯片槽位管理（支持专利芯片与协调芯片规则）+ 技能栏位
-->
<template>
  <div class="module-equipment">
    <ModuleHeader title="芯片" subtitle="Chips">
      <template #right>
        <span class="slot-info">
          芯片槽位：<span class="slot-count">{{ usedSlots }}/{{ maxSlots }}</span>
          <span v-if="patentCount > 0" class="patent-info">
            (专利{{ patentCount }}个<span v-if="coordinationCount > 0">，协调{{ coordinationCount }}个</span>)
          </span>
        </span>
      </template>
    </ModuleHeader>

    <!-- 芯片槽位列表 -->
    <div class="chip-slots-list">
      <ChipSlot
        v-for="(slot, index) in chipSlots"
        :key="index"
        v-model="slot.type"
        v-model:patent-chip-id="slot.patentChipId"
        v-model:chip-config="slot.config"
        :index="index"
        :available-options="getAvailableOptions(index)"
        :manufacturer-id="selectedManufacturerId"
        :selected-patent-chip-ids="otherPatentChipIds(index)"
        @update:modelValue="handleSlotChange(index, $event)"
      />
    </div>

    <!-- 规则提示 -->
    <div class="rules-hint">
      <p v-if="isPatentDisabled" class="patent-disabled-warning">
        ⚠ {{ isBlackMarket ? '黑品人形无法使用专利芯片' : '越狱刷机人形无法使用专利芯片' }}
      </p>
      <p v-else>规则：每2个专利芯片之间需要1个协调芯片</p>
      <p v-if="!isPatentDisabled">1专利=1槽 | 2专利+1协调=3槽 | 3专利+3协调=6槽</p>
    </div>

    <!-- 技能栏位 -->
    <div class="skills-section">
      <AttributeAllocator
        :attribute-points="999"
        :attribute-limit="5"
        :attributes="skillValues"
        :attribute-data="skillData"
        :show-divider="false"
        :modifier-rules="modifierRules"
        :special-ranges="lockedSkillRanges"
        :hide-zero="true"
        @update:attributes="updateSkillValues"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { useModuleOutputsStore } from '@/stores/moduleOutputs'
import { useAutoOutput } from '@/composables/useModuleOutput'
import ChipSlot from '@/components/ChipSlot.vue'
import AttributeAllocator from '@/components/AttributeAllocator.vue'
import ModuleHeader from '@/components/ModuleHeader.vue'
import patentChipsData from '@/data/patentChips.json'
import skillsData from '@/data/skills.json'

const outputsStore = useModuleOutputsStore()

// ==================== 芯片部分 ====================

// 读取M2输出
const m2Output = computed(() => outputsStore.getModuleOutput('M2') || {})

// 读取M3输出（用于检查剩余属性点）
const m3Output = computed(() => outputsStore.getModuleOutput('M3') || {})

// 读取M9输出（用于检查尖端武装词条）
const m9Output = computed(() => outputsStore.getModuleOutput('M9') || {})

// 计算动态槽位上限
const maxSlots = computed(() => {
  // 获取特质ID列表
  const traitIds = m2Output.value?.traitIds || []
  
  // 最高优先级：复古智能（ID=17）→ 槽位为0
  if (traitIds.includes('17') || traitIds.includes(17)) {
    return 0
  }
  
  // 基础槽位
  let slots = 6
  
  // 过度拟合（ID=1）→ 槽位+1
  if (traitIds.includes('1') || traitIds.includes(1)) {
    slots += 1
  }
  
  // 检查尖端武装是否包含"集成"词条（ID=1）
  const weaponTags = m9Output.value?.weapon?.tags || []
  const hasIntegrated = weaponTags.some(tag => tag === 1 || tag === '1')
  
  if (hasIntegrated) {
    // 读取M3剩余属性点
    const remainingPoints = m3Output.value?.remainingPoints || 0
    // 如果剩余属性点小于2，槽位-2
    if (remainingPoints < 2) {
      slots -= 2
    }
  }
  
  // 确保槽位不小于0
  return Math.max(0, slots)
})

// 芯片槽位数据（包含类型、专利芯片ID和配置）
const chipSlots = ref(
  Array(6).fill(null).map(() => ({
    type: 'none',
    patentChipId: null,
    config: {
      normalSubtype: null,
      skillId: null,
      skillUpgraded: false,
      programName: '',
      programUpgraded: false,
      upgradeEffect: ''
    }
  }))
)

// 监听槽位上限变化，调整槽位数组
watch(maxSlots, (newMax, oldMax) => {
  if (newMax < oldMax) {
    // 槽位减少时，截断多余的槽位
    chipSlots.value = chipSlots.value.slice(0, newMax)
  } else if (newMax > oldMax) {
    // 槽位增加时，添加新的空槽位
    const slotsToAdd = newMax - oldMax
    for (let i = 0; i < slotsToAdd; i++) {
      chipSlots.value.push({
        type: 'none',
        patentChipId: null,
        config: {
          normalSubtype: null,
          skillId: null,
          skillUpgraded: false,
          programName: '',
          programUpgraded: false,
          upgradeEffect: ''
        }
      })
    }
  }
}, { immediate: true })

// 读取M2选择的人形企业ID
const selectedManufacturerId = computed(() => {
  return m2Output.value?.manufacturerId || null
})

// 获取特质ID列表
const traitIds = computed(() => {
  return m2Output.value?.traitIds || []
})

// 是否是黑品（ID=8）
const isBlackMarket = computed(() => {
  return selectedManufacturerId.value === 8
})

// 是否有越狱刷机特质（ID=10）
const hasJailbreak = computed(() => {
  return traitIds.value.includes('10') || traitIds.value.includes(10)
})

// 是否禁用专利芯片（黑品或越狱刷机）
const isPatentDisabled = computed(() => {
  return isBlackMarket.value || hasJailbreak.value
})

// 统计专利芯片数量
const patentCount = computed(() => {
  return chipSlots.value.filter(slot => slot.type === 'patent').length
})

// 统计协调芯片数量
const coordinationCount = computed(() => {
  return chipSlots.value.filter(slot => slot.type === 'coordination').length
})

// 统计常规芯片数量
const normalCount = computed(() => {
  return chipSlots.value.filter(slot => slot.type === 'normal').length
})

// 计算实际占用的槽位
const usedSlots = computed(() => {
  return chipSlots.value.filter(slot => slot.type !== 'none').length
})

// 检查是否可以添加专利芯片
const canAddPatent = computed(() => {
  // 黑品或越狱刷机：禁用专利芯片
  if (isPatentDisabled.value) return false
  
  const p = patentCount.value
  const c = coordinationCount.value

  if (p === 0) return true
  if (p === 1 && c >= 1) return true
  if (p === 2 && c >= 3) return true
  return false
})

// 获取其他槽位已选择的专利芯片ID（用于去重）
const otherPatentChipIds = (currentIndex) => {
  return chipSlots.value
    .filter((slot, index) => index !== currentIndex && slot.type === 'patent' && slot.patentChipId)
    .map(slot => slot.patentChipId)
}

// 获取某个槽位可用的选项
const getAvailableOptions = (index) => {
  const currentValue = chipSlots.value[index].type

  const options = [
    { label: '未安装', value: 'none', disabled: false },
    { label: '常规芯片', value: 'normal', disabled: false },
    { label: '协调芯片', value: 'coordination', disabled: false }
  ]

  const isPatentAvailable = canAddPatent.value || currentValue === 'patent'
  options.push({
    label: '专利芯片',
    value: 'patent',
    disabled: !isPatentAvailable
  })

  return options
}

// 处理槽位变更
const handleSlotChange = (index, newValue) => {
  const oldValue = chipSlots.value[index].type

  if (newValue === 'patent' && oldValue !== 'patent') {
    if (!canAddPatent.value) {
      return
    }
  }

  chipSlots.value[index].type = newValue

  if (oldValue === 'coordination' && newValue !== 'coordination') {
    validateAndFixPatentChips()
  }
}

// 验证并修复专利芯片配置
const validateAndFixPatentChips = () => {
  const p = patentCount.value
  const c = coordinationCount.value

  let isValid = true
  if (p === 0) {
    isValid = true
  } else if (p === 1) {
    isValid = true
  } else if (p === 2) {
    isValid = c >= 1
  } else if (p >= 3) {
    isValid = c >= 3
  }

  if (!isValid) {
    for (let i = chipSlots.value.length - 1; i >= 0; i--) {
      if (chipSlots.value[i].type === 'patent') {
        chipSlots.value[i].type = 'none'
        chipSlots.value[i].patentChipId = null
        chipSlots.value[i].config = {
          normalSubtype: null,
          skillId: null,
          skillUpgraded: false,
          programName: '',
          programUpgraded: false,
          upgradeEffect: ''
        }
        validateAndFixPatentChips()
        return
      }
    }
  }
}

// 监听芯片变化，自动验证
watch(chipSlots, () => {
  validateAndFixPatentChips()
}, { deep: true })

// 专利芯片ID列表（仅ID编号）
const patentChipIds = computed(() => {
  return chipSlots.value
    .filter(slot => slot.type === 'patent' && slot.patentChipId)
    .map(slot => slot.patentChipId)
})

// 额外程序芯片详细数据
const extraPrograms = computed(() => {
  return chipSlots.value
    .filter(slot =>
      slot.type === 'normal' &&
      slot.config?.normalSubtype === 'program' &&
      slot.config?.programName
    )
    .map(slot => ({
      name: slot.config.programName,
      upgraded: slot.config.programUpgraded || false,
      upgradeEffect: slot.config.upgradeEffect || null
    }))
})

// ==================== 技能部分 ====================

// 技能数据（用于 AttributeAllocator）
const skillData = computed(() => {
  const data = {}
  skillsData.forEach(skill => {
    data[skill.id] = {
      name: skill.name,
      description: skill.description,
      effects: skill.effects
    }
  })
  return data
})

// 从芯片计算技能值（类似 M15 的算法）
const skillValues = computed(() => {
  const values = {}

  // 初始化所有技能为 0
  skillsData.forEach(skill => {
    values[skill.id] = 0
  })

  // 从芯片读取技能加成
  chipSlots.value.forEach(slot => {
    if (slot.type === 'normal' &&
        slot.config?.normalSubtype === 'skill' &&
        slot.config?.skillId) {
      const skillId = slot.config.skillId.toString()
      // 升级 +3，未升级 +2
      values[skillId] = slot.config.skillUpgraded ? 3 : 2
    }
  })

  return values
})

// 锁定范围（1~15 技能无法调整，min=max=当前值）
const lockedSkillRanges = computed(() => {
  const ranges = {}
  // 1~15 技能锁定
  for (let i = 1; i <= 15; i++) {
    const value = skillValues.value[i] || 0
    ranges[i] = {
      min: value,
      max: value,
      normalMin: value,
      normalMax: value
    }
  }
  // 人性（16）也锁定，因为 M7 不调整技能
  ranges[16] = {
    min: 0,
    max: 0,
    normalMin: 0,
    normalMax: 0
  }
  return ranges
})

// 更新技能值（所有技能只读，不处理更新）
const updateSkillValues = () => {
  // 所有技能只读，不处理更新
}

// 调整值规则
const modifierRules = [
  // 天穹防务 -> 闪避 +2
  {
    id: 'aetherguard_evasion',
    name: '天穹防务',
    watch: () => m2Output.value.manufacturerId,
    match: 2,
    value: 2,
    target: '2' // 闪避
  },
  // 海渊之子 -> 侦察 +2
  {
    id: 'abyssal_investigation',
    name: '海渊之子',
    watch: () => m2Output.value.manufacturerId,
    match: 7,
    value: 2,
    target: '10' // 侦察
  },
  // 人形机械 -> 交涉 -2
  {
    id: 'mechanical_doll_negotiation',
    name: '人形机械',
    watch: () => m2Output.value.traitIds,
    match: (traits) => traits && traits.includes('2'),
    value: -2,
    target: '15' // 交涉
  },
  // 偶像歌姬 -> 交涉 +2
  {
    id: 'idol_diva_negotiation',
    name: '偶像歌姬',
    watch: () => m2Output.value.traitIds,
    match: (traits) => traits && traits.includes('5'),
    value: 2,
    target: '15' // 交涉
  },
  // 俄狄浦斯 -> 交涉 +1
  {
    id: 'oedipus_negotiation',
    name: '俄狄浦斯',
    watch: () => m2Output.value.traitIds,
    match: (traits) => traits && traits.includes('12'),
    value: 1,
    target: '15' // 交涉
  },
  // 肉鸡僵尸（ID=7）-> 网络攻击（破解 ID=5）+2，网络防御（编程 ID=6）-2
  {
    id: 'zombie_bot_hacking',
    name: '肉鸡僵尸',
    watch: () => m2Output.value.traitIds,
    match: (traits) => traits && (traits.includes('7') || traits.includes(7)),
    value: 2,
    target: '5' // 破解（网络攻击）
  },
  {
    id: 'zombie_bot_programming',
    name: '肉鸡僵尸',
    watch: () => m2Output.value.traitIds,
    match: (traits) => traits && (traits.includes('7') || traits.includes(7)),
    value: -2,
    target: '6' // 编程（网络防御）
  }
]

// 获取技能调整值
function getSkillModifier(skillId) {
  if (!skillId) return 0
  const skillIdStr = skillId.toString()
  let modifier = 0

  modifierRules.forEach(rule => {
    if (rule.target !== skillIdStr) return
    const currentValue = rule.watch ? rule.watch() : null
    let matched = false
    if (typeof rule.match === 'function') {
      matched = rule.match(currentValue)
    } else if (rule.match !== undefined) {
      matched = currentValue === rule.match
    }
    if (matched) {
      modifier += rule.value
    }
  })

  return modifier
}

// 技能总值计算（包含芯片值和特质调整值）
const skillTotals = computed(() => {
  const totals = {}
  skillsData.forEach(skill => {
    const skillId = skill.id.toString()
    // 基础值（芯片）
    let value = skillValues.value[skillId] || 0
    // 加上调整值
    const modifier = getSkillModifier(skillId)
    totals[skillId] = value + modifier
  })
  return totals
})

// 技能加成列表（用于兼容旧代码）
const skillBonuses = computed(() => {
  return Object.entries(skillTotals.value)
    .filter(([_, value]) => value > 0)
    .map(([skillId, value]) => ({
      skillId,
      upgraded: value >= 3
    }))
})

// 是否应该输出数据（只有安装了专利芯片或常规芯片时才输出）
const shouldOutput = computed(() => {
  return patentCount.value > 0 || normalCount.value > 0
})

// 数据输出（分开输出各个字段）
useAutoOutput({
  patentChips: patentChipIds,
  skillBonuses,
  extraPrograms,
  skillValues: skillTotals,
  shouldOutput
})
</script>

<style lang="scss" scoped>
$cyber-cyan: #00f3ff;

.module-equipment {
  .slot-info {
    color: rgba(255, 255, 255, 0.7);
    font-size: 13px;
    font-family: "Courier New", monospace;

    .slot-count {
      color: $cyber-cyan;
      font-weight: 600;
    }

    .patent-info {
      color: rgba(255, 255, 255, 0.5);
      margin-left: 8px;
      font-size: 12px;
    }
  }

  .chip-slots-list {
    margin-top: 12px;
  }

  .rules-hint {
    margin-top: 16px;
    padding: 12px;
    background: rgba(0, 243, 255, 0.05);
    border: 1px dashed rgba(0, 243, 255, 0.3);
    border-radius: 4px;

    p {
      color: rgba(255, 255, 255, 0.5);
      font-size: 12px;
      margin: 4px 0;
      font-family: "Courier New", monospace;
    }

    .patent-disabled-warning {
      color: #ff2a6d;
      font-weight: 600;
      background: rgba(255, 42, 109, 0.1);
      padding: 8px;
      border-radius: 4px;
      border: 1px solid rgba(255, 42, 109, 0.3);
    }
  }

  .skills-section {
    margin-top: 24px;
    padding-top: 16px;
    border-top: 1px solid rgba($cyber-cyan, 0.1);

    .skills-list {
      margin-top: 12px;
    }
  }
}
</style>
