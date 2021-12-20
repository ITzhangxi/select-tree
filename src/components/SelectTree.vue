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
      />
    </template>
  </el-select>
</template>
<script>
import { defineComponent, watch, ref, getCurrentInstance, nextTick } from "vue";
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
  emits: ["update:modelValue", "node-click", "clear", "check"],
  setup(props, context) {
    const self = getCurrentInstance();
    const ctx = self.ctx || {};
    const $attrs = ctx.$attrs || {};
    const selectProps = $attrs.selectProps || {};
    const map = mapData(props.treeData, props.nodeKey, props.props.children);

    const selectVal = ref(selectProps.multiple ? [] : "");
    const checkedKeys = ref([]);
    const expandedKeys = ref([]);
    const selectRef = ref(null);
    const treeRef = ref(null);

    // 设置tree选中数据
    const setCheckedKeys = (id, reset = false) => {
      const ids = Array.isArray(id) ? id : [id];
      if (selectProps.multiple) {
        if (reset) checkedKeys.value = [];
        ids.forEach((item) => {
          const index = checkedKeys.value.indexOf(item);
          if (index === -1) checkedKeys.value.push(item);
          else checkedKeys.value.splice(index, 1);
        });
      } else {
        // eslint-disable-next-line no-debugger
        debugger;
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
      if (selectProps.multiple) {
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
        return context.emit("update:modelValue", "");
      if (selectProps.multiple) {
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
      },
      {
        immediate: true,
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
      const id = val.id || "";
      setCheckedKeys(id);
      context.emit("node-click", ...arguments);
    };
    const handleCheck = (node, { checkedKeys: handleCheckedKeys = [] }) => {
      if (selectProps.multiple) {
        setCheckedKeys(handleCheckedKeys);
      } else {
        setCheckedKeys(node[props.nodeKey]);
      }
      context.emit("node-click", ...arguments);
    };
    const handleRemoveTag = () => {
      // context.emit("update:modelValue", "");
    };
    const handleClear = () => {
      context.emit("update:modelValue", "");
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
</style>
