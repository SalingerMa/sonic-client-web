<script setup>
/*
 *  Copyright (C) [SonicCloudOrg] Sonic Project
 *
 *  Licensed under the Apache License, Version 2.0 (the "License");
 *  you may not use this file except in compliance with the License.
 *  You may obtain a copy of the License at
 *
 *         http://www.apache.org/licenses/LICENSE-2.0
 *
 *  Unless required by applicable law or agreed to in writing, software
 *  distributed under the License is distributed on an "AS IS" BASIS,
 *  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 *  See the License for the specific language governing permissions and
 *  limitations under the License.
 *
 */
import { ref, onMounted, watch, onUnmounted, onBeforeMount } from 'vue';
import { useRouter } from 'vue-router';
import { useI18n } from 'vue-i18n';
import { Operation } from '@element-plus/icons';
import { useStore } from 'vuex';
import { ElMessage } from 'element-plus';
import useClipboard from 'vue-clipboard3';
import ColorImg from '@/components/ColorImg.vue';
import DeviceDes from '../components/DeviceDes.vue';
import Pageable from '../components/Pageable.vue';
import axios from '../http/axios';

const store = useStore();
const { t: $t } = useI18n();

const { toClipboard } = useClipboard();
const img = import.meta.globEager('./../assets/img/*');
const router = useRouter();
const currentTab = ref('device');
const timer = ref(null);
const avgTem = ref(0);
const checkAllAndroid = ref(false);
const isAllAndroid = ref(false);
const checkAlliOS = ref(false);
const isAlliOS = ref(false);
const checkAllHarmony = ref(false);
const isAllHarmony = ref(false);
const checkAllMan = ref(false);
const isAllMan = ref(false);
const checkAllCpu = ref(false);
const isAllCpu = ref(false);
const checkAllSize = ref(false);
const isAllSize = ref(false);
const checkAllAgent = ref(false);
const isAllAgent = ref(false);
const checkAllStatus = ref(false);
const isAllStatus = ref(false);
const pageData = ref({});
const pageSize = ref(12);
const currDevicesPage = ref(1);
const checkMan = ref([]);
const name = ref('');
const androidSystem = ref([]);
const iOSSystem = ref([]);
const harmonySystem = ref([]);
const cpu = ref([]);
const status = ref([]);
const size = ref([]);
const agentIds = ref([]);
const cpus = ref([]);
const sizes = ref([]);
const isFlush = ref('0');
const androidSystemVersion = ref([5, 6, 7, 8, 9, 10, 11, 12, 13]);
const iOSSystemVersion = ref([9, 10, 11, 12, 13, 14, 15, 16]);
const harmonySystemVersion = ref([1, 2, 3]);
const manufacturer = ref([
  'APPLE',
  'HUAWEI',
  'Xiaomi',
  'OPPO',
  'vivo',
  'samsung',
  'HONOR',
  'Meizu',
  'Google',
  'OnePlus',
  '360',
  'blackshark',
  'Yulong',
  'BBK',
  'nubia',
  'realme',
  'deltainno',
  'GIONEE',
  'HTC',
  'LGE',
  'Sony',
  'motorola',
  'asus',
  'Lenovo',
  'HMD Global',
  'Teclast',
]);
const statusList = ref([
  {
    value: 'ONLINE',
  },
  {
    value: 'DEBUGGING',
  },
  {
    value: 'TESTING',
  },
  {
    value: 'DISCONNECTED',
  },
  {
    value: 'OFFLINE',
  },
  {
    value: 'UNAUTHORIZED',
  },
  {
    value: 'ERROR',
  },
]);
const agentList = ref([]);
const formatHighTemp = (value) => {
  return `${value} ℃`;
};
const robotList = [
  { name: '钉钉群机器人', value: 1, img: 'DingTalk' },
  { name: '企业微信机器人', value: 2, img: 'WeChat' },
  { name: '飞书群机器人', value: 3, img: 'FeiShu' },
  { name: '友空间机器人', value: 4, img: 'You' },
];
const dialogAgent = ref(false);
const updateAgentForm = ref(null);
watch(dialogAgent, (newValue, oldValue) => {
  if (!newValue) {
    agent.value = {
      id: 0,
      name: '',
      highTemp: 45,
      highTempTime: 15,
      robotSecret: '',
      robotToken: '',
      robotType: 1,
    };
  }
});
const agent = ref({
  id: 0,
  name: '',
  highTemp: 45,
  highTempTime: 15,
  robotSecret: '',
  robotToken: '',
  robotType: 1,
});
const editAgent = async (
  id,
  name,
  highTemp,
  highTempTime,
  robotType,
  robotToken,
  robotSecret
) => {
  agent.value = {
    id,
    name,
    highTemp,
    highTempTime,
    robotType,
    robotToken,
    robotSecret,
  };
  await openAgent();
};
const openAgent = () => {
  dialogAgent.value = true;
};
const shutdownAgent = (id) => {
  axios.get('/transport/exchange/stop', { params: { id } }).then((resp) => {
    if (resp.code === 2000) {
      ElMessage.success({
        message: resp.message,
      });
    }
  });
};
const copy = (value) => {
  try {
    toClipboard(value);
    ElMessage.success({
      message: $t('dialog.copy.success'),
    });
  } catch (e) {
    ElMessage.error({
      message: $t('dialog.copy.fail'),
    });
  }
};
const updateAgent = () => {
  updateAgentForm.value.validate((valid) => {
    if (valid) {
      axios.put('/controller/agents/update', agent.value).then((resp) => {
        if (resp.code === 2000) {
          ElMessage.success({
            message: resp.message,
          });
          dialogAgent.value = false;
          getAllAgents();
        }
      });
    }
  });
};
const handleAndroid = (val) => {
  androidSystem.value = val ? androidSystemVersion.value : [];
  isAllAndroid.value = false;
  findAll();
};
const handleCheckedAndroid = (value) => {
  const checkedCount = value.length;
  checkAllAndroid.value = checkedCount === androidSystemVersion.value.length;
  isAllAndroid.value =
    checkedCount > 0 && checkedCount < androidSystemVersion.value.length;
  findAll();
};
const handleIOS = (val) => {
  iOSSystem.value = val ? iOSSystemVersion.value : [];
  isAlliOS.value = false;
  findAll();
};
const handleCheckedIOS = (value) => {
  const checkedCount = value.length;
  checkAlliOS.value = checkedCount === androidSystemVersion.value.length;
  isAlliOS.value =
    checkedCount > 0 && checkedCount < androidSystemVersion.value.length;
  findAll();
};
const handleHarmony = (val) => {
  harmonySystem.value = val ? harmonySystemVersion.value : [];
  isAllHarmony.value = false;
  findAll();
};
const handleCheckedHarmony = (value) => {
  const checkedCount = value.length;
  checkAllHarmony.value = checkedCount === harmonySystemVersion.value.length;
  isAllHarmony.value =
    checkedCount > 0 && checkedCount < harmonySystemVersion.value.length;
  findAll();
};
const handleMan = (val) => {
  checkMan.value = val ? manufacturer.value : [];
  isAllMan.value = false;
  findAll();
};
const handleCheckedMan = (value) => {
  const checkedCount = value.length;
  checkAllMan.value = checkedCount === manufacturer.value.length;
  isAllMan.value = checkedCount > 0 && checkedCount < manufacturer.value.length;
  findAll();
};
const handleCpu = (val) => {
  cpu.value = val ? cpus.value : [];
  isAllCpu.value = false;
  findAll();
};
const handleCheckedCpu = (value) => {
  const checkedCount = value.length;
  checkAllCpu.value = checkedCount === cpus.value.length;
  isAllCpu.value = checkedCount > 0 && checkedCount < cpus.value.length;
  findAll();
};
const handleSize = (val) => {
  size.value = val ? sizes.value : [];
  isAllSize.value = false;
  findAll();
};
const handleCheckedSize = (value) => {
  const checkedCount = value.length;
  checkAllSize.value = checkedCount === sizes.value.length;
  isAllSize.value = checkedCount > 0 && checkedCount < sizes.value.length;
  findAll();
};
const handleAgent = (val) => {
  agentIds.value = val
    ? agentList.value.map((item) => {
        return item.id;
      })
    : [];
  isAllAgent.value = false;
  findAll();
};
const handleCheckedAgent = (value) => {
  const checkedCount = value.length;
  checkAllAgent.value = checkedCount === agentList.value.length;
  isAllAgent.value = checkedCount > 0 && checkedCount < agentList.value.length;
  findAll();
};
const handleStatus = (val) => {
  status.value = val
    ? statusList.value.map((item) => {
        return item.value;
      })
    : [];
  isAllStatus.value = false;
  findAll();
};
const handleCheckedStatus = (value) => {
  const checkedCount = value.length;
  checkAllStatus.value = checkedCount === statusList.value.length;
  isAllStatus.value =
    checkedCount > 0 && checkedCount < statusList.value.length;
  findAll();
};
const handleInput = () => {
  findAll();
};
const findAll = (pageNum, pSize) => {
  axios
    .get('/controller/devices/list', {
      params: {
        page: pageNum || 1,
        pageSize: pSize || pageSize.value,
        androidVersion:
          androidSystem.value.length === 0 ? undefined : androidSystem.value,
        iOSVersion: iOSSystem.value.length === 0 ? undefined : iOSSystem.value,
        hmVersion:
          harmonySystem.value.length === 0 ? undefined : harmonySystem.value,
        manufacturer:
          checkMan.value.length === 0 ||
          checkMan.value.length === manufacturer.value.length
            ? undefined
            : checkMan.value,
        cpu:
          cpu.value.length === 0 || cpu.value.length === cpus.value.length
            ? undefined
            : cpu.value,
        size:
          size.value.length === 0 || size.value.length === sizes.value.length
            ? undefined
            : size.value,
        agentId:
          agentIds.value.length === 0 ||
          agentIds.value.length === agentList.value.length
            ? undefined
            : agentIds.value,
        status:
          status.value.length === 0 ||
          status.value.length === statusList.value.length
            ? undefined
            : status.value,
        deviceInfo: name.value.length > 0 ? name.value : undefined,
      },
    })
    .then((resp) => {
      if (resp.code === 2000) {
        pageData.value = resp.data;
      }
    })
    .catch(() => {
      clearInterval(timer.value);
    });
};
// 根据接口返回页数处理
const handleFindAll = (pageNum, pageSize) => {
  if (pageNum) {
    currDevicesPage.value = pageNum;
  }
  findAll(currDevicesPage.value, pageSize);
};
const getAllAgents = () => {
  axios
    .get('/controller/agents/list')
    .then((resp) => {
      agentList.value = resp.data;
    })
    .catch(() => {
      clearInterval(timer.value);
    });
};
const getImg = (name) => {
  let result;
  if (name === 'meizu') {
    name = 'Meizu';
  }
  if (name === 'LENOVO') {
    name = 'Lenovo';
  }
  try {
    result = img[`./../assets/img/${name}.jpg`].default;
  } catch {
    result = img['./../assets/img/unName.jpg'].default;
  }
  return result;
};
const getFilterOption = () => {
  axios
    .get('/controller/devices/getFilterOption')
    .then((resp) => {
      if (resp.code === 2000) {
        cpus.value = resp.data.cpu;
        sizes.value = resp.data.size;
      }
    })
    .catch(() => {
      clearInterval(timer.value);
    });
};
const findTemper = () => {
  axios
    .get('/controller/devices/findTemper')
    .then((resp) => {
      if (resp.code === 2000) {
        if (resp.data !== null) {
          avgTem.value = resp.data;
        }
      }
    })
    .catch(() => {
      clearInterval(timer.value);
    });
};
const switchTabs = (e) => {
  refreshNow('switch');
};
const refresh = () => {
  switch (currentTab.value) {
    case 'device':
      handleFindAll();
      findTemper();
      getAllAgents();
      break;
    case 'agent':
      getAllAgents();
      break;
  }
  clearInterval(laterTimer.value);
};
const refreshNow = (t) => {
  if (t !== 'switch') {
    localStorage.setItem('SonicIsRefresh', t);
  }
  if (t === '1' || t === 'switch') {
    if (t === '1') {
      timer.value = setInterval(refresh, 15000);
    }
    refresh();
  } else {
    clearInterval(timer.value);
  }
};
onBeforeMount(() => {
  isFlush.value = localStorage.getItem('SonicIsRefresh')
    ? localStorage.getItem('SonicIsRefresh')
    : '0';
});
const laterTimer = ref(null);
onMounted(() => {
  getFilterOption();
  refresh();
  if (isFlush.value === '1') {
    if (currentTab.value === 'device') {
      laterTimer.value = setInterval(refresh, 1500);
    }
    timer.value = setInterval(refresh, 15000);
  }
});
onUnmounted(() => {
  clearInterval(laterTimer.value);
  clearInterval(timer.value);
});
</script>

