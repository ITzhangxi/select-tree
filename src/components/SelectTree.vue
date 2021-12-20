<template>
  <el-select
    v-model="selectVal"
    :placeholder="placeholder"
    class="select-tree"
    ref="selectRef"
    v-bind="$attrs.selectProps"
    @remove-tag="handleRemoveTag"
    @clear="handleClear"
  >
    <template #empty>
      <el-tree
        class="select-tree"
        :data="treeData"
        :node-key="nodeKey"
        :show-checkbox="true"
        :check-strictly="true"
        @check="handleCheck"
        @node-click="handleNodeClick"
        ref="treeRef"
        :expand-on-click-node="false"
        :checked-keys="checkedKeys"
        :expanded-keys="expandedKeys"
        :props="props"
        v-bind="$attrs.treeProps"
      >
      </el-tree>
    </template>
  </el-select>
</template>
<script>
import {
  defineComponent,
  watch,
  ref,
  getCurrentInstance,
  nextTick,
  computed,
} from "vue";
const setActive = (() => {
  const sty = document.createElement("style");
  document.body.appendChild(sty);
  return function (ids) {
    sty.innerText = ids
      .map((id) => {
        return `
        .is-checked[data-key="${id}"]>.el-tree-node__content {
            background-color: #f5f7fa !important;
        }
        `;
      })
      .join("");
  };
})();

function mapData(arr = [], id, children) {
  const map = {};
  arr.forEach((item) => {
    map[item[id]] = item;
    if (Array.isArray(item[children])) {
      Object.assign(map, mapData(item[children], id, children));
    }
  });
  return map;
}
export default defineComponent({
  name: "SelectTree",
  props: {
    treeData: {
      type: Array,
      default: () => [],
    },
    placeholder: {
      type: String,
      default: "请选择",
    },
    modelValue: {
      type: [String, Number, Array],
      default: "",
    },
    nodeKey: {
      type: String,
      required: true,
      default: "label",
    },
    props: {
      type: Object,
      default: () => ({ children: "children", label: "label" }),
    },
  },
  emits: ["update:modelValue", "node-click", "clear", "check", "remove-tag"],
  setup(props, context) {
    const self = getCurrentInstance();
    const ctx = self.ctx || {};
    const $attrs = ctx.$attrs || {};
    const selectProps = $attrs.selectProps || {};
    const map = mapData(props.treeData, props.nodeKey, props.props.children);
    const isMultiple = computed(() => selectProps.multiple);

    const selectVal = ref(isMultiple.value ? [] : "");
    const checkedKeys = ref([]);
    const expandedKeys = ref([]);
    const selectRef = ref(null);
    const treeRef = ref(null);

    // 设置tree选中数据
    const setCheckedKeys = (id, reset = false) => {
      const ids = Array.isArray(id) ? id : [id];
      if (isMultiple.value) {
        const keys = reset ? [] : [...checkedKeys.value];
        ids.forEach((item) => {
          const index = keys.indexOf(item);
          if (index === -1) keys.push(item);
          else keys.splice(index, 1);
        });
        checkedKeys.value = [...new Set([...keys])];
      } else {
        if (!ids || !Array.isArray(ids) || !ids.length)
          return (checkedKeys.value = []);
        if (checkedKeys.value.includes(id)) {
          checkedKeys.value = [];
        } else {
          checkedKeys.value = ids;
        }
      }
    };

    // 设置select选择的值
    const setSelectVal = (ids) => {
      if (!Array.isArray(ids) || !ids.length) return (selectVal.value = "");
      if (isMultiple.value) {
        selectVal.value = [];
        ids.forEach((id) => {
          selectVal.value.push((map[id] && map[id][props.props.label]) || id);
        });
      } else {
        const id = ids[0];
        const value = (map[id] && map[id][props.props.label]) || id;
        selectVal.value = value;
      }
    };

    // 设置node checkbox
    const setSelectNodeCheckbox = (ids) => {
      nextTick(() => {
        treeRef.value.setCheckedKeys(ids);
      });
    };
    // 更新 v-model
    const updateModel = (ids) => {
      if (!Array.isArray(ids) || !ids.length)
        return context.emit("update:modelValue", isMultiple.value ? [] : "");
      if (isMultiple.value) {
        context.emit("update:modelValue", ids);
      } else {
        const id = ids[0];
        context.emit("update:modelValue", id);
      }
    };

    watch(
      () => checkedKeys.value,
      (ids = []) => {
        setSelectVal(ids);
        // 选项背景高亮
        setActive(ids);
        // node选择
        setSelectNodeCheckbox(ids);
        updateModel(ids);
        expandedKeys.value = [...ids];
      }
    );

    watch(
      () => props.modelValue,
      (id) => {
        const ids = Array.isArray(id) ? id : [id];
        if (JSON.stringify(ids) !== JSON.stringify(checkedKeys.value))
          setCheckedKeys(ids, true);
      },
      { immediate: true }
    );

    const handleNodeClick = (val = {}) => {
      if (val.disabled) return;
      const id = val.id || "";
      setCheckedKeys(id);
      context.emit("node-click", ...arguments);
    };
    const handleCheck = (node, { checkedKeys: handleCheckedKeys = [] }) => {
      if (selectProps.multiple) {
        setCheckedKeys(handleCheckedKeys, true);
      } else {
        setCheckedKeys(node[props.nodeKey]);
      }
      context.emit("node-click", ...arguments);
    };
    const handleRemoveTag = (tag) => {
      const data = Object.values(map).find(
        (item) => item[props.props.label] === tag
      );
      if (data) {
        const id = data[props.nodeKey] || "";
        const keys = [...checkedKeys.value];
        const index = keys.indexOf(id);
        if (index !== -1) keys.splice(index, 1);
        checkedKeys.value = keys;
        context.emit("remove-tag", data);
      }
    };
    const handleClear = () => {
      context.emit("update:modelValue", isMultiple.value ? [] : "");
      context.emit("clear", ...arguments);
    };

    return {
      handleNodeClick,
      handleCheck,
      selectVal,
      checkedKeys,
      expandedKeys,
      selectRef,
      handleRemoveTag,
      handleClear,
      map,
      treeRef,
    };
  },
});
</script>
<style lang="scss">
.el-tree-node:focus > .el-tree-node__content {
  background-color: transparent !important;
}
.custom-select-tree-node {
  background-color: red;
}
</style>
