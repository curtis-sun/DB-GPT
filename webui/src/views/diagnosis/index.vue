<template>
  <div style="height: calc(100vh - 30px)">
    <el-container style="height: 100%">
      <el-aside width="200px" style="border-right: 1px solid var(--border-color); height: 100%;">
        <div class="columnSS" style="width: 100%; margin-top: 20px; padding: 10px">
          <div class="rowBC" style="margin: 0 0 10px 10px; width: calc(100% - 20px)">
            <div style="font-weight: bold; font-size: 20px; ">{{ $t('Diagnosis') }}</div>

            <el-popover
                placement="right"
                title=""
                effect="dark"
                width="600"
                trigger="click"
            >
              <div ref="outputScrollDiv" class="diagnose-output-container">
                <div class="output" style="white-space: pre-wrap; word-break: break-all" v-html="diagnoseTerminalOutputHtml"/>
                <div v-if="diagnoseStatus" style="position: fixed; right: 30px; bottom: 100px">
                  <el-icon class="is-loading" size="26" style="color: limegreen; font-weight: bold">
                    <Loading/>
                  </el-icon>
                </div>
              </div>
              <template #reference>
                <div class="relative" style="cursor: pointer">
                  <div class="blinking-dot" />
                  <div style="margin-top:  5px">
                    <svg t="1710246465905" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="4107" width="20" height="20"
                         data-spm-anchor-id="a313x.search_index.0.i4.5b253a81w0866s">
                      <path d="M457.728 692.48a21.824 21.824 0 0 1-12.928-9.728 21.952 21.952 0 0 1-2.24-15.936l100.864-401.408a20.928 20.928 0 0 1 20.544-15.936c16.512 3.584 23.232 14.656 20.544 26.112L483.648 675.2a20.928 20.928 0 0 1-20.544 16l-5.376 1.28z m-101.76-47.36a21.504 21.504 0 0 1-15.168-6.208L190.72 485.824a20.672 20.672 0 0 1 0-29.184l150.016-153.152a21.12 21.12 0 0 1 30.784 0 21.568 21.568 0 0 1 0 30.976L261.76 445.12a38.336 38.336 0 0 0 0 53.12l111.168 110.144a21.568 21.568 0 0 1 0 28.8 24.64 24.64 0 0 1-16.96 7.936z m315.136 0a21.504 21.504 0 0 1-16-7.04 21.568 21.568 0 0 1 0-30.976l109.76-108.864a35.2 35.2 0 0 0 0-51.392l-110.72-111.488a20.672 20.672 0 0 1 0-28.8 23.68 23.68 0 0 1 17.92-9.28c5.632 0 11.072 2.24 15.104 6.208l150.016 153.152a21.12 21.12 0 0 1 0 29.184l-149.568 153.088a21.504 21.504 0 0 1-16.512 6.208zM64 128h896v704H64V128z m64 64v576h768V192H128z m64 704h640v64H192v-64z" fill="#000000" p-id="4108"/></svg>
                  </div>
                </div>
              </template>
            </el-popover>
          </div>
          <div v-for="(item, index) in tasks" :key="index"
               :class="`rowSC step-menu-item ${activeIndex === item.value ? ' active' : ''}`"
               @click="onMenuClick(item.value)">
            <span>{{ $t(item.title) }}</span>
          </div>
          <el-empty v-if="!diagnoseStatus" :description="$t('NoDiagnosisTask')">
            <el-upload
                :class="diagnoseStatus ? 'upload-disabled' : ''"
                action=""
                :disabled="diagnoseStatus"
                accept=".json,.jsonl"
                :http-request="uploadFile"
            >
              <template #trigger>
                <el-button size="default" :disabled="diagnoseStatus" type="primary">{{ $t('SelectExceptionFile') }}</el-button>
              </template>
            </el-upload>
          </el-empty>
        </div>
      </el-aside>
      <el-main style="height: 100%;">
        <div class="columnSS chat-container">
          <div ref="messagesScrollDiv" class="messages-container">
            <template v-if="activeIndex === 'roleAssignment'">
              <DiagnosisOneChat
                  :sender="diagnoseData?.roleAssignment?.sender"
                  :can-fold="false"
                  :messages="diagnoseData?.roleAssignment?.messages"/>
            </template>
            <template v-if="activeIndex === 'expertDiagnosis'">
              <div v-for="(item, index) in diagnoseData?.expertDiagnosis?.experts" :key="index">
                <DiagnosisOneChat
                    v-if="item.messages.length > 0"
                    :sender="item.name"
                    :can-fold="true"
                    :is-fold="item.complete && index !== (diagnoseData?.expertDiagnosis?.experts.length - 1)"
                    :messages="item.messages"/>
              </div>
            </template>
            <template v-if="activeIndex === 'groupDiscussion'">
              <DiagnosisGroupChat :messages="diagnoseData?.groupDiscussion?.messages"/>
            </template>
            <template v-if="activeIndex === 'reportGeneration'">
              <DiagnosisOneChat :sender="diagnoseData?.reportGeneration?.sender" :can-fold="false" :messages="diagnoseData?.reportGeneration?.messages"/>
            </template>
            <template v-if="activeIndex === 'reportDemonstration'">
              <div style="padding: 20px; background: #f2f2f2; border-radius: 8px" v-html="reportContent" />
            </template>
          </div>
          <BottomInputContainer
              style="width: 100%"
              :placeholder="diagnoseData.placeholder || ''"
              :input-disabled="!(diagnoseStatus && needInput)"
              @send-click="onSendClick">
            <template v-if="diagnoseStatus" #right>
              <div style="margin-left: 10px; cursor: pointer" @click="onStopDiagnoseClick">
                <svg t="1709818874544" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="4540" width="32" height="32">
                  <path
                      d="M512 1024a512 512 0 1 1 512-512 512 512 0 0 1-512 512z m0-896a384 384 0 1 0 384 384A384 384 0 0 0 512 128z m128 576h-256a64 64 0 0 1-64-64v-256a64 64 0 0 1 64-64h256a64 64 0 0 1 64 64v256a64 64 0 0 1-64 64z"
                      fill="#d81e06" p-id="4541"/>
                </svg>
              </div>
            </template>
          </BottomInputContainer>
        </div>
      </el-main>
    </el-container>

  </div>
