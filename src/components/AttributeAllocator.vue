<template>
  <div class="attribute-allocator">
    <el-divider v-if="showDivider" />
    <el-form-item label="属性点分配">
      <div class="attribute-section">
        <div class="attribute-points-info">
          <span>可用属性点：<span class="points-highlight">{{ remainingPoints }}</span> / {{ totalPoints }}</span>
        </div>
        <div v-if="sourceInfo" class="attribute-source-info">
          {{ sourceInfo }}
        </div>

        <div v-for="attrName in attributeOrder" :key="attrName" class="attribute-row">
          <div class="attribute-label">
            <span>{{ getAttributeName(attrName) }}</span>
            <el-tooltip :content="getAttributeDescription(attrName)" placement="top" :disabled="!getAttributeDescription(attrName)">
              <el-icon class="info-icon-small"><InfoFilled /></el-icon>
            </el-tooltip>
          </div>
          <div class="attribute-controls">
            <el-button size="small" :disabled="!canDecrease(attrName)" @click="decreaseAttribute(attrName)" class="attr-btn">
              <el-icon><Minus /></el-icon>
            </el-button>
            <span class="attribute-value">{{ getCurrentAttributeValue(attrName) }}</span>
            <el-button size="small" :disabled="!canIncrease(attrName)" @click="increaseAttribute(attrName)" class="attr-btn">
              <el-icon><Plus /></el-icon>
            </el-button>
          </div>
        </div>
      </div>
    </el-form-item>
  </div>
</template>

<script setup>
import { computed } from "vue";
import { ElMessage } from "element-plus";
import { Plus, Minus, InfoFilled } from "@element-plus/icons-vue";

/**
 * Props 定义
 * @property {Number} attributePoints - 总属性点数
 * @property {Number} attributeLimit - 单项属性上限（默认 5）
 * @property {Object} attributes - 当前各属性值对象 { structure, strength, athletics, compute, information, power }
 * @property {Object} attributeData - 属性列表数据（用于获取描述）
 * @property {Boolean} showDivider - 是否显示分割线
 * @property {String} sourceInfo - 属性点来源信息
 */
const props = defineProps({
  attributePoints: {
    type: Number,
    default: 0,
  },
  attributeLimit: {
    type: Number,
    default: 5,
  },
  attributes: {
    type: Object,
    default: () => ({ structure: 0, strength: 0, athletics: 0, compute: 0, information: 0, power: 0 }),
  },
  attributeData: {
    type: Object,
    default: () => ({}),
  },
  showDivider: {
    type: Boolean,
    default: true,
  },
  sourceInfo: {
    type: String,
    default: "",
  },
});

// 属性顺序（使用英文键名）：structure(结构), strength(力量), athletics(运动), compute(算力), information(信息), power(功率)
const attributeOrder = ["structure", "strength", "athletics", "compute", "information", "power"];

// 属性名称映射（英文键名 -> 中文显示名）
const attributeNameMap = {
  structure: "结构",
  strength: "力量",
  athletics: "运动",
  compute: "算力",
  information: "信息",
  power: "功率",
};

/**
 * Emit 定义
 * @event update:attributes - 更新属性值
 */
const emit = defineEmits(["update:attributes"]);

// 计算已分配属性点总数
const allocatedPoints = computed(() => {
  return Object.values(props.attributes).reduce((sum, val) => sum + val, 0);
});

// 计算剩余可分配属性点
const remainingPoints = computed(() => {
  return props.attributePoints - allocatedPoints.value;
});

// 获取当前属性值
const getCurrentAttributeValue = (attrName) => {
  return props.attributes[attrName] || 0;
};

// 获取属性显示名称
const getAttributeName = (attrName) => {
  return attributeNameMap[attrName] || attrName;
};

// 检查属性是否可以增加（未达上限且有剩余点数）
const canIncrease = (attrName) => {
  const currentValue = props.attributes[attrName] || 0;
  return currentValue < props.attributeLimit && remainingPoints.value > 0;
};

// 检查属性是否可以减少（当前值大于 0）
const canDecrease = (attrName) => {
  return (props.attributes[attrName] || 0) > 0;
};

// 增加属性
const increaseAttribute = (attrName) => {
  if (canIncrease(attrName)) {
    const newAttrs = { ...props.attributes, [attrName]: (props.attributes[attrName] || 0) + 1 };
    emit("update:attributes", newAttrs);
  } else {
    const currentValue = props.attributes[attrName] || 0;
    if (currentValue >= props.attributeLimit) {
      ElMessage.warning("单项属性上限为 5 点");
    } else {
      ElMessage.warning("没有足够的属性点");
    }
  }
};

// 减少属性
const decreaseAttribute = (attrName) => {
  if (canDecrease(attrName)) {
    const newAttrs = { ...props.attributes, [attrName]: (props.attributes[attrName] || 0) - 1 };
    emit("update:attributes", newAttrs);
  }
};

// 获取属性描述
const getAttributeDescription = (attrName) => {
  const attr = props.attributeData[attrName];
  return attr ? attr.description : "";
};
</script>

<style lang="scss" scoped>
// 赛博朋克配色
$cyber-cyan: #00f3ff;
$cyber-purple: #bc13fe;
$cyber-dark: #0a0a0f;
$cyber-darker: #050508;

.attribute-allocator {
  width: 100%;
}

.attribute-section {
  width: 100%;
}

.attribute-points-info {
  margin-bottom: 12px;
  padding: 8px 12px;
  background: rgba(0, 243, 255, 0.08);
  border-radius: 4px;
  font-size: 13px;
  color: rgba(255, 255, 255, 0.8);
  font-family: "Courier New", "Consolas", monospace;

  .points-highlight {
    color: $cyber-cyan;
    font-weight: 700;
    font-size: 16px;
  }
}

.attribute-source-info {
  margin-bottom: 12px;
  padding: 6px 10px;
  background: rgba(188, 19, 254, 0.1);
  border-radius: 4px;
  font-size: 12px;
  color: rgba(188, 19, 254, 0.8);
  font-family: "Courier New", "Consolas", monospace;
  border: 1px solid rgba(188, 19, 254, 0.2);
}

.attribute-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 0;
  border-bottom: 1px solid rgba(0, 243, 255, 0.1);

  &:last-child {
    border-bottom: none;
  }
}

.attribute-label {
  display: flex;
  align-items: center;
  gap: 6px;
  color: rgba(255, 255, 255, 0.7);
  font-size: 13px;
  font-family: "Microsoft YaHei", sans-serif;
}

.attribute-controls {
  display: flex;
  align-items: center;
  gap: 8px;

  .attribute-value {
    min-width: 24px;
    text-align: center;
    color: $cyber-cyan;
    font-size: 16px;
    font-weight: 700;
    font-family: "Courier New", "Consolas", monospace;
  }

  .attr-btn {
    width: 28px;
    height: 28px;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    border-color: rgba(0, 243, 255, 0.3);
    background: rgba(0, 243, 255, 0.05);
    color: $cyber-cyan;

    &:hover:not(:disabled) {
      background: rgba(0, 243, 255, 0.15);
      border-color: $cyber-cyan;
    }

    &:disabled {
      opacity: 0.3;
      cursor: not-allowed;
    }

    .el-icon {
      font-size: 14px;
    }
  }
}

// 小信息图标
.info-icon-small {
  color: rgba(0, 243, 255, 0.5);
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s ease;

  &:hover {
    color: $cyber-cyan;
  }
}
</style>
