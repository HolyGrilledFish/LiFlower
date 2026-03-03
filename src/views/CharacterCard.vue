<template>
  <div class="character-card-container">
    <!-- 顶部标题栏 -->
    <div class="top-bar">
      <div class="title-section">
        <h1 class="main-title">锂之花 <span class="title-separator">//</span> 角色卡</h1>
        <p class="sub-title">Lithium Lilith Ludus <span class="version">v0.8</span></p>
      </div>
      <div class="type-switcher">
        <div class="switch-segments">
          <div
            class="segment"
            :class="{ active: character.type === 'Anthropos' }"
            @click="onTypeChange('Anthropos')"
          >
            人类 Anthropos
          </div>
          <div
            class="segment"
            :class="{ active: character.type === 'Anthroform' }"
            @click="onTypeChange('Anthroform')"
          >
            人形 Anthroform
          </div>
        </div>
      </div>
    </div>

    <div class="three-column-layout">
      <!-- 左栏：当前表单 -->
      <div class="column column-left">
        <el-card class="form-card">
          <template #header>
            <span class="card-header">基本信息</span>
          </template>
          <el-form :model="character" label-position="top">
            <el-form-item label="角色名称" required>
              <el-input
                v-model="character.name"
                placeholder="请输入角色名称"
                size="large"
              />
            </el-form-item>

            <el-form-item
              v-if="character.type === 'Anthroform'"
              label="硬件规格"
              required
            >
              <div style="display: flex; align-items: center; gap: 8px; width: 100%">
                <CyberSelect
                  v-model="character.hardwareSpec"
                  placeholder="请选择硬件规格"
                  :options="hardwareSpecOptions"
                  @change="onHardwareChange"
                  style="flex: 1; min-width: 0"
                />
                <el-popover
                  v-if="currentHardwareDesc"
                  placement="right"
                  :width="300"
                  trigger="hover"
                  :content="currentHardwareDesc"
                >
                  <template #reference>
                    <el-icon class="info-icon"><InfoFilled /></el-icon>
                  </template>
                </el-popover>
              </div>
              <div v-if="currentHardwareDesc" class="description-text">
                {{ currentHardwareDesc }}
              </div>
            </el-form-item>

            <el-form-item v-if="character.type === 'Anthroform'" label="生产企业">
              <div style="display: flex; align-items: center; gap: 8px; width: 100%">
                <CyberSelect
                  v-model="character.manufacturer"
                  placeholder="请选择生产企业（可选）"
                  :options="manufacturerOptions"
                  :clearable="true"
                  @change="onEnterpriseChange"
                  style="flex: 1; min-width: 0"
                />
                <el-popover
                  v-if="currentManufacturerDesc"
                  placement="right"
                  :width="300"
                  trigger="hover"
                  :content="currentManufacturerDesc"
                >
                  <template #reference>
                    <el-icon class="info-icon"><InfoFilled /></el-icon>
                  </template>
                </el-popover>
              </div>
              <div v-if="currentManufacturerDesc" class="description-text">
                {{ currentManufacturerDesc }}
              </div>
            </el-form-item>

            <!-- 属性点分配 -->
            <AttributeAllocator
              v-if="character.type === 'Anthroform' && character.attributePoints > 0"
              :attribute-points="character.attributePoints"
              :attribute-limit="character.attributeLimit"
              :attributes="character.attributes"
              :attribute-data="attributeData"
              :source-info="attributePointsInfo"
              @update:attributes="character.attributes = $event"
            />
          </el-form>
        </el-card>
      </div>

      <!-- 中间栏：待添加 -->
      <div class="column column-middle">
        <el-card class="placeholder-card">
          <template #header>
            <span class="card-header">中间栏位</span>
          </template>
          <div class="placeholder-content">
            <el-icon size="48" color="#c0c4cc"><Plus /></el-icon>
            <p>待添加内容</p>
          </div>
        </el-card>
      </div>

      <!-- 右栏：待添加 -->
      <div class="column column-right">
        <el-card class="placeholder-card">
          <template #header>
            <span class="card-header">右边栏位</span>
          </template>
          <div class="placeholder-content">
            <el-icon size="48" color="#c0c4cc"><Plus /></el-icon>
            <p>待添加内容</p>
          </div>
        </el-card>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, computed } from "vue";
import { ElMessage } from "element-plus";
import { Plus, InfoFilled, Warning } from "@element-plus/icons-vue";
import CyberSelect from "@/components/CyberSelect.vue";
import AttributeAllocator from "@/components/AttributeAllocator.vue";

