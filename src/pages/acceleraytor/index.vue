<template>
  <div class="accele-raytor container">
    <div class="page-head fs-container">
      <span class="title">Pools</span>
    </div>

    <div>
      <Input size="large" placeholder="Search by Pool name, Token symbol, Address">
        <Icon slot="prefix" type="search" />
      </Input>
      <div class="filter">
        <div>
          <h5>Access</h5>
          <RadioGroup v-model="filter.access">
            <RadioButton value="all">All</RadioButton>
            <RadioButton value="ray">RAY</RadioButton>
            <RadioButton value="community">Community</RadioButton>
          </RadioGroup>
        </div>
        <div>
          <h5>Status</h5>
          <RadioGroup v-model="filter.status">
            <RadioButton value="all">All</RadioButton>
            <RadioButton value="open">Open</RadioButton>
            <RadioButton value="upcoming">Upcoming</RadioButton>
            <RadioButton value="closed">Closed</RadioButton>
          </RadioGroup>
        </div>
        <div>
          <h5>My pools</h5>
          <RadioGroup v-model="filter.mine">
            <RadioButton value="all">All</RadioButton>
            <RadioButton value="joined">Joined</RadioButton>
          </RadioGroup>
        </div>
      </div>
      <Table
        :columns="columns"
        :data-source="pools"
        :pagination="false"
        row-key="idoId"
        :custom-row="
          (record) => {
            return {
              on: {
                click: () => {
                  $router.push(`/acceleraytor/${record.idoId}/`)
                }
              }
            }
          }
        "
      >
        <span slot="name" slot-scope="base" class="icon">
          <img :src="importIcon(`/coins/${base.symbol.toLowerCase()}.png`)" />
          <span> {{ base.symbol }}</span>
        </span>
        <span slot="price" slot-scope="info, record"> {{ info.price }} {{ record.quote.symbol }} </span>
        <span slot="access" slot-scope="info" class="access">
          <span v-if="info.isRayPool" class="ray">
            <span>RAY Pool</span>
          </span>
          <span v-else class="community"><span>Community Pool</span></span>
        </span>
        <span slot="raise" slot-scope="info, record"> {{ 50000 * info.price }} {{ record.quote.symbol }} </span>
        <span slot="progress"> 0.00% </span>
        <span slot="status" slot-scope="info" class="status">
          <span v-if="info.endTime < getUnixTs() / 1000" class="closed"> Closed </span>
          <span v-else-if="info.startTime < getUnixTs() / 1000" class="open"> Open </span>
          <span v-else class="upcoming"> Upcoming </span>
        </span>
      </Table>
    </div>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Watch } from 'nuxt-property-decorator'

import { Input, Icon, Radio, Table } from 'ant-design-vue'
import { filter } from 'lodash-es'

import { getUnixTs } from '@/utils'
import importIcon from '@/utils/import-icon'
import { IdoPool } from '@/utils/ido'

const RadioGroup = Radio.Group
const RadioButton = Radio.Button

@Component({
  head: {
    title: 'Raydium AcceleRaytor'
  },

  components: {
    Input,
    Icon,
    RadioGroup,
    RadioButton,
    Table
  },

  async asyncData({ $accessor }) {
    if (!$accessor.ido.initialized) {
      await $accessor.ido.requestInfos()
    }
  }
})
export default class AcceleRaytor extends Vue {
  filter = {
    access: 'all',
    status: 'all',
    mine: 'all'
  }

  pools: IdoPool[] = []

  @Watch('filter', { immediate: true, deep: true })
  onPersonChanged({ access }: { access: string; status: string; mine: string }) {
    const rules = {
      info: {}
    } as any

    if (access !== 'all') {
      if (access === 'ray') {
        rules.info.isRayPool = true
      } else {
        rules.info.isRayPool = false
      }
    }

    this.pools = filter(this.$accessor.ido.pools, rules)
  }

  columns = [
    {
      title: 'Pool name',
      dataIndex: 'base',
      key: 'name',
      scopedSlots: { customRender: 'name' }
    },
    {
      title: 'Price per token',
      dataIndex: 'info',
      key: 'price',
      scopedSlots: { customRender: 'price' },
      align: 'center'
    },
    {
      title: 'Access',
      dataIndex: 'info',
      key: 'access',
      scopedSlots: { customRender: 'access' },
      align: 'center'
    },
    {
      title: 'Total raise',
      dataIndex: 'info',
      key: 'raise',
      scopedSlots: { customRender: 'raise' },
      align: 'center'
    },
    {
      title: 'Progress',
      dataIndex: 'info',
      key: 'progress',
      scopedSlots: { customRender: 'progress' },
      align: 'center'
    },
    {
      title: 'Status',
      dataIndex: 'info',
      key: 'status',
      scopedSlots: { customRender: 'status' },
      align: 'center'
    }
  ]

  getUnixTs = getUnixTs
  importIcon = importIcon
}
</script>

<style lang="less" scoped>
.accele-raytor.container {
  max-width: 1200px;
}

.accele-raytor {
  .filter {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    margin-top: 16px;
    padding: 24px;
    background: #0f1429;
    color: #f1f1f2bf;

    h5 {
      color: #f1f1f2bf;
    }
  }

  .icon {
    display: flex;
    flex-direction: row;
    align-items: center;

    img {
      border-radius: 50%;
      width: 40px;
      height: 40px;
      margin-right: 8px;
    }
  }

  .access {
    .community,
    .ray {
      padding: 5px 1px;
      border-radius: 16px;

      span {
        padding: 4px 14px;
        border-radius: 16px;
      }
    }

    .community {
      background: #364371;
    }

    .ray {
      background: linear-gradient(245.22deg, #c200fb 7.97%, #3772ff 49.17%, #3773fe 49.17%, #5ac4be 92.1%);

      span {
        background: #1c274f;
        opacity: 0.9;
      }
    }
  }

  .status {
    .upcoming {
      padding: 4px 14px;
      border-radius: 16px;
      background: rgba(90, 196, 190, 0.2);
      border: 1px solid #5ac4be;
    }
  }
}
</style>

<style lang="less">
@import '../../styles/variables';

.has-error {
  input {
    border-color: #f5222d;

    &:active,
    &:focus,
    &:hover {
      border-color: #f5222d !important;
    }
  }
}

.accele-raytor {
  .ant-input-affix-wrapper {
    input {
      background-color: @bg-color;
    }
  }

  .ant-radio-button-wrapper {
    border: 1px solid transparent;
    box-shadow: none;
    border-radius: 4px;
    color: #f1f1f2bf;

    &:not(:first-child) {
      margin-left: 8px;
    }

    &:not(:first-child)::before {
      width: 0;
    }

    &:hover,
    &:active {
      border: 1px solid #f1f1f2 !important;
      box-shadow: none;
    }

    &:not(.ant-radio-button-wrapper-disabled):first-child {
      border: 1px solid transparent;
    }
  }

  .ant-radio-button-wrapper-checked {
    color: #0f1429;

    &:hover,
    &:active {
      color: #0f1429;
    }
  }

  .ant-table {
    background: #1c274f;
  }

  .ant-table-thead > tr > th {
    border-bottom: 1px solid #364371;
  }

  .ant-table-tbody > tr > td {
    padding: 24px 16px;
    border-bottom: 1px solid transparent;
  }

  .ant-table-tbody {
    &:not:first-child {
      border-bottom: 1px solid #364371;
    }
  }

  .ant-table-row:not(:last-child) {
    tr,
    td {
      border-bottom: 1px solid #364371;
    }
  }

  .ant-table-row:hover {
    cursor: pointer;
  }
}
</style>
