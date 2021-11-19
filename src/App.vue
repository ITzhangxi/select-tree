<template>
  <el-form ref="form" :model="formff" label-width="120px">
    <el-form-item label="Activity name">
      <select-tree
        v-model="formff.name"
        :treeData="treeData"
        nodeKey="id"
        clearable
        :selectProps="{ clearable: true }"
      ></select-tree>
    </el-form-item>
  </el-form>
</template>

<script>
import { defineComponent, ref } from "vue";
import SelectTree from "./components/SelectTree.vue";

export default defineComponent({
  name: "App",
  components: { SelectTree },
  setup() {
    const formff = ref({ name: 2 });
    return {
      formff,
      treeData: [
        {
          label: "Level one 1",
          id: 1,
          children: [
            {
              label: "Level two 1-1",
              id: 2,
              children: [
                {
                  label: "Level three 1-1-1",
                  id: 9,
                },
              ],
            },
          ],
        },
        {
          label: "Level one 2",
          id: 3,
          children: [
            {
              label: "Level two 2-1",
              id: 4,
              children: [
                {
                  label: "Level three 2-1-1",
                  id: 5,
                },
              ],
            },
            {
              label: "Level two 2-2",
              id: 6,
              children: [
                {
                  label: "Level three 2-2-1",
                  id: 7,
                },
              ],
            },
          ],
        },
      ],
    };
  },
});
</script>

<style></style>
