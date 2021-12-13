<template>
  <el-form ref="formRef" :model="formff" label-width="120px">
    <el-form-item
      label="Activity name"
      prop="name"
      :rules="[
        {
          required: true,
          message: 'Please input email address',
          trigger: 'change',
        },
      ]"
    >
      <select-tree
        v-model="formff.name"
        :treeData="treeData"
        nodeKey="id"
        :selectProps="{ clearable: true }"
      ></select-tree>
    </el-form-item>
  </el-form>
  <el-button @click="handleSubmit">Submit</el-button>
</template>

<script>
import { defineComponent, ref, watch } from "vue";
import SelectTree from "./components/SelectTree.vue";

export default defineComponent({
  name: "App",
  components: { SelectTree },
  setup() {
    const formff = ref({ name: "" });
    watch(
      () => formff.value.name,
      () => {
        console.log("watch", formff.value.name);
      }
    );
    const nameValidator = (rule, value, callback) => {
      console.log("rule", rule);
      console.log("value", value);
      callback(new Error("adsfas"));
    };
    const formRef = ref(null);
    const handleSubmit = () => {
      if (formRef.value) {
        formRef.value.validate();
      }
    };
    return {
      nameValidator,
      handleSubmit,
      formRef,
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
