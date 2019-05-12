<template>
  <div class="block">
    <span class="demonstration">默认</span>

    <el-date-picker
      v-model="value1"
      type="monthrange"
      range-separator="至"
      value-format="yyyy-MM"
      start-placeholder="开始日期"
      end-placeholder="结束日期"
    ></el-date-picker>
    <el-date-picker
      v-model="value2"
      type="yearrange"
      range-separator="至"
      value-format="yyyy"
      start-placeholder="开始日期"
      end-placeholder="结束日期"
    ></el-date-picker>
    <el-date-picker
      v-model="value3"
      type="year"
      value-format="yyyy"
      placeholder="开始日期"
    ></el-date-picker>

    {{value1}}
    {{value2}}
    {{value3}}
  </div>
</template>

<script>
export default {
  data() {
    return {
      value1: '',
      value2: '',
      value3: ''
    };
  },
};
</script>