<template>
  <el-switch
    v-model="isFlush"
    class="refresh"
    active-value="1"
    inactive-value="0"
    style="float: right; margin-top: -10px"
    :active-text="$t('devices.refresh')"
    active-color="#13ce66"
    @change="refreshNow"
  />
  <el-tabs
    v-model="currentTab"
    style="margin-top: 20px"
    type="border-card"
    stretch
    @tab-click="switchTabs"
  >
    <el-tab-pane name="device" :label="$t('devices.deviceCenter')">
      <el-card>
        <el-collapse>
          <el-collapse-item name="1">
            <template #title>
              <el-input
                v-model="name"
                style="width: 440px"
                type="text"
                size="small"
                :placeholder="$t('devices.filter.placeholder')"
                maxlength="40"
                clearable
                @input="handleInput"
              >
              </el-input>

              <strong
                v-if="avgTem !== 0"
                style="
                  margin-left: 20px;
                  display: flex;
                  align-items: center;
                  font-size: 16px;
                  color: #909399;
                "
                >{{ $t('devices.avgTem') }}
                <div
                  :style="{
                    position: 'realtive',
                    display: 'flex',
                    'align-items': 'center',
                    color:
                      avgTem < 300
                        ? '#67C23A'
                        : avgTem < 350
                        ? '#E6A23C'
                        : '#F56C6C',
                  }"
                >
                  <ColorImg
                    :src="img['./../assets/img/tem.png'].default"
                    :width="20"
                    :height="20"
                    :color="
                      avgTem < 300
                        ? '#67C23A'
                        : avgTem < 350
                        ? '#E6A23C'
                        : '#F56C6C'
                    "
                  />
                  {{ (avgTem / 10).toFixed(1) + ' ℃' }}
                </div>
              </strong>
            </template>

            <el-form
              label-position="left"
              class="filter-table-expand"
              label-width="90px"
            >
              <el-form-item :label="$t('devices.filter.platform.ANDROID')">
                <div style="display: flex">
                  <el-checkbox
                    v-model="checkAllAndroid"
                    :label="$t('devices.filter.all')"
                    :indeterminate="isAllAndroid"
                    @change="handleAndroid"
                  ></el-checkbox>
                  <el-checkbox-group
                    v-model="androidSystem"
                    class="device-radio-p"
                    @change="handleCheckedAndroid"
                  >
                    <el-checkbox
                      v-for="version in androidSystemVersion"
                      :key="version"
                      :label="version"
                    >
                      {{ version }}
                    </el-checkbox>
                  </el-checkbox-group>
                </div>
              </el-form-item>
              <el-form-item :label="$t('devices.filter.platform.IOS')">
                <div style="display: flex">
                  <el-checkbox
                    v-model="checkAlliOS"
                    :label="$t('devices.filter.all')"
                    :indeterminate="isAlliOS"
                    @change="handleIOS"
                  ></el-checkbox>
                  <el-checkbox-group
                    v-model="iOSSystem"
                    class="device-radio-p"
                    @change="handleCheckedIOS"
                  >
                    <el-checkbox
                      v-for="version in iOSSystemVersion"
                      :key="version"
                      :label="version"
                    >
                      {{ version }}
                    </el-checkbox>
                  </el-checkbox-group>
                </div>
              </el-form-item>
              <el-form-item :label="$t('devices.filter.platform.HARMONY')">
                <div style="display: flex">
                  <el-checkbox
                    v-model="checkAllHarmony"
                    :label="$t('devices.filter.all')"
                    :indeterminate="isAllHarmony"
                    @change="handleHarmony"
                  ></el-checkbox>
                  <el-checkbox-group
                    v-model="harmonySystem"
                    class="device-radio-p"
                    @change="handleCheckedHarmony"
                  >
                    <el-checkbox
                      v-for="version in harmonySystemVersion"
                      :key="version"
                      :label="version"
                    >
                      {{ version }}
                    </el-checkbox>
                  </el-checkbox-group>
                </div>
              </el-form-item>
              <el-form-item :label="$t('devices.filter.manufacturer')">
                <div style="display: flex">
                  <el-checkbox
                    v-model="checkAllMan"
                    :label="$t('devices.filter.all')"
                    :indeterminate="isAllMan"
                    @change="handleMan"
                  ></el-checkbox>
                  <el-checkbox-group
                    v-model="checkMan"
                    class="device-radio-p"
                    @change="handleCheckedMan"
                  >
                    <el-checkbox
                      v-for="man in manufacturer"
                      :key="man"
                      :label="man"
                    >
                      {{ man }}
                    </el-checkbox>
                  </el-checkbox-group>
                </div>
              </el-form-item>
              <el-form-item :label="$t('devices.filter.cpu')">
                <div style="display: flex">
                  <el-checkbox
                    v-model="checkAllCpu"
                    :label="$t('devices.filter.all')"
                    :indeterminate="isAllCpu"
                    @change="handleCpu"
                  ></el-checkbox>
                  <el-checkbox-group
                    v-model="cpu"
                    class="device-radio-p"
                    @change="handleCheckedCpu"
                  >
                    <el-checkbox
                      v-for="c in cpus"
                      :key="c"
                      :label="c"
                    ></el-checkbox>
                  </el-checkbox-group>
                </div>
              </el-form-item>
              <el-form-item :label="$t('devices.filter.size')">
                <div style="display: flex">
                  <el-checkbox
                    v-model="checkAllSize"
                    :label="$t('devices.filter.all')"
                    :indeterminate="isAllSize"
                    @change="handleSize"
                  ></el-checkbox>
                  <el-checkbox-group
                    v-model="size"
                    class="device-radio-p"
                    @change="handleCheckedSize"
                  >
                    <el-checkbox
                      v-for="s in sizes"
                      :key="s"
                      :label="s"
                    ></el-checkbox>
                  </el-checkbox-group>
                </div>
              </el-form-item>
              <el-form-item :label="$t('devices.filter.agent')">
                <div style="display: flex">
                  <el-checkbox
                    v-model="checkAllAgent"
                    :label="$t('devices.filter.all')"
                    :indeterminate="isAllAgent"
                    @change="handleAgent"
                  ></el-checkbox>
                  <el-checkbox-group
                    v-model="agentIds"
                    class="device-radio-p"
                    @change="handleCheckedAgent"
                  >
                    <el-checkbox
                      v-for="agent in agentList"
                      :key="agent"
                      :label="agent.id"
                      >{{ agent.name }}
                    </el-checkbox>
                  </el-checkbox-group>
                </div>
              </el-form-item>
              <el-form-item :label="$t('devices.filter.status')">
                <div style="display: flex">
                  <el-checkbox
                    v-model="checkAllStatus"
                    :label="$t('devices.filter.all')"
                    :indeterminate="isAllStatus"
                    @change="handleStatus"
                  ></el-checkbox>
                  <el-checkbox-group
                    v-model="status"
                    class="device-radio-p"
                    @change="handleCheckedStatus"
                  >
                    <el-checkbox
                      v-for="statusDevice in statusList"
                      :key="statusDevice"
                      :label="statusDevice.value"
                      >{{ $t('devices.status.' + statusDevice.value) }}
                    </el-checkbox>
                  </el-checkbox-group>
                </div>
              </el-form-item>
            </el-form>
          </el-collapse-item>
        </el-collapse>

        <el-row :gutter="20">
          <el-col
            v-for="device in pageData.content"
            :key="device"
            :xs="12"
            :sm="12"
            :md="12"
            :lg="6"
            :xl="6"
            style="margin-top: 20px"
          >
            <DeviceDes
              :device="device"
              :agent-list="agentList"
              @flush="findAll"
            />
          </el-col>
        </el-row>
        <pageable
          :is-page-set="false"
          :total="pageData['totalElements']"
          :current-page="pageData['number'] + 1"
          :page-size="pageData['size']"
          @change="handleFindAll"
        ></pageable>
      </el-card>
    </el-tab-pane>
    <el-tab-pane name="agent" :label="$t('devices.agentCenter')">
      <el-button type="primary" size="mini" @click="openAgent">{{
        $t('agent.newAgent')
      }}</el-button>
      <el-table :data="agentList" border style="margin-top: 10px">
        <el-table-column
          prop="id"
          label="Agent ID"
          align="center"
          width="90"
        ></el-table-column>
        <el-table-column
          prop="name"
          label="Agent Name"
          header-align="center"
          show-overflow-tooltip
        ></el-table-column>
        <el-table-column
          prop="host"
          label="Host"
          align="center"
          show-overflow-tooltip
          width="150"
        ></el-table-column>
        <el-table-column
          prop="port"
          label="Port"
          align="center"
          width="90"
        ></el-table-column>
        <el-table-column
          prop="systemType"
          :label="$t('agent.system')"
          align="center"
          width="150"
        >
          <template #default="scope">
            <div
              style="
                display: flex;
                align-items: center;
                justify-content: center;
              "
            >
              {{ scope.row.systemType }}
              <img
                v-if="
                  scope.row.systemType !== 'unknown' &&
                  (scope.row.systemType.indexOf('Mac') !== -1 ||
                    scope.row.systemType.indexOf('Windows') !== -1 ||
                    scope.row.systemType.indexOf('Linux') !== -1)
                "
                style="margin-left: 10px"
                height="20"
                :src="
                  getImg(
                    scope.row.systemType.indexOf('Mac') !== -1
                      ? 'Mac'
                      : scope.row.systemType.indexOf('Linux') !== -1
                      ? 'Linux'
                      : 'Windows'
                  )
                "
              />
            </div>
          </template>
        </el-table-column>
        <el-table-column
          prop="version"
          :label="$t('agent.version')"
          align="center"
          width="150"
        ></el-table-column>
        <el-table-column
          prop="secretKey"
          label="Agent Key"
          align="center"
          width="150"
        >
          <template #default="scope">
            <el-button size="mini" @click="copy(scope.row.secretKey)">{{
              $t('agent.clickToCopy')
            }}</el-button>
          </template>
        </el-table-column>
        <el-table-column
          prop="status"
          :label="$t('agent.status.name')"
          align="center"
          width="90"
        >
          <template #default="scope">
            <el-tag v-if="scope.row.status === 1" size="small" type="success"
              >{{ $t('agent.status.online') }}
            </el-tag>
            <el-tag v-if="scope.row.status === 2" size="small" type="info"
              >{{ $t('agent.status.offline') }}
            </el-tag>
            <el-tag v-if="scope.row.status === 3" size="small" type="warning"
              >{{ $t('agent.status.s2ae') }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column
          :label="$t('agent.operation')"
          align="center"
          width="180"
        >
          <template #default="scope">
            <el-button
              size="mini"
              type="primary"
              @click="
                editAgent(
                  scope.row.id,
                  scope.row.name,
                  scope.row.highTemp,
                  scope.row.highTempTime,
                  scope.row.robotType,
                  scope.row.robotToken,
                  scope.row.robotSecret
                )
              "
            >
              {{ $t('common.edit') }}
            </el-button>
            <el-button
              size="mini"
              type="danger"
              :disabled="scope.row.status !== 1"
              @click="shutdownAgent(scope.row.id)"
            >
              {{ $t('agent.shutdown') }}
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-tab-pane>
  </el-tabs>
  <el-dialog
    v-model="dialogAgent"
    :title="$t('dialog.agentInfo')"
    width="600px"
  >
    <el-form
      v-if="dialogAgent"
      ref="updateAgentForm"
      :model="agent"
      size="small"
      class="demo-table-expand"
      label-width="90px"
      label-position="left"
    >
      <el-form-item
        prop="name"
        :label="$t('agent.edit.name')"
        :rules="{
          required: true,
          message: $t('agent.edit.rule'),
          trigger: 'blur',
        }"
      >
        <el-input
          v-model="agent.name"
          :placeholder="$t('agent.edit.namePlaceholder')"
        ></el-input>
      </el-form-item>
      <el-divider></el-divider>
      <el-alert
        style="margin-bottom: 10px"
        :title="$t('devices.hint')"
        type="info"
        show-icon
        :closable="false"
      >
        <template #default>
          <div>
            {{ $t('devices.adTemperature.temperature')
            }}<span style="color: #409eff">
              {{ $t('devices.adTemperature.height') }}</span
            >
            {{ $t('devices.adTemperature.onlyAd') }}
          </div>
          <div>
            {{ $t('devices.adTemperature.then')
            }}<span style="color: #e6a23c">
              {{ $t('devices.adTemperature.timeout') }}</span
            >
            {{ $t('devices.adTemperature.continuedTime')
            }}<span style="color: #409eff">
              {{ $t('devices.adTemperature.height') }}</span
            >
            {{ $t('devices.adTemperature.onlyAdNotice')
            }}<span style="color: #f56c6c">{{
              $t('devices.adTemperature.shutdown')
            }}</span
            >。
          </div>
        </template>
      </el-alert>
      <el-form-item prop="name" :label="$t('agent.edit.highTemp')">
        <el-slider
          v-model="agent.highTemp"
          :format-tooltip="formatHighTemp"
          show-input
          :max="80"
          :min="1"
        />
      </el-form-item>
      <el-form-item prop="name" :label="$t('agent.edit.highTempTime')">
        <el-input-number
          v-model="agent.highTempTime"
          show-input
          :max="120"
          :min="1"
        />
        <span style="margin-left: 10px">min</span>
      </el-form-item>
      <el-form-item :label="$t('robot.robotType')">
        <el-select
          v-model="agent.robotType"
          style="width: 100%"
          :placeholder="$t('robot.robotTypePlaceholder')"
        >
          <el-option
            v-for="item in robotList"
            :key="item.name"
            :value="item.value"
            :label="item.name"
            :disabled="item['disabled']"
          >
            <div style="display: flex; align-items: center">
              <el-avatar
                style="margin-right: 10px"
                :size="30"
                :src="getImg(item.img)"
                shape="square"
              ></el-avatar>
              {{ item.name }}
            </div>
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item :label="$t('robot.robotToken')" prop="robotToken">
        <el-input
          v-model="agent.robotToken"
          :placeholder="$t('robot.robotTokenPlaceholder')"
        ></el-input>
      </el-form-item>
      <el-form-item :label="$t('robot.robotSecret')" prop="robotSecret">
        <el-input
          v-model="agent.robotSecret"
          :placeholder="$t('robot.robotSecretPlaceholder')"
          type="password"
        ></el-input>
      </el-form-item>
    </el-form>
    <div style="text-align: center">
      <el-button size="small" type="primary" @click="updateAgent">{{
        $t('form.confirm')
      }}</el-button>
    </div>
  </el-dialog>
</template>
