<template>
  <div>
    <Card>
      <Form @submit.native.prevent label-postion="left" inline :label-width="100">
        <FormItem label="关键词:">
          <Input type="text" v-model="keyword"></Input>
        </FormItem>

        <FormItem label="文件类型:">
          <Select clearable v-model="status" style="width:200px">
            <Option value="PUBLISHED">已发布</Option>
            <Option value="RECYCLE">回收站</Option>
            <Option value="CHECK">审核中</Option>
            <Option value="NO">审核失败</Option>
          </Select>
        </FormItem>
        <FormItem>
          <Button type="primary" @click="handleList()">查询</Button>
        </FormItem>
      </Form>
      <Button type="primary" @click="toPage('写文章')">
        <Icon type="md-add" />写文章
      </Button>
    </Card>
    <Card class="b-card" style="margin:20px 0">
      <Table
        id="data"
        :columns="managerColumns"
        :data="articleData"
        align="center"
        stripe
        :loading="loading"
      >
        <div slot-scope="{row}" slot="status">
          <div v-if="row.status === 'PUBLISHED'">
            <Badge status="success" text="已发布" />
          </div>
          <div v-else-if="row.status === 'RECYCLE'">
            <Badge status="error" text="回收站" />
          </div>
          <div v-else-if="row.status === 'CHECK'">
            <Badge status="warning" text="审核中" />
          </div>
          <div v-else-if="row.status === 'NO'">
            <Badge color="yellow" text="审核失败" />
          </div>
        </div>
        <div slot-scope="{row}" slot="tagsTitle">
          <Tag color="error" v-for="(item,index) in row.tagsTitle" :key="index">{{item}}</Tag>
        </div>
        <div slot-scope="{ row, index}" slot="action">
          <!-- 文章发布状态 -->
          <div v-if="row.status === 'PUBLISHED'">
            <Button type="primary" style="margin-right: 5px" @click="eidtArticle(row)">编辑</Button>              
            <Poptip confirm title="确定要讲这篇文章放入回收站么?" @on-ok="editPostStatus(row,'RECYCLE')">
              <Button type="warning">回收站</Button>
            </Poptip>
            <Poptip confirm title="确定要删除这篇文章么?" @on-ok="deleteArticle(row)">
              <Button type="error">删除</Button>
            </Poptip>
          </div>
          <div v-else-if="row.status === 'RECYCLE' ">
            <Button type="primary" style="margin-right: 5px" @click="editPostStatus(row,'PUBLISHED')">还原</Button>
            <Poptip confirm title="确定要删除这篇文章么?" @on-ok="deleteArticle(row)">
              <Button type="error">删除</Button>
            </Poptip>
          </div>
          <div v-else-if="row.status === 'CHECK'">

            <Button type="primary" style="margin-right: 5px" @click="eidtArticle(row)">编辑</Button>
            <Button type="warning" disabled>回收站</Button>
          </div>
          <div v-else-if="row.status === 'NO'">
            <Button type="primary" style="margin-right: 5px" @click="eidtArticle(row)">编辑</Button> 
            <Button style="margin-right: 5px" type="error" @click="deleteArticle(row)">删除</Button>
          </div>
        </div>
      </Table>
      <!-- 分页列表 -->
      <div style="margin: 10px;overflow: hidden">
        <div style="float: right;">
          <Page
            :total="total"
            :current="pageNum"
            @on-change="changepage"
            @on-page-size-change="_nowPageSize"
            :page-size-opts="page"
            show-total
            show-sizer
            :page-size="pageSize"
          ></Page>
        </div>
      </div>
    </Card>
  </div>
</template>
<script>
import {
  Page,
  Card,
  Table,
  Badge,
  Poptip,
  Notice,
  Tag,
  Form,
  Message,
  FormItem,
  Select,
  Option,
  Button,
  Input,
  Icon,
  Modal
} from "view-design";
import { mapGetters, mapActions } from "vuex";
import router from "@/router";
import articleApi from "@/api/article";

export default {
  name: "articleManager",
  components: {
    Page,
    Card,
    Table,
    Badge,
    Poptip,
    Tag,
    Form,
    FormItem,
    Select,
    Option,
    Button,
    Input,
    Icon,
    Modal
  },
  data() {
    return {
      loading: false,
      pageSize: 5,
      pageNum: 1,
      page: [5, 10, 20, 50],
      keyword: "",
      status: ""
    };
  },
  methods: {
    ...mapActions(["getArticleList", "updateArticleStatus"]),
    deleteArticle(row) {
      console.log(row);
      articleApi.deleteArticle(row.id).then(response => {
        Notice.success({
          title: "删除文章成功！",
          desc: "注意，此操作不可逆"
        });
        this.queryArticleList();
      });
    },
    editPostStatus(row, status) {
      this.loading = true;
      this.updateArticleStatus({
        index: row._index,
        id: row.id,
        status: status
      }).then(response => {
        this.loading = false;
      });
    },
    eidtArticle(row) {
      Modal.confirm({
        title: "警告！",
        content: "您确定要更新文章？此操作会导致已发布文章变成审核状态！",
        onOk: () => {
          articleApi.getDetail(row.id).then(response => {
            const data = response.data;
            var article = data;
            this.$router.push({
              name: "写文章",
              params: article
            });
          });
        }
      });
    },
    handleList() {
      this.pageNum = 1;
      this.pageSize = 5;
      this.queryArticleList();
    },
    queryArticleList() {
      this.loading = true;
      let postParams = {};
      console.log("keyword:" + this.keyword);

      if (this.keyword != "") {
        postParams.keyword = this.keyword;
      }
      if (this.status != "") {
        postParams.status = this.status;
      }
      this.getArticleList({
        pageSize: this.pageSize,
        pageNum: this.pageNum,
        postParams: postParams
      }).then(response => {
        this.loading = false;
      });
    },
    //点击，切换页面
    changepage(index) {
      this.pageNum = index;
      this.queryArticleList();
    },

    //每页显示的数据条数
    _nowPageSize(size) {
      //实时获取当前需要显示的条数
      this.queryArticleList();
    },
    // 去往某个页面
    toPage(n) {
      this.$router.push({ name: n });
    }
  },
  computed: {
    ...mapGetters(["total", "managerColumns", "articleData"])
  },
  mounted() {
    this.queryArticleList();
  }
};
</script>


<style scoped>
.b-card {
  padding: 10px 10px;
}
.ivu-table-wrapper {
  position: static !important;
}
</style>

<style>
.ivu-poptip-confirm .ivu-poptip-body .ivu-icon {
  left: 8%;
}
#data .ivu-table th,
#data .ivu-table td {
  text-align: center;
}
</style>
