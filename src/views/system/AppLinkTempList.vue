<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="4" :sm="6">
          <a-form-item >
            <a-input placeholder="模板名称" v-model="queryParam.tempName"></a-input>
          </a-form-item>
          </a-col>
          <a-col :md="4" :sm="6">
            <a-form-item >
              <a-input placeholder="小说名称" v-model="queryParam.novelName"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->
    
    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>

      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">
        
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无此图片</span>
          <img v-else :src="getImgView(text)" alt="图片不存在" style="width:100%;height:90px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无此文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="uploadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>

              </a-menu-item>

            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>

    <appLinkTemp-modal ref="modalForm" @ok="modalFormOk"></appLinkTemp-modal>
  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import AppLinkTempModal from  './modules/AppLinkTempModal'
  import { axios } from '@/utils/request'

  export default {
    name: "AppLinkTempList",
    mixins:[JeecgListMixin],
    components: {
      AppLinkTempModal
    },
    data () {
      return {
        description: '推广模板管理页面',
        // 表头
        columns: [

          {
            title:'模板名称',
            align:"center",
            dataIndex: 'tempName'
          },
          {
            title:'方案标题',
            align:"center",
            dataIndex: 'contentTitle'

          },

          {
            title:'小说名称',
            align:"center",
            dataIndex: 'novelName'
          },
          {
            title:'备注',
            align:"center",
            dataIndex: 'note'
          },
          {
            title:'类型',
            align:"center",
            dataIndex: 'type',
            customRender:function (t,r,index) {
              return t==1?"小说":"漫画";
            }

          },
          {
            title:'创建人',
            align:"center",
            dataIndex: 'createBy',

          },

          {
            title:'创建日期',
            align:"center",
            dataIndex: 'createTime',
            sorter:'true',
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
        url: {
          list: "/app/appLinkTemp/list",
          delete: "/app/appLinkTemp/delete",
          deleteBatch: "/app/appLinkTemp/deleteBatch",
          exportXlsUrl: "/app/appLinkTemp/exportXls",
          importExcelUrl: "app/appLinkTemp/importExcel",
        },
        dictOptions:{
        } 
      }
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      }
    },
    methods: {

       
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>