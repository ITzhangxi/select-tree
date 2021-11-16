<template>
  <el-select v-model="selectVal" :placeholder="placeholder" class="select-tree">
    <template #empty>
      <el-tree
        class="select-tree"
        :data="treeData"
        :node-key="nodeKey"
        @node-click="handleNodeClick"
        :expand-on-click-node="false"
        :default-checked-keys="checkedKeys"
        :default-expanded-keys="expandedKeys"
      />
    </template>
  </el-select>
</template>
<script>
import { defineComponent, watch, ref } from "vue";
const setActive = (() => {
  const sty = document.createElement("style");
  document.body.appendChild(sty);
  return function (id) {
    sty.innerText = `
        .is-checked[data-key="${id}"]>.el-tree-node__content {
            background-color: #f5f7fa !important;
        }
        `;
  };
})();

function mapData(arr = [], id) {
  const map = {};
  arr.forEach((item) => {
    map[item[id]] = item;
    if (Array.isArray(item.children)) {
      Object.assign(map, mapData(item.children, id));
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
      type: [String, Number],
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
  emits: ["update:modelValue", "node-click"],
  setup(props, context) {
    const map = mapData(props.treeData, props.nodeKey);

    const selectVal = ref("");
    const checkedKeys = ref([]);
    const expandedKeys = ref([]);

    watch(
      () => props.modelValue,
      (val) => {
        selectVal.value = (map[val] && map[val].label) || val;
        checkedKeys.value = [val];
        expandedKeys.value = [val];
        setActive(val);
      },
      { immediate: true, deep: true }
    );

    const handleNodeClick = (val) => {
      context.emit("update:modelValue", val[props.nodeKey]);
      context.emit("node-click", ...arguments);
    };
    return { handleNodeClick, selectVal, checkedKeys, expandedKeys };
  },
});
</script>
<style lang="scss">
.el-tree-node:focus > .el-tree-node__content {
  background-color: transparent !important;
}
</style>
