<template>
  <div>
    <BasicTable @register="registerTable">
      <template #toolbar>
        <a-button type="primary" @click="handleCreate"> 新增 </a-button>
      </template>
      <template #action="{ record }">
        <TableAction
          :actions="[
            {
              tooltip: '添加子分类',
              icon: 'ant-design:plus-outlined',
              onClick: handleCreateChild.bind(null, record),
            },
            {
              tooltip: '修改',
              icon: 'clarity:note-edit-line',
              onClick: handleEdit.bind(null, record),
            },
            {
              tooltip: '删除',
              icon: 'ant-design:delete-outlined',
              color: 'error',
              onClick: (e)=>{e.stopPropagation();},
              popConfirm: {
                title: '是否确认删除',
                confirm: handleDelete.bind(null, record),
              },
            },
          ]"
        />
      </template>
    </BasicTable>
    <GetDicModal @register="registerModal" @success="handleSuccess" />
  </div>
</template>
<script lang="ts">
  import { defineComponent } from 'vue';
  import { BasicTable, useTable, TableAction } from '/@/components/Table';
  import { getDicTypes, deleteByIds } from '/@/api/base/dicType';

  import { columns, searchFormSchema } from './dicType.data';
  import GetDicModal from './DicTypeModal.vue';
  import { useMessage } from '/@/hooks/web/useMessage';
  import { useModal } from '/@/components/Modal';
  const { createMessage } = useMessage();

  export default defineComponent({
    name: 'GetDic',
    components: { BasicTable, TableAction, GetDicModal },
    setup() {
      const [registerModal, { openModal, setModalProps }] = useModal();
      const [registerTable, { reload }] = useTable({
        title: '列表',
        api: getDicTypes,
        columns,
        formConfig: {
          labelWidth: 120,
          schemas: searchFormSchema,
          showAdvancedButton: false,
          showResetButton: false,
          autoSubmitOnEnter: true,
        },
        canColDrag: true,
        pagination:false,
        expandRowByClick: true,
        useSearchForm: true,
        bordered: true,
        showIndexColumn: false,
        actionColumn: {
          width: 120,
          title: '操作',
          dataIndex: 'action',
          slots: { customRender: 'action' },
          fixed: false,
        },
      });

      function handleCreate() {
        setModalProps({title: '新增字典分类'});
        openModal(true, {
          isUpdate: false,
        });
      }

      function handleEdit(record: Recordable, e) {
        e.stopPropagation();
        setModalProps({title: '修改字典分类'});
        openModal(true, {
          record,
          isUpdate: true,
        });
      }

      function handleCreateChild(record: Recordable, e) {
        e.stopPropagation();
        setModalProps({title: '新增【'+record.name+'】的子分类'});
        record = {pid: record.id};
        openModal(true, {
          record,
          isUpdate: true,
        });
      }

      function handleDelete(record: Recordable) {
        if(record.children&&record.children.length>0){
          createMessage.warning("有子节点，不能删除！")
          return;
        }
        deleteByIds([record.id]).then((res) => {
          reload();
        });
      }

      function doSearch() {
        reload();
      }

      function handleSuccess() {
        reload();
      }

      return {
        registerTable,
        registerModal,
        handleCreate,
        handleEdit,
        handleCreateChild,
        handleDelete,
        handleSuccess,
        doSearch,
      };
    },
  });
</script>
