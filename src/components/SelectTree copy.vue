<template>
  <el-select v-model="selectVal" placeholder="Select">
    <template #empty>
      <el-tree :data="treeData" @node-click="handleNodeClick" />
    </template>
  </el-select>
</template>
<script>
import { defineComponent, watch, ref } from "vue";
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
    value: String,
  },
  setup(props, context) {
    const handleNodeClick = () => {};
    const selectVal = ref("");
    watch(
      () => props.value,
      (val) => {
        selectVal.value = val;
      },
      { immediate: true }
    );
    watch(
      () => selectVal.value,
      (val) => {
        context.$emit("input", val);
      },
      { immediate: true }
    );
    return { handleNodeClick, selectVal };
  },
});
</script>
<style lang="less" scoped>
.select-tree {
}
</style>
