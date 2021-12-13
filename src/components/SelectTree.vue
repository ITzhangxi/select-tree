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
    <!-- <el-option
      v-for="item in Object.values(map)"
      :key="item[nodeKey]"
      :label="item[props.label]"
      :value="item[nodeKey]"
      style="display: none"
    >
    </el-option> -->
    <template #empty>
      <el-tree
        class="select-tree"
        :data="treeData"
        :node-key="nodeKey"
        @node-click="handleNodeClick"
        :expand-on-click-node="false"
        :default-checked-keys="checkedKeys"
        :default-expanded-keys="expandedKeys"
        :props="props"
        v-bind="$attrs.treeProps"
      />
    </template>
  </el-select>
</template>
<script>
import { defineComponent, watch, ref, getCurrentInstance } from "vue";
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
  emits: ["update:modelValue", "node-click", "clear"],
  setup(props, context) {
    const self = getCurrentInstance();
    const ctx = self.ctx || {};
    const $attrs = ctx.$attrs || {};
    const selectProps = $attrs.selectProps || {};
    const map = mapData(props.treeData, props.nodeKey, props.props.children);

    const selectVal = ref(selectProps.multiple ? [] : "");
    const checkedKeys = ref([]);
    const expandedKeys = ref([]);

    watch(
      () => props.modelValue,
      (val) => {
        const mVal = Array.isArray(val) ? val : [val];
        if (selectProps.multiple) {
          selectVal.value = [];
          mVal.forEach((item) => {
            selectVal.value.push(
              (map[item] && map[item][props.props.label]) || item
            );
          });
        } else {
          selectVal.value = (map[val] && map[val][props.props.label]) || val;
        }
        checkedKeys.value = [...mVal];
        expandedKeys.value = [...mVal];
        setActive(mVal);
      },
      { immediate: true, deep: true }
    );

    const selectRef = ref(null);

    const handleNodeClick = (val) => {
      const modelValue = Array.isArray(props.modelValue)
        ? props.modelValue
        : [];
      let selectVal = new Set(modelValue);
      if (selectProps.multiple) {
        selectVal.has(val[props.nodeKey])
          ? selectVal.delete(val[props.nodeKey])
          : selectVal.add(val[props.nodeKey]);
        selectVal = [...selectVal];
      } else {
        selectVal = val[props.nodeKey];
        selectRef.value && selectRef.value.blur();
      }
      context.emit("update:modelValue", selectVal);
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
      selectVal,
      checkedKeys,
      expandedKeys,
      selectRef,
      handleRemoveTag,
      handleClear,
      map,
    };
  },
});
</script>
<style lang="scss">
.el-tree-node:focus > .el-tree-node__content {
  background-color: transparent !important;
}
</style>
