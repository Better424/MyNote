```vue
<template>
  <div v-if="!submitStatus">
    <a-card v-if="loading" :loading="true"></a-card>
    <div v-else>
      <a-tabs @change="onTabChange">
        <a-tab-pane key="1" tab="业务信息">
          <a-collapse
            expand-icon-position="right"
            class="marT16"
            v-model:activeKey="activeKey">
            <a-collapse-panel
              header="业务信息"
              key="1">
              <a-form
                ref="formRef"
                :model="form"
                :rules="rules">
                <a-row type="flex">
                  <a-col
                    :xxl="6"
                    :xl="6"
                    :xs="24">
                    <a-form-item label="业务宗号">
                      <a-input v-model:value="form.caseId" disabled />
                    </a-form-item>
                  </a-col>
                  <a-col
                    :xxl="6"
                    :xl="6"
                    :xs="24">
                    <a-form-item label="流程名称">
                      <a-input v-model:value="bizName" disabled />
                    </a-form-item>
                  </a-col>
                  <a-col
                    :xxl="6"
                    :xl="6"
                    :xs="24">
                    <a-form-item label="业务名称">
                      <a-input v-model:value="bizName" disabled />
                    </a-form-item>
                  </a-col>
                  <a-col
                    :xxl="6"
                    :xl="6"
                    :xs="24">
                    <a-form-item label="申请时间">
                      <a-input v-model:value="applyTime" disabled />
                    </a-form-item>
                  </a-col>
                  <a-col
                    :xxl="12"
                    :xl="12"
                    :xs="24">
                    <a-form-item label="企业名称">
                      <a-select
                        style="width: 100%"
                        v-model:value="form.developerId"
                        allowClear>
                        <a-select-option
                          v-for="item in selects.developers"
                          :value="item.id"
                          :key="item.id"
                          :label="item.name">
                          {{ item.name }}
                        </a-select-option>
                      </a-select>
                    </a-form-item>
                  </a-col>
                  <a-col
                    :xxl="12"
                    :xl="12"
                    :xs="24">
                    <a-form-item label="项目名称">
                      <a-select v-model:value="form.projectId">
                        <template v-if="form.developerId">
                          <a-select-option
                            v-for="item in selects.projects"
                            :value="item.projectId"
                            :key="item.id">
                            {{item.projectName}}
                          </a-select-option>
                        </template>
                        <a-select-option disabled v-else>
                          请先选择开发企业
                        </a-select-option>
                      </a-select>
                    </a-form-item>
                  </a-col>
                  <a-col
                    :xxl="12"
                    :xl="12"
                    :xs="24">
                    <a-form-item label="结算节点">
                      <a-select v-model:value="form.temp1">
                        <a-select-option
                          v-for="item in selects.node"
                          :value="item.projectId"
                          :key="item.id">
                          {{item.projectName}}
                        </a-select-option>
                      </a-select>
                    </a-form-item>
                  </a-col>
                  <a-col
                    :xxl="6"
                    :xl="6"
                    :xs="24">
                    <a-form-item label="结算比例">
                      <a-input
                        disabled
                        suffix="%"></a-input>
                    </a-form-item>
                  </a-col>
                  <a-col
                    :xxl="12"
                    :xl="12"
                    :xs="24">
                    <a-form-item label="预售资金监管账号">
                      <a-select v-model:value="form.temp1">
                        <template v-if="form.developerId">
                          <a-select-option
                            v-for="item in selects.node"
                            :value="item.projectId"
                            :key="item.id">
                            {{item.projectName}}
                          </a-select-option>
                        </template>
                        <a-select-option disabled v-else>
                          请先选择开发企业
                        </a-select-option>
                      </a-select>
                    </a-form-item>
                  </a-col>
                  <a-col
                    :xxl="12"
                    :xl="12"
                    :xs="24">
                    <a-form-item label="监管账户名称">
                      <a-input disabled></a-input>
                    </a-form-item>
                  </a-col>
                  <a-col
                    :xxl="12"
                    :xl="12"
                    :xs="24">
                    <a-form-item label="监管账户开户行">
                      <a-input disabled></a-input>
                    </a-form-item>
                  </a-col>
                  <a-col
                    :xxl="12"
                    :xl="12"
                    :xs="24">
                    <a-form-item label="本次结算总额">
                      <a-input disabled suffix="元"></a-input>
                    </a-form-item>
                  </a-col>
                </a-row>
              </a-form>
            </a-collapse-panel>

            <a-collapse-panel
              header="合同信息"
              key="2"
              style="margin-top: 16px">
              <div style="float: right">
                <a-button
                  type="primary"
                  @click="openChooseBuilding">
                  智能读取合同
                </a-button>
              </div>
              <a-table
                bordered
                :columns="contarctColumns"
                :data-source="contarctList"
                :loading ="contarctLoading"
                :scroll="{ x: 1400 }"
                size="small"
                :pagination="false"
                :customRow="customRow"
                rowKey="id"
                class="tableRowHeight"
                style="margin-top: 8px">
              </a-table>
            </a-collapse-panel>
          </a-collapse>
        </a-tab-pane>
        <a-tab-pane key="3" tab="材料列表">
          <a-card>
            <sw-files :evidences="evidences.fileData" />
          </a-card>
        </a-tab-pane>
        <a-tab-pane key="4" tab="审批情况">
          <a-card :loading="!workflowBpmn">
            <easy-designer
              ref="workflow"
              :treeNode="workflowBpmn"
              :nodeStatus="nodeStatus"/>
            <opinion></opinion>
          </a-card>
        </a-tab-pane>
      </a-tabs>
      <FixedBottom>
        <div class="btnbar" style="margin-top: 16px">
          <a-card>
            <button-group
              align="right"
              :option="buttonOption"
              :loadingKey="buttonLoadingKey">
              <template #extra>
                <a-input-group compact style="display: flex">
                  <a-dropdown :trigger="['click']">
                    <a-button>常用批语 <DownOutlined /></a-button>
                    <template #overlay>
                      <a-menu @click="onClick" v-if="selects.opinion.length">
                        <a-menu-item
                          v-for="item in selects.opinion"
                          :key="item.id"
                          :value="item.id">
                          <a>{{ item.content }}</a>
                        </a-menu-item>
                      </a-menu>
                      <a-menu v-else>
                        <a-menu-item disabled>暂无批语</a-menu-item>
                      </a-menu>
                    </template>
                  </a-dropdown>
                  <a-textarea
                    @change="changeOpinion"
                    :auto-size="{ minRows: 1, maxRows: 1 }"
                    v-model:value="opinion"
                    :class="opinionStatus ? 'wrong-opinions' : ''"
                    :placeholder="
                      '请输入审批意见'

                    "/>
                </a-input-group>
              </template>
            </button-group>
          </a-card>
        </div>
      </FixedBottom>
    </div>
  </div>
  <a-card v-else>
    <a-result status="success" title="业务申请成功">
      <template #extra>
        <a-button key="buy" @click="closePage()">关闭当前页</a-button>
      </template>
    </a-result>
  </a-card>
</template>
<script src="./index.ts">
</script>

```

