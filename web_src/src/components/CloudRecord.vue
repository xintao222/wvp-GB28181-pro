<template>
	<div id="app">
		<el-container>
			<el-header>
				<uiHeader></uiHeader>
			</el-header>
			<el-main>
        <div style="background-color: #FFFFFF; margin-bottom: 1rem; position: relative; padding: 0.5rem; text-align: left;">
          <span style="font-size: 1rem; font-weight: bold;">云端录像</span>
          <div style="position: absolute; right: 5rem; top: 0.3rem;">
            节点选择:
            <el-select size="mini" @change="chooseMediaChange" style="width: 16rem; margin-right: 1rem;" v-model="mediaServerId" placeholder="请选择" :disabled="recordDetail">
            <el-option
              v-for="item in mediaServerList"
              :key="item.id"
              :label="item.id"
              :value="item.id">
            </el-option>
          </el-select>
          </div>
          <div style="position: absolute; right: 1rem; top: 0.3rem;">
            <el-button v-if="!recordDetail" icon="el-icon-refresh-right" circle size="mini" :loading="loading" @click="getRecordList()"></el-button>
            <el-button v-if="recordDetail" icon="el-icon-arrow-left" circle size="mini" @click="backToList()"></el-button>
          </div>
        </div>
        <div v-if="!recordDetail">

          <!--设备列表-->
          <el-table :data="recordList" border style="width: 100%" :height="winHeight">
            <el-table-column prop="app" label="应用名" align="center">
            </el-table-column>
            <el-table-column prop="stream" label="流ID" align="center">
            </el-table-column>
            <el-table-column prop="time" label="时间" align="center">
            </el-table-column>
            <el-table-column label="操作" width="360" align="center" fixed="right">
              <template slot-scope="scope">
                <el-button-group>
                  <el-button size="mini" icon="el-icon-video-camera-solid" type="primary" @click="showRecordDetail(scope.row)">查看</el-button>
<!--                  <el-button size="mini" icon="el-icon-delete" type="danger"  @click="deleteRecord(scope.row)">删除</el-button>-->
                </el-button-group>
              </template>
            </el-table-column>
          </el-table>
          <el-pagination
            style="float: right"
            @size-change="handleSizeChange"
            @current-change="currentChange"
            :current-page="currentPage"
            :page-size="count"
            :page-sizes="[15, 25, 35, 50]"
            layout="total, sizes, prev, pager, next"
            :total="total">
          </el-pagination>
        </div>
        <cloud-record-detail ref="cloudRecordDetail" v-if="recordDetail" :recordFile="chooseRecord" :mediaServerId="mediaServerId" ></cloud-record-detail>
			</el-main>
		</el-container>
	</div>
</template>

<script>
	import uiHeader from './UiHeader.vue'
	import cloudRecordDetail from './CloudRecordDetail.vue'
  import MediaServer from './service/MediaServer'
	export default {
		name: 'app',
		components: {
			uiHeader, cloudRecordDetail
		},
		data() {
			return {
        mediaServerList: [], // 滅体节点列表
        mediaServerId: null, // 媒体服务
        recordList: [], // 设备列表
        chooseRecord: null, // 媒体服务

        updateLooper: 0, //数据刷新轮训标志
        winHeight: window.innerHeight - 250,
        currentPage:1,
        count:15,
        total:0,
        loading: false,
        mediaServerObj : new MediaServer(),
        recordDetail: false

			};
		},
		computed: {

		},
		mounted() {
			this.initData();
		},
		destroyed() {
			// this.$destroy('videojs');
		},
		methods: {
			initData: function() {
			  // 获取媒体节点列表
			  this.getMediaServerList();
			  // this.getRecordList();
			},
      currentChange: function(val){
        this.currentPage = val;
        this.getRecordList();
      },
      handleSizeChange: function(val){
        this.count = val;
        this.getRecordList();
      },
      getMediaServerList: function (){
        let that = this;
        that.mediaServerObj.getOnlineMediaServerList((data)=>{
          that.mediaServerList = data.data;
          if (that.mediaServerList.length > 0) {
            that.mediaServerId = that.mediaServerList[0].id
            that.getRecordList();
          }
        })
      },
      getRecordList: function (){
        let that = this;
        this.$axios({
          method: 'get',
          url:`/record_proxy/${that.mediaServerId}/api/record/list`,
          params: {
            page: that.currentPage,
            count: that.count
          }
        }).then(function (res) {
          console.log(res)
          that.total = res.data.data.total;
          that.recordList = res.data.data.list;
          that.loading = false;
        }).catch(function (error) {
          console.log(error);
          that.loading = false;
        });
      },
      backToList(){
			  this.recordDetail= false;
      },
      chooseMediaChange(val){
          console.log(val)
          this.total = 0;
          this.recordList = [];
          this.getRecordList();
      },
      showRecordDetail(row){
        this.recordDetail = true;
        this.chooseRecord = row;
        // 查询是否存在录像
        // this.$axios({
        //   method: 'delete',
        //   url:`/record_proxy/api/record/delete`,
        //   params: {
        //     page: this.currentPage,
        //     count: this.count
        //   }
        // }).then((res) => {
        //   console.log(res)
        //   this.total = res.data.data.total;
        //   this.recordList = res.data.data.list;
        // }).catch(function (error) {
        //   console.log(error);
        // });

      },
      deleteRecord(){
			  // TODO
        let that = this;
        this.$axios({
          method: 'delete',
          url:`/record_proxy/api/record/delete`,
          params: {
            page: that.currentPage,
            count: that.count
          }
        }).then(function (res) {
          console.log(res)
          that.total = res.data.data.total;
          that.recordList = res.data.data.list;
        }).catch(function (error) {
          console.log(error);
        });
      }


		}
	};
</script>

<style>

</style>
