<template>
	<section>
		<!--搜索框-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item label="姓名：">
					<el-input v-model="filters.name" placeholder="姓名"></el-input>
				</el-form-item>
				<el-form-item label="手机号码：">
					<el-input v-model="filters.name" placeholder="手机号码"></el-input>
				</el-form-item>
				<el-form-item label="会员编号：">
					<el-input v-model="filters.name" placeholder="会员编号"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getUsers">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleAdd">新增</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="users" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
			<el-table-column type="selection" width="55">
			</el-table-column>
			<el-table-column type="index" label="会员编号" width="120" sortable>
			</el-table-column>
			<el-table-column prop="name" label="姓名" width="120">
			</el-table-column>
			<el-table-column prop="sex" label="手机" width="180" :formatter="formatSex">
			</el-table-column>
			<el-table-column prop="age" label="会员等级" width="120">
			</el-table-column>
			<el-table-column prop="birth" label="积分" width="120">
			</el-table-column>
			<el-table-column prop="age" label="余额" min-width="120">
			</el-table-column>
			<el-table-column prop="age" label="总消费" min-width="120">
			</el-table-column>
			<el-table-column label="操作" width="300">
				<template slot-scope="scope">
					<el-button size="small" @click="dialogTableVisible = true">记录</el-button>
					<el-button size="small" @click="shopingVisible = true">消费</el-button>
					<el-button size="small" @click="pushMoneyVisible = true">充值</el-button>
					<el-button size="small" @click="changeCountVisible = true">兑换</el-button>
					<!-- <el-button size="small" @click="handleEdit(scope.$index, scope.row)">兑换</el-button> -->
					<!-- <el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button> -->
				</template>
			</el-table-column>
		</el-table>

		<!-- 记录弹窗 -->
		<el-dialog title="记录" :visible.sync="dialogTableVisible" width=30%>
			<el-form :inline="true" :model="filters">
				<el-form-item label="记录类型：">
					<el-select v-model="value" placeholder="请选择">
						<el-option
						v-for="item in options"
						:key="item.value"
						:label="item.label"
						:value="item.value">
						</el-option>
					</el-select>
				</el-form-item>
			</el-form>
			<el-table :data="gridData">
				<el-table-column property="date" label="日期"></el-table-column>
				<el-table-column property="name" label="操作"></el-table-column>
			</el-table>
		</el-dialog>

		<!--消费弹窗-->
		<el-dialog title="消费" v-model="shopingVisible" :close-on-click-modal="false" width=30%>
			<el-form :model="shopingForm" label-width="100px" :rules="shopingFormRules" ref="shopingForm">
				<el-form-item label="消费金额：" prop="name">
					<el-input v-model="shopingForm.name" auto-complete="off"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="shopingVisible = false">取消</el-button>
				<el-button type="primary" @click.native="shopingSubmit" :loading="shopingLoading">提交</el-button>
			</div>
		</el-dialog>

		<!--充值弹窗-->
		<el-dialog title="充值" v-model="pushMoneyVisible" :close-on-click-modal="false" width=30%>
			<el-form :model="pushMoneyForm" label-width="100px" :rules="pushMoneyFormRules" ref="pushMoneyForm">
				<el-form-item label="充值金额：" prop="name">
					<el-input v-model="pushMoneyForm.name" auto-complete="off"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="pushMoneyVisible = false">取消</el-button>
				<el-button type="primary" @click.native="pushMoneySubmit" :loading="pushMoneyLoading">提交</el-button>
			</div>
		</el-dialog>

		<!--兑换弹窗-->
		<el-dialog title="兑换积分" v-model="changeCountVisible" :close-on-click-modal="false" width=30%>
			<el-form :model="changeCountForm" label-width="100px" :rules="changeCountFormRules" ref="changeCountForm">
				<el-form-item label="兑换积分：" prop="name">
					<el-input v-model="changeCountForm.name" auto-complete="off"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="changeCountVisible = false">取消</el-button>
				<el-button type="primary" @click.native="changeCountSubmit" :loading="changeCountLoading">提交</el-button>
			</div>
		</el-dialog>

		<!--工具条-->
		<el-col :span="24" class="toolbar">
			<el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
			<el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="20" :total="total" style="float:right;">
			</el-pagination>
		</el-col>

		<!--新增界面-->
		<el-dialog title="新增" v-model="addFormVisible" :close-on-click-modal="false">
			<el-form :model="addForm" label-width="100px" :rules="addFormRules" ref="addForm">
				<el-form-item label="姓名" prop="name">
					<el-input v-model="addForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="电话" prop="phone">
					<el-input v-model="addForm.phone" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="介绍人" prop="jsr">
					<el-input v-model="addForm.jsr" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="首冲" prop="sc">
					<el-input v-model="addForm.sc" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="会员级别：">
					<el-select v-model="addForm.hyjb" placeholder="请选择">
						<el-option
						v-for="item in options"
						:key="item.value"
						:label="item.label"
						:value="item.value">
						</el-option>
					</el-select>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="addFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
			</div>
		</el-dialog>
	</section>