</template>

<script setup lang="ts" name="Index">

import {
  diagnoseSerializationOutputReq,
  diagnoseStatusReq,
  diagnoseStopDiagnoseReq,
  diagnoseTerminalOutputReq,
  diagnoseUserFeedbackReq,
  runDiagnoseReq
} from "@/api/diagnose.js";
import BottomInputContainer from "@/components/BottomInputContainer.vue";
import DiagnosisOneChat from '@/components/DiagnosisOneChat.vue'
import DiagnosisGroupChat from '@/components/DiagnosisGroupChat.vue'
import marked from '@/utils/markdownConfig.js'
import {ElMessage, ElMessageBox} from "element-plus";
import {ref} from 'vue'
const router = useRouter()

const tasks = [
  {"value": "roleAssignment", "title": "RoleAssignment"},
  {"value": "expertDiagnosis", "title": "ExpertDiagnosis"},
  {"value": "groupDiscussion", "title": "GroupDiscussion"},
  {"value": "reportGeneration", "title": "ReportGeneration"},
  {"value": "reportDemonstration", "title": "ReportDemonstration"}
]

const activeIndex: Ref<string> = ref('roleAssignment')

const diagnoseTerminalOutput: Ref<string> = ref('')

const diagnoseStatus: Ref<Boolean> = ref(false)

const messagesScrollDiv = ref<HTMLElement | null>(null);

const outputScrollDiv = ref<HTMLElement | null>(null);

const diagnoseData: Ref<Object> = ref({})

const diagnoseTerminalOutputHtml = computed(() => {
  return marked.parse(diagnoseTerminalOutput.value)
})

const reportContent = computed(() => {
  return marked.parse(diagnoseData.value?.reportDemonstration?.report || '')
})

const needInput = computed(() => {
  return diagnoseData.value?.needInput || false
})

watch(() => diagnoseTerminalOutputHtml.value, () => {
  nextTick(() => {
    if (outputScrollDiv.value) {
      outputScrollDiv.value.scrollTop = outputScrollDiv.value.scrollHeight;
    }
  })
})