// 导入数据文件
import hardwareSpecData from "@/data/硬件规格.json";
import manufacturerData from "@/data/企业技术.json";
import attributeDataJson from "@/data/基本属性.json";

/**
 * 角色数据对象
 * @property {String} name - 角色名称
 * @property {String} type - 角色类型：Anthropos(人类) / Anthroform(人形)
 * @property {String} hardwareSpec - 硬件规格
 * @property {String} manufacturer - 生产企业
 * @property {String} origin - 起源
 * @property {Number} attributePoints - 属性点总数
 * @property {Number} attributeLimit - 单项属性上限
 * @property {Object} attributes - 各项属性值
 */
const character = reactive({
  name: "",
  type: "Anthroform",
  hardwareSpec: "",
  manufacturer: "",
  origin: "",
  baseAttributePoints: 0,  // 基础属性点（来自硬件规格）
  attributePoints: 0,      // 总属性点（含企业加值）
  attributeLimit: 5,
  attributes: { structure: 0, strength: 0, athletics: 0, compute: 0, information: 0, power: 0 },
});

// 数据列表
const hardwareSpecList = hardwareSpecData;
const manufacturerList = manufacturerData;
const attributeData = attributeDataJson.attributeSystem;

/**
 * 硬件规格选项
 * @property {String} label - 显示名称
 * @property {String} value - 值
 * @property {String} extra - 额外信息（属性点和上限加值）
 */
const hardwareSpecOptions = hardwareSpecList.map((item) => ({
  label: item.name,
  value: item.name,
  extra: `属性点:${item.effect.attributePointsBonus} | 上限加值:${item.effect.attributeLimitBonus}`,
}));

/**
 * 生产企业选项
 * @property {String} label - 中文名称
 * @property {String} value - 值
 * @property {String} extra - 英文名称
 */
const manufacturerOptions = manufacturerList.map((item) => ({
  label: item.nameZh,
  value: item.nameZh,
  extra: item.nameEn,
}));

// 计算已分配属性点总数
const allocatedPoints = computed(() => {
  return Object.values(character.attributes).reduce((sum, val) => sum + val, 0);
});

// 计算属性：获取当前选择的硬件规格效果描述
const currentHardwareDesc = computed(() => {
  if (!character.hardwareSpec) return "";
  const selected = hardwareSpecList.find((item) => item.name === character.hardwareSpec);
  return selected ? selected.effectDescription || "" : "";
});

// 计算属性：获取当前选择的企业技术效果描述
const currentManufacturerDesc = computed(() => {
  if (!character.manufacturer) return "";
  const selected = manufacturerList.find((item) => item.nameZh === character.manufacturer);
  return selected ? selected.effectDescription || "" : "";
});

// 计算属性：获取当前企业技术的属性点加值
const manufacturerAttributeBonus = computed(() => {
  if (!character.manufacturer) return 0;
  const selected = manufacturerList.find((item) => item.nameZh === character.manufacturer);
  return selected?.effect?.attributePointsBonus || 0;
});

// 计算属性：显示属性点来源信息
const attributePointsInfo = computed(() => {
  const hardware = character.hardwareSpec || "未选择";
  const manufacturer = character.manufacturer || "无";
  const bonus = manufacturerAttributeBonus.value;
  return `硬件规格：${character.baseAttributePoints}${bonus > 0 ? ` + 企业加值：${bonus}` : ""}`;
});

/**
 * 切换角色类型
 * @param {String} type - 角色类型：Anthropos 或 Anthroform
 */
const onTypeChange = (type) => {
  character.type = type;
  if (type === "Anthropos") {
    character.hardwareSpec = "";
    character.manufacturer = "";
    character.baseAttributePoints = 0;
    character.attributePoints = 0;
    character.attributes = { structure: 0, strength: 0, athletics: 0, compute: 0, information: 0, power: 0 };
  }
};

/**
 * 硬件规格变更处理
 * @param {String} value - 选中的硬件规格名称
 */
const onHardwareChange = (value) => {
  const selected = hardwareSpecList.find((item) => item.name === value);
  if (selected) {
    // 设置基础属性点（来自硬件规格的效果）
    character.baseAttributePoints = selected.effect.attributePointsBonus;
    character.attributeLimit = 5;
    // 重置所有属性为 0
    character.attributes = { structure: 0, strength: 0, athletics: 0, compute: 0, information: 0, power: 0 };
    // 重置生产企业
    character.manufacturer = "";
    // 更新总属性点（基础 + 企业加值，此时企业加值为 0）
    character.attributePoints = character.baseAttributePoints;
  }
};

/**
 * 生产企业变更处理
 * @param {String} value - 选中的企业名称
 */