</template>

<script>
	import util from '../../common/js/util'
	//import NProgress from 'nprogress'
	import { getUserListPage, removeUser, batchRemoveUser, editUser, addUser } from '../../api/api';

	export default {
		data() {
			return {
				gridData: [{
					date: '2016-05-02',
					name: '王小虎',
					}, {
					date: '2016-05-04',
					name: '王小虎',
					}, {
					date: '2016-05-01',
					name: '王小虎',
					}, {
					date: '2016-05-03',
					name: '王小虎',
				}],
				dialogTableVisible: false,
				options: [{
						value: '1',
						label: '消费记录'
					}, {
						value: '2',
						label: '积分兑换'
				}],
				value: '1',
				shopingVisible: false,//编辑界面是否显示
				shopingLoading: false,
				shopingFormRules: {
					count: [
						{ required: true, message: '请输入消费金额', trigger: 'blur' }
					]
				},
				//消费界面数据
				shopingForm: {
					count: ''
				},
				pushMoneyVisible: false,//编辑界面是否显示
				pushMoneyLoading: false,
				pushMoneyFormRules: {
					count: [
						{ required: true, message: '请输入充值金额', trigger: 'blur' }
					]
				},
				//充值界面数据
				pushMoneyForm: {
					count: ''
				},
				changeCountVisible: false,//编辑界面是否显示
				changeCountLoading: false,
				changeCountFormRules: {
					count: [
						{ required: true, message: '请输入兑换积分', trigger: 'blur' }
					]
				},
				//积分界面数据
				changeCountForm: {
					count: ''
				},

				filters: {
					name: ''
				},
				users: [],
				total: 0,
				page: 1,
				listLoading: false,
				sels: [],//列表选中列

				addFormVisible: false,//新增界面是否显示
				addLoading: false,
				addFormRules: {
					name: [
						// { required: true, message: '请输入姓名', trigger: 'blur' }
					]
				},
				//新增界面数据
				addForm: {
					name: '',
					phone: '',
					jsr: '',
					sc: '',
					hyjb: ''
				}

			}
		},
		methods: {
			//性别显示转换
			formatSex: function (row, column) {
				return row.sex == 1 ? '男' : row.sex == 0 ? '女' : '未知';
			},
			handleCurrentChange(val) {
				this.page = val;
				this.getUsers();
			},
			//获取用户列表
			getUsers() {
				let para = {
					page: this.page,
					name: this.filters.name
				};
				this.listLoading = true;
				//NProgress.start();
				getUserListPage(para).then((res) => {
					this.total = res.data.total;
					this.users = res.data.users;
					this.listLoading = false;
					//NProgress.done();
				});
			},
			//删除
			handleDel: function (index, row) {
				this.$confirm('确认删除该记录吗?', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					//NProgress.start();
					let para = { id: row.id };
					removeUser(para).then((res) => {
						this.listLoading = false;
						//NProgress.done();
						this.$message({
							message: '删除成功',
							type: 'success'
						});
						this.getUsers();
					});
				}).catch(() => {

				});
			},
			//显示新增界面
			handleAdd: function () {
				this.addFormVisible = true;
				this.addForm = {
					name: '',
					phone: '',
					jsr: '',
					sc: '',
					hyjb: ''
				};
			},
			//编辑
			editSubmit: function () {
				this.$refs.editForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.editLoading = true;
							//NProgress.start();
							let para = Object.assign({}, this.editForm);
							para.birth = (!para.birth || para.birth == '') ? '' : util.formatDate.format(new Date(para.birth), 'yyyy-MM-dd');
							editUser(para).then((res) => {
								this.editLoading = false;
								//NProgress.done();
								this.$message({
									message: '提交成功',
									type: 'success'
								});
								this.$refs['editForm'].resetFields();
								this.editFormVisible = false;
								this.getUsers();
							});
						});
					}
				});
			},
			//新增
			addSubmit: function () {
				this.$refs.addForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.addLoading = true;
							//NProgress.start();
							let para = Object.assign({}, this.addForm);
							para.birth = (!para.birth || para.birth == '') ? '' : util.formatDate.format(new Date(para.birth), 'yyyy-MM-dd');
							addUser(para).then((res) => {
								this.addLoading = false;
								//NProgress.done();
								this.$message({
									message: '提交成功',
									type: 'success'
								});
								this.$refs['addForm'].resetFields();
								this.addFormVisible = false;
								this.getUsers();
							});
						});
					}
				});
			},
			selsChange: function (sels) {
				this.sels = sels;
			},
			//批量删除
			batchRemove: function () {
				var ids = this.sels.map(item => item.id).toString();
				this.$confirm('确认删除选中记录吗？', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					//NProgress.start();
					let para = { ids: ids };
					batchRemoveUser(para).then((res) => {
						this.listLoading = false;
						//NProgress.done();
						this.$message({
							message: '删除成功',
							type: 'success'
						});
						this.getUsers();
					});
				}).catch(() => {

				});
			}
		},
		mounted() {
			this.getUsers();
		}
	}

</script>

<style scoped>

</style>