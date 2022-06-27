```vue
<template>
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
            <a-descriptions
              bordered
              size="small"
              :column="{ xl: 3, md: 2, sm: 1 }">
              <a-descriptions-item label="业务宗号">
                {{ form.caseId }}
              </a-descriptions-item>
              <a-descriptions-item label="业务名称">
                {{ bizName }}
              </a-descriptions-item>
              <a-descriptions-item label="业务类型">
                {{ bizName }}
              </a-descriptions-item>
              <a-descriptions-item label="申请时间">
                {{ applyTime }}
              </a-descriptions-item>
              <a-descriptions-item label="企业名称">
                {{ form.developerName}}
              </a-descriptions-item>
              <a-descriptions-item label="项目名称">
                {{form.projectName }}
              </a-descriptions-item>
            </a-descriptions>
          </a-collapse-panel>

          <a-collapse-panel
            header="楼栋信息"
            key="2"
            style="margin-top: 16px">
          </a-collapse-panel>
        </a-collapse>
      </a-tab-pane>
      <a-tab-pane key="3" tab="材料列表">
        <a-card>
          <sw-files :evidences="evidences.fileData" isRead/>
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
      <approval></approval>
    </FixedBottom>
  </div>
</template>
<script lang="ts" src="./index.ts"></script>
<style scoped>
:deep(td .location) {
  text-align: left !important;
}
:deep(.red-tip input) {
  color: red !important;
  font-weight: bolder !important;
}
</style>

```