const onEnterpriseChange = (value) => {
  const oldBonus = manufacturerAttributeBonus.value;
  let newBonus = 0;

  if (!value) {
    // 取消选择企业，减去加值
    newBonus = 0;
    character.manufacturer = "";
    character.attributePoints = character.baseAttributePoints;
    ElMessage.info("已取消选择生产企业");
  } else {
    const selected = manufacturerList.find((item) => item.nameZh === value);
    if (selected) {
      newBonus = selected.effect?.attributePointsBonus || 0;
      character.manufacturer = value;
      // 更新总属性点（基础 + 新企业加值）
      character.attributePoints = character.baseAttributePoints + newBonus;
      ElMessage.success(
        `已选择 ${selected.nameZh} - ${selected.effectDescription || "无效果描述"}${newBonus > 0 ? ` (属性点 +${newBonus})` : ""}`
      );
    }
  }

  // 如果总属性点减少，从已分配属性中扣除
  const pointDifference = oldBonus - newBonus;
  if (pointDifference > 0 && allocatedPoints.value > 0) {
    // 需要扣除的属性点数
    let pointsToDeduct = pointDifference;
    if (pointsToDeduct > allocatedPoints.value) {
      pointsToDeduct = allocatedPoints.value;
    }

    // 从各属性中平均扣除
    const newAttrs = { ...character.attributes };
    const attributeKeys = Object.keys(newAttrs);

    while (pointsToDeduct > 0) {
      // 找到当前值大于 0 的属性
      const availableAttrs = attributeKeys.filter(key => newAttrs[key] > 0);
      if (availableAttrs.length === 0) break;

      // 按顺序从每个属性扣除 1 点
      for (const key of availableAttrs) {
        if (pointsToDeduct <= 0) break;
        if (newAttrs[key] > 0) {
          newAttrs[key] -= 1;
          pointsToDeduct -= 1;
        }
      }
    }

    character.attributes = newAttrs;
    if (pointDifference > allocatedPoints.value) {
      ElMessage.warning(`企业变更导致属性点不足，已重置所有属性`);
    } else {
      ElMessage.warning(`企业变更导致属性点减少 ${pointDifference} 点，已从已分配属性中扣除`);
    }
  }
};
</script>

<style lang="scss" scoped>
// 赛博朋克配色
$cyber-cyan: #00f3ff;
$cyber-purple: #bc13fe;
$cyber-dark: #0a0a0f;
$cyber-darker: #050508;

.character-card-container {
  min-height: 100vh;
  background-color: $cyber-darker;
  padding-bottom: 40px;
}

// 顶部标题栏
.top-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 30px 40px;
  background: linear-gradient(135deg, rgba(10, 10, 15, 0.95), rgba(26, 26, 46, 0.95));
  border-bottom: 2px solid $cyber-cyan;
  box-shadow: 0 4px 20px rgba(0, 243, 255, 0.15);
  position: relative;

  &::after {
    content: "";
    position: absolute;
    bottom: -2px;
    left: 0;
    width: 100%;
    height: 2px;
    background: linear-gradient(90deg, transparent, $cyber-cyan, transparent);
  }
}

.title-section {
  flex: 1;
}

.main-title {
  font-size: 32px;
  font-weight: 700;
  color: $cyber-cyan;
  margin: 0;
  font-family: "Courier New", "Consolas", monospace;
  text-shadow: 0 0 10px rgba(0, 243, 255, 0.5);
  letter-spacing: 4px;

  .title-separator {
    color: $cyber-purple;
    margin: 0 8px;
  }
}

.sub-title {
  font-size: 12px;
  color: rgba(255, 255, 255, 0.6);
  margin: 8px 0 0 0;
  font-family: "Courier New", "Consolas", monospace;
  letter-spacing: 2px;

  .version {
    color: $cyber-purple;
    font-weight: 700;
  }
}

// 类型切换器
.type-switcher {
  flex-shrink: 0;
}

.switch-segments {
  display: flex;
  background: rgba(26, 26, 46, 0.8);
  border: 1px solid rgba(0, 243, 255, 0.3);
  border-radius: 4px;
  overflow: hidden;
}

.segment {
  padding: 12px 24px;
  font-size: 13px;
  color: rgba(255, 255, 255, 0.6);
  cursor: pointer;
  transition: all 0.3s ease;
  font-family: "Courier New", "Consolas", monospace;
  border-right: 1px solid rgba(0, 243, 255, 0.2);

  &:last-child {
    border-right: none;
  }

  &:hover {
    background: rgba(0, 243, 255, 0.1);
    color: $cyber-cyan;
  }

  &.active {
    background: linear-gradient(135deg, rgba(0, 243, 255, 0.3), rgba(188, 19, 254, 0.3));
    color: $cyber-cyan;
    font-weight: 700;
    box-shadow: inset 0 0 10px rgba(0, 243, 255, 0.2);
  }
}

