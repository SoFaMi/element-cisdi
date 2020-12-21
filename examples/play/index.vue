<template>
  <div style="margin: 20px;">
    <el-input v-model="input" placeholder="请输入内容"></el-input>
    <el-date-picker format="yyyy-MM-dd HH:mm"
                    v-model="value" type="datetime"
                    :clearable="false" value-format="timestamp">
    </el-date-picker>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        input: 'Hello Element UI!',
        value: new Date(),
      };
    }
  };
</script>
