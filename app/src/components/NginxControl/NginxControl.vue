<script setup lang="ts">
import { message } from 'ant-design-vue'
import { ReloadOutlined } from '@ant-design/icons-vue'
import gettext from '@/gettext'
import ngx from '@/api/ngx'
import { logLevel } from '@/views/config/constants'

const { $gettext } = gettext
const status = ref(0)
function get_status() {
  ngx.status().then(r => {
    if (r?.running === true)
      status.value = 0
    else
      status.value = -1
  })
}

function reload_nginx() {
  status.value = 1
  ngx.reload().then(r => {
    if (r.level < logLevel.Warn)
      message.success($gettext('Nginx reloaded successfully'))
    else if (r.level === logLevel.Warn)
      message.warn(r.message)
    else
      message.error(r.message)
  }).catch(e => {
    message.error(`${$gettext('Server error')} ${e?.message}`)
  }).finally(() => {
    status.value = 0
  })
}

function restart_nginx() {
  status.value = 2
  ngx.restart().then(r => {
    if (r.level < logLevel.Warn)
      message.success($gettext('Nginx restarted successfully'))
    else if (r.level === logLevel.Warn)
      message.warn(r.message)
    else
      message.error(r.message)
  }).catch(e => {
    message.error(`${$gettext('Server error')} ${e?.message}`)
  }).finally(() => {
    status.value = 0
  })
}

const visible = ref(false)

watch(visible, v => {
  if (v)
    get_status()
})
</script>

<template>
  <APopover
    v-model:open="visible"
    placement="bottomRight"
    @confirm="reload_nginx"
  >
    <template #content>
      <div class="content-wrapper">
        <h4>{{ $gettext('Nginx Control') }}</h4>
        <ABadge
          v-if="status === 0"
          color="green"
          :text="$gettext('Running')"
        />
        <ABadge
          v-else-if="status === 1"
          color="blue"
          :text="$gettext('Reloading')"
        />
        <ABadge
          v-else-if="status === 2"
          color="orange"
          :text="$gettext('Restarting')"
        />
        <ABadge
          v-else
          color="red"
          :text="$gettext('Stopped')"
        />
      </div>
      <ASpace>
        <AButton
          size="small"
          type="link"
          @click="restart_nginx"
        >
          {{ $gettext('Restart') }}
        </AButton>
        <AButton
          size="small"
          type="link"
          @click="reload_nginx"
        >
          {{ $gettext('Reload') }}
        </AButton>
      </ASpace>
    </template>
    <a>
      <ReloadOutlined />
    </a>
  </APopover>
</template>

<style lang="less" scoped>
a {
  color: #000000;
}

.dark {
  a {
    color: #fafafa;
  }
}

.content-wrapper {
  text-align: center;
  padding-top: 5px;
  padding-bottom: 5px;

  h4 {
    margin-bottom: 5px;
  }
}
</style>