// 三栏布局
.three-column-layout {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 24px;
  padding: 30px 40px;
  max-width: 1800px;
  margin: 0 auto;
}

.column {
  min-width: 0;
}

.form-card,
.placeholder-card {
  height: 100%;
  background: linear-gradient(145deg, rgba(26, 26, 46, 0.9), rgba(10, 10, 15, 0.95));
  border: 1px solid rgba(0, 243, 255, 0.2);
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  transition: all 0.3s ease;

  &:hover {
    box-shadow: 0 8px 30px rgba(0, 243, 255, 0.15);
    border-color: rgba(0, 243, 255, 0.4);
  }
}

.card-header {
  font-size: 14px;
  font-weight: 600;
  color: $cyber-cyan;
  font-family: "Courier New", "Consolas", monospace;
  letter-spacing: 2px;
}

:deep(.el-card__header) {
  background: rgba(10, 10, 15, 0.8);
  border-bottom-color: rgba(0, 243, 255, 0.2);
  padding: 14px 20px;
}

:deep(.el-card__body) {
  padding: 20px;
}

// 表单样式
:deep(.el-form-item__label) {
  color: $cyber-cyan;
  font-family: "Courier New", "Consolas", monospace;
  font-size: 12px;
  letter-spacing: 1px;
  margin-bottom: 8px;
}

:deep(.el-form-item) {
  width: 100%;
}

:deep(.el-form-item__content) {
  width: 100% !important;
}

// 信息图标样式
.info-icon {
  color: $cyber-cyan;
  font-size: 18px;
  cursor: pointer;
  transition: all 0.3s ease;

  &:hover {
    color: rgba(0, 243, 255, 0.7);
    transform: scale(1.1);
  }
}

// 描述文字样式
.description-text {
  margin-top: 8px;
  padding: 10px 12px;
  width: 100%;
  box-sizing: border-box;
  background: rgba(0, 243, 255, 0.05);
  border: 1px solid rgba(0, 243, 255, 0.2);
  border-radius: 4px;
  color: rgba(255, 255, 255, 0.7);
  font-size: 13px;
  line-height: 1.6;
  font-family: "Microsoft YaHei", sans-serif;

  &.warning-text {
    background: rgba(255, 193, 7, 0.1);
    border-color: rgba(255, 193, 7, 0.3);
    color: rgba(255, 193, 7, 0.9);
    display: flex;
    align-items: center;
    gap: 6px;
  }
}

// Popover 样式
:deep(.el-popover) {
  background: rgba(10, 10, 15, 0.98) !important;
  border: 1px solid rgba(0, 243, 255, 0.3) !important;
  box-shadow: 0 4px 20px rgba(0, 243, 255, 0.15) !important;
  color: rgba(255, 255, 255, 0.8) !important;
  font-family: "Microsoft YaHei", sans-serif;
  font-size: 13px;
  line-height: 1.6;
}

// 输入框样式
:deep(.el-input__wrapper) {
  background: rgba(10, 10, 15, 0.8);
  border: 1px solid rgba(0, 243, 255, 0.2);
  box-shadow: none;

  .el-input__inner {
    color: #fff;
    font-family: "Courier New", "Consolas", monospace;
  }

  &:focus-within {
    box-shadow: 0 0 10px rgba(0, 243, 255, 0.2);
    border-color: $cyber-cyan;
  }
}

.placeholder-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 60px 20px;
  color: rgba(0, 243, 255, 0.4);

  p {
    margin-top: 16px;
    font-size: 13px;
    color: rgba(192, 196, 204, 0.5);
    font-family: "Courier New", "Consolas", monospace;
  }
}

// 响应式
@media (max-width: 1200px) {
  .three-column-layout {
    grid-template-columns: 1fr;
    padding: 20px;
  }

  .top-bar {
    flex-direction: column;
    gap: 20px;
    padding: 20px;
  }

  .title-section {
    text-align: center;
  }

  .main-title {
    font-size: 24px;
  }

  .switch-segments {
    width: 100%;
  }

  .segment {
    flex: 1;
    text-align: center;
  }
}

@media (min-width: 1201px) and (max-width: 1600px) {
  .three-column-layout {
    grid-template-columns: 1fr 1fr;
  }

  .column-right {
    grid-column: span 2;
  }
}
</style>
