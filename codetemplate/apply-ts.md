```tsx
import {
  defineComponent, reactive, toRefs, inject, createVNode,
} from 'vue';
import { useRoute } from 'vue-router';
import { EasyDesigner, FixedBottom, ButtonGroup } from 'sw-cui';
import SwFiles from '@/components/swfiles/index.vue';
import Opinion from '@/components/approval/opinion.vue';
import CancelModal from '@/components/cancelmodal/cancelmodal.vue';
import { AppClient } from '@/libs/AppClient';
import { BpmInfoApis } from '@/libs/baseinfoapis';
import { DownOutlined, ExclamationCircleOutlined } from '@ant-design/icons-vue';
import { message, Modal } from 'ant-design-vue';
import { AppConsts } from '@/libs/appconst';
import _ from 'lodash';

function initEvidences() {
  const form = { externalKey: null, fileData: [] };
  return form;
}
export default defineComponent({
  components: {
    SwFiles,
    FixedBottom,
    CancelModal,
    Opinion,
    EasyDesigner,
    ButtonGroup,
    DownOutlined,
    ExclamationCircleOutlined,
  },
  setup() {
    const route = useRoute();
    const routeQuery: any = route.query;
    const close: any = inject('close');
    const dataMap = reactive({
      loading: true,
      submitStatus: false,
      activeKey: ['1', '2', '3'],
      selects: {
        projects: [],
        opinion: [],
        developers: [],
      } as any, // 下拉框集合
      form: {} as any,
      flowName: null,
      bizName: null,
      formRef: null,
    });
    const workFlow = reactive({
      workflowBpmn: null,
      nodeStatus: [] as any,
      evidences: initEvidences() as any,
      opinion: '',
      opinionStatus: false,
      changeOpinion() {
        if (!workFlow.opinion || workFlow.opinion.length === 0) {
          workFlow.opinionStatus = true;
        } else {
          workFlow.opinionStatus = false;
        }
      },
      chooseOpinion(e: any) {
        workFlow.opinion = e.content;
      },
      // 初始化电子档案证件材料，一般在页面初始化时需要调用到此方法。
      async initElectronicLicense() {
        const eFile = await AppClient.req(BpmInfoApis.getEfilePage, { externalKey: routeQuery.instanceKey });
        const findEfile = eFile.data.list;
        if (findEfile.length == 0) {
          const res = await AppClient.req(BpmInfoApis.getFlowEvidenceBizdefinitionKey, { bizKey: routeQuery.bizKey });
          res.data.forEach((element: any) => {
            element.isSystem = true;
            element.required = false;
          });
          workFlow.evidences.fileData = res.data;
          workFlow.evidences.externalKey = routeQuery.instanceKey;
        } else {
          findEfile[0].fileData = JSON.parse(findEfile[0].fileData!);
          workFlow.evidences = findEfile[0];
        }
      },
      // 新增或者更新电子档案证件材料，在业务保存或者提交是需要调用此方法
      async saveOrUpdateEvidences() {
        if (!workFlow.evidences.id && workFlow.evidences.id !== 0) {
          let submitEvidence = _.cloneDeep(workFlow.evidences);
          if (submitEvidence.fileData == undefined) {
            submitEvidence = initEvidences();
            submitEvidence.fileData = JSON.stringify(_.cloneDeep(workFlow.evidences));
          } else {
            submitEvidence.fileData = JSON.stringify(submitEvidence.fileData);
          }

          submitEvidence.externalKey = routeQuery.instanceKey;
          const res = await AppClient.req(BpmInfoApis.createeFile, submitEvidence);
          const eFile = await AppClient.req(BpmInfoApis.geteFileById, { id: res.data });
          eFile.data.fileData = JSON.parse(eFile.data.fileData!);
          workFlow.evidences = eFile.data.fileData;
        } else {
          const submitEvidence = _.cloneDeep(workFlow.evidences);
          submitEvidence.fileData = JSON.stringify(submitEvidence.fileData);
          submitEvidence.externalKey = routeQuery.instanceKey;
          await AppClient.req(BpmInfoApis.updateeFile, submitEvidence);
        }
      },
      // 校验电子档案是否有必传项。
      validateEvidences() {
        for (let x = 0; x < workFlow.evidences.fileData.length; x++) {
          const element = workFlow.evidences.fileData[x];
          if (element.required == true) {
            if (element.attachments == undefined || element.attachments.length == 0) {
              Modal.warning({
                title: '系统提示',
                content: `请上传《${element.name}》的必须件`,
                okText: '确认',
              });
              return;
            }
          }
        }
      },
      // 更新流程实例扩展表，在业务保存或者提交时需要调用到此方法，参数为业务表单
      async putInstanceext(_value: any) {
        const params: any = {
          instanceKey: routeQuery.instanceKey,
          caseId: routeQuery.caseId,
        };
        await AppClient.req(BpmInfoApis.putFlowInstanceext, params);
      },
      // 提交审批，在业务提交时需要调用到此方法。
      async workFlowsComplete() {
        const params:any = {
          completeCode: '1',
          opinion: workFlow.opinion,
        };
        await AppClient.req(BpmInfoApis.workFlowsComplete, params, { taskId: routeQuery.taskId });
      },
      async onTabChange(key: any) {
        if (key == 4 && !workFlow.workflowBpmn) {
          if (routeQuery.key != null) {
            const params = {
              pageNo: 1,
              pageSize: 10,
              key: routeQuery.key,
              groupKey: AppConsts.GROUP_KEY,
            };
            const repRes = await AppClient.req(BpmInfoApis.getFlowDefinitionHisId, params);
            const flowId = repRes.data.list[0].flowDefinitionHisId;
            const flowRes = await AppClient.req(BpmInfoApis.getFlowDefinitionHis, { id: flowId });
            workFlow.workflowBpmn = JSON.parse(flowRes.data.jsonResource);
            const nodeRes = await AppClient.req(BpmInfoApis.getNodeStatus, { instanceKey: routeQuery.instanceKey });
            workFlow.nodeStatus = nodeRes.data;
          }
        }
      },
    });
    const buttonConfig = reactive({
      cancelModal: null as any,
      buttonLoadingKey: '',
      buttonOption: [
        {
          key: 'saveOpinion',
          name: '保存意见',
          props: {
            type: 'primary',
            ghost: true,
          },
          disabled: () => !workFlow.opinion.length,
          event: async () => {
            try {
              buttonConfig.buttonLoadingKey = 'saveOpinion';
              await AppClient.req(BpmInfoApis.saveOpinion, { content: workFlow.opinion, taskId: routeQuery.taskId });
              message.success('意见保存成功！');
            } catch (error) {
              console.log(error);
            } finally {
              buttonConfig.buttonLoadingKey = '';
            }
          },
        },
        // {
        //   key: 'cancel',
        //   name: '注销',
        //   props: {
        //     type: 'danger',
        //     ghost: true,
        //   },
        //   disabled: () => dataMap.form.id == null,
        //   async event() {
        //     try {
        //       buttonConfig.buttonLoadingKey = 'cancel';
        //       buttonConfig.cancelModal.openModal();
        //     } finally {
        //       buttonConfig.buttonLoadingKey = '';
        //     }
        //   },
        // },
        {
          key: 'save',
          name: '保存',
          props: {
            type: 'primary',
            ghost: true,
          },
          async event() {
            try {
              buttonConfig.buttonLoadingKey = 'save';
              workFlow.saveOrUpdateEvidences();
              workFlow.putInstanceext(dataMap.form);
              message.success('保存成功');
            } finally {
              buttonConfig.buttonLoadingKey = '';
            }
          },
        },
        {
          key: 'submit',
          name: '提交',
          disabled: () => dataMap.form.id == null,
          props: {
            type: 'primary',
          },
          async event() {
            try {
              buttonConfig.buttonLoadingKey = 'submit';
              // await dataMap.formRef.validate();

              Modal.confirm({
                title: '请确认提交的信息是否有误，并提交',
                icon: createVNode(ExclamationCircleOutlined),
                async onOk() {
                  // await dataMap.formRef.validate();
                  await workFlow.saveOrUpdateEvidences();
                  await workFlow.workFlowsComplete();
                  dataMap.submitStatus = true;
                },
                onCancel() {},
              });
            } catch (error: any) {
              if (error.errorFields) {
                Modal.warning({
                  title: '系统提示',
                  content: '请先完善业务信息',
                  okText: '确认',
                });
              }
            } finally {
              buttonConfig.buttonLoadingKey = '';
            }
          },
        },
      ],
    });
    async function initPage() {
      try {
        dataMap.loading = true;
        await workFlow.initElectronicLicense();
        dataMap.flowName = routeQuery.flowName;
        dataMap.bizName = routeQuery.applyType;
      } finally {
        dataMap.loading = false;
      }
    }
    initPage();
    return {
      ...toRefs(dataMap),
      ...toRefs(workFlow),
      ...toRefs(buttonConfig),
    };
  },
});

```

