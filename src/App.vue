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
  <select-tree
    v-model="formff.name"
    :treeData="treeData"
    nodeKey="id"
    :selectProps="{ clearable: true }"
  ></select-tree>
  {{ typeof formff.name }}
  {{ formff.name }}
</template>

<script>
import { defineComponent, ref, watch } from "vue";
import SelectTree from "./components/SelectTree.vue";

export default defineComponent({
  name: "App",
  components: { SelectTree },
  setup() {
    const formff = ref({ name: 1 });
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
          status: true,
          children: [
            {
              label: "Level two 1-1",
              id: 2,
              status: true,
              children: [
                {
                  label: "Level three 1-1-1",
                  id: 9,
                  status: false,
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
