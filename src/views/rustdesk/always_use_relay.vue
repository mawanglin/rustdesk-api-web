<template>
  <el-card class="simple-card" shadow="hover" v-loading="form.loading">
    <template #header>
      <div class="card-header">
        <span>ALWAYS_USE_RELAY</span>
      </div>
    </template>
    <el-form :disabled="!canSend">
      <el-form-item>
        <el-switch v-model="form.option" active-value="Y" inactive-value="N"></el-switch>
      </el-form-item>
      <el-form-item>
        <el-button @click="get">{{ T('Refresh') }}</el-button>
        <el-button @click="save" type="primary">{{ T('Save') }}</el-button>
      </el-form-item>
    </el-form>
  </el-card>
</template>
<script setup>

  import { T } from '@/utils/i18n'
  import { reactive, watch } from 'vue'
  import { sendCmd } from '@/api/rustdesk'
  import { ElMessage } from 'element-plus'
  import { ID_TARGET } from '@/views/rustdesk/options'

  const props = defineProps({
    canSend: Boolean,
  })

  const form = reactive({
    cmd: 'aur',
    option: '',
    target: ID_TARGET,
    value: 0,
    loading: false,
  })
  const get = async () => {
    form.loading = true
    const res = await sendCmd({ cmd: 'aur', target: ID_TARGET }).catch(_ => false)
    form.loading = false
    if (res) {
      if (res.data === 'ALWAYS_USE_RELAY: true' || res.data === 'ALWAYS_USE_RELAY: true\n') {
        form.option = 'Y'
      } else {
        form.option = 'N'
      }
    }
  }
  // forapi bugfix: 这里以前保存成功后会通过 `success` 事件让 control.vue 立刻把
  // RelayServers 卡片当前的值也重新保存一遍——那是在补一个服务端 bug：hbbs 的
  // "aur" 命令处理会把这里发的 "Y"/"N" 误当成中继地址列表，悄悄把 relay-servers
  // 清空。真机 2026-09-24 复现两次（保存本开关后中继地址被清空、所有客户端连接
  // 失败）。服务端已直接修掉这个 bug（hbbs 的 "aur" 分支不再误发 RelayServers0），
  // 这个前端补丁式的自动重新保存不再需要，而且它本身有风险：如果 RelayServers
  // 卡片当时还没加载出真实值（form.option 仍是初始的空字符串），"重新保存"反而会
  // 主动把中继地址覆盖成空——所以直接删掉，不要再加回来。
  const save = async () => {
    const res = await sendCmd(form).catch(_ => false)
    if (res) {
      ElMessage.success(T('OperationSuccess'))
    }
  }

  watch(() => props.canSend, (v) => {
    if (v) {
      get()
    }
  })
</script>


<style scoped lang="scss">

</style>