watch(() => diagnoseData.value, () => {
  nextTick(() => {
    if (diagnoseData.value['currentTask']) {
      activeIndex.value = diagnoseData.value['currentTask']
    }
  })
})

onMounted(() => {
  nextTick(() => {
    getDiagnoseStatus()
  })
});

const onMenuClick = (index) => {
  activeIndex.value = index
}

const getDiagnoseStatus = async () => {
  diagnoseStatusReq().then(res => {
    diagnoseStatus.value = res.data.is_alive;
    if (diagnoseStatus.value) {
      setTimeout(() => {
        getDiagnoseStatus()
      }, 5000)
    }
  }).finally(() => {
    getDiagnoseTerminalOutput()
    getDiagnoseSerializationOutputReq()
  })
}

const getDiagnoseTerminalOutput = async () => {
  diagnoseTerminalOutputReq().then(res => {
    diagnoseTerminalOutput.value = res.data.output;
  })
}


const getDiagnoseSerializationOutputReq = async () => {
  diagnoseSerializationOutputReq().then(res => {
    diagnoseData.value = res.data
  })
}

const uploadFile = (files) => {
  runDiagnoseReq(files.file).then(() => {
    ElMessage({
      type: 'success',
      message: 'Upload Successfully!',
    })
    getDiagnoseStatus()
  }).finally(() => {

  })
}

const onSendClick = (value) => {
  if (value === '') {
    ElMessage.error('Please input something')
    return
  }
  diagnoseTerminalOutput.value += `\nUser: ${value}`
  diagnoseUserFeedbackReq(value).then(() => {
    ElMessage({
      type: 'success',
      message: 'Feedback Successfully!',
    })
  }).finally(() => {
  })
}

const onStopDiagnoseClick = () => {

  ElMessageBox.confirm(
      'Will stop diagnose. Continue?',
      'Warning',
      {
        confirmButtonText: 'Stop',
        cancelButtonText: 'Cancel',
        type: 'warning',
      }
  )
      .then(() => {
        diagnoseStopDiagnoseReq().then(() => {
          ElMessage({
            type: 'success',
            message: 'Stop Successfully!',
          })
        }).finally(() => {
          getDiagnoseStatus()
        })
      }).catch(() => {
  })
}

</script>

<style lang="scss">
//.el-upload {
//  --el-upload-dragger-padding-horizontal: 0px !important;
//}
//
//.el-upload-dragger {
//  border: none;
//  background: transparent;;
//}

.upload-disabled {
  background: #e2e2e2;
}

.output {
  font-size: 14px;
  color: #ffffff;
  white-space: pre-wrap;
  word-break: break-all;

  h1, h2, h3, h4, h5, h6 {
    font-size: 14px;
    color: #ffffff;
  }
}
</style>

<style lang="scss" scoped>

.step-menu-item {
  width: 100%;
  margin-bottom: 10px;
  padding: 10px 10px;
  border-radius: 10px;
  cursor: pointer;
  border: 1px solid var(--el-color-primary);
  color: var(--el-color-primary);
  background: var(--el-color-primary-light-9);

  .el-icon {
    margin-right: 10px;
  }

  &:hover {
    background: var(--el-color-primary);
    color: white;
  }

  &.active {
    background: var(--el-color-primary);
    color: white;
  }
}

.chat-container {
  height: 100%;
  overflow: hidden;

  .messages-container {
    height: calc(100vh - 60px);
    width: 100%;
    flex-shrink: 1;
    background: transparent;
    overflow-y: auto;
    padding-bottom: 20px;
  }
}


.blinking-dot {
  width: 10px;
  height: 10px;
  background-color: #67C23A;
  border-radius: 50%;
  position: absolute;
  top: 1px;
  right: -4px;
  animation: breathe 1.5s infinite;
}

@keyframes breathe {
  0%, 100% {
    opacity: 1;
    transform: scale(1);
  }
  50% {
    opacity: 0.5;
    transform: scale(0.8);
  }
}

.diagnose-output-container {
  position: relative;
  height: calc(100vh - 110px);
  width: calc(100% - 20px);
  flex-shrink: 1;
  color: #ffffff;
  overflow-y: auto;
  border-radius: 10px;
  margin: 10px;
}

</style>
