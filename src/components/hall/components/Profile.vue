<template>
		<div class="account-profile">
				<div class="profile-form">
						<Form ref="profile"
						      class="form-body"
						      :model="profileData"
						      :rules="profileValid"
						>
								<!--头像-->
								<FormItem>
										<div class="avatar-item">
												<Upload :action="upload"
												        class="upload-class"
												        v-show="!profileData.headImg"
												        :show-upload-list="false"
												        name="file"
												        :format="['jpg','jpeg','png','JPEG','JPG','PNG']"
												        :max-size="1024"
												        :on-format-error="handleFormatError"
												        :on-exceeded-size="handleMaxSize"
												        :on-success="handleSuccess"
												        :headers="headers"
												        :on-error="handlerError">
														<div class="upload-button">
																<Icon type="ios-log-out" size="18" color="#fff"
																      class="pointer"
																      style="font-weight: bold;transform: rotate(-90deg)"></Icon>
														</div>
												</Upload>
												<div class="image-position" v-show="profileData.headImg">
														<img :src="profileData.headImg||defaultAvatar" alt="avatar"/>
														<div class="demo-upload-list-cover">
																<Icon type="ios-trash-outline" size="24" @click.native="handleRemove"></Icon>
														</div>
												</div>
										</div>
								
								</FormItem>
								<!--mos 账号-->
								<FormItem label="MOS 账号">
										<div class="mos-item">
												<span class="area-code">{{profileData.areaCode}}</span>
												<div class="phone-input">
														<span class="phone-span">{{profileData.phone}}</span>
														<span class="change-span" @click="onChangePhone">更换</span>
												</div>
										</div>
								</FormItem>
								
								<!--昵称-->
								<FormItem label="姓名/昵称">
										<i-input v-model="profileData.name"
										         :maxlength="16" placeholder="请输入您的姓名/昵称"></i-input>
								</FormItem>
								
								<FormItem label="性别">
										<i-select v-model="profileData.sex">
												<Option v-for="item in sexList" :value="item.value" :key="item.value"> {{item.name}}</Option>
										</i-select>
								</FormItem>
								<FormItem label="微信号" prop="weChat">
										<i-input v-model="profileData.weChat"
										         :maxlength="20" placeholder="请输入微信号/手机号"></i-input>
								</FormItem>
								<FormItem label="QQ号" prop="qq">
										<i-input v-model="profileData.qq"
										         prop="qq"
										         :maxlength="15" placeholder="请输入您的QQ号码"></i-input>
								</FormItem>
						</Form>
						<Button style="height: 48px;margin-top: 60px;" long type="primary"
						        :loading="isLoading"
						        @click="onSaveProfile">保存
						</Button>
						
						<p style="text-align: center;margin-top: 24px;">
								<span @click="onCancelAccount" class="pointer">注销账号</span>
						</p>
						
						<Spin size="large" fix v-if="isLoading"></Spin>
				</div>
				<!--spin loading-->
		
		</div>
</template>

<script>
	import defaultAvatar from '../../../assets/images/default-avatar.png';
	
	export default {
		name: "Profile",
		data() {
			const checkWeChat = (rule, value, callback) => {
				if (value) {
					if (value.length > 5 && value.length < 21) {
						callback();
					} else callback('请输入合法的微信账号，6-20位');
				} else callback();
			};
			const checkQQ = (rule, value, callback) => {
				if (value) {
					if (/\d{5,15}/.test(value)) {
						callback();
					} else callback('请输入合法的QQ账号，5-15位数字');
				} else callback();
			};
			return {
				isEdit: false,
				isLoading: false,
				defaultAvatar,
				upload: '/file-server/api/resources/files/upload',
				headers: {
					token: localStorage.getItem('token'),
				},
				profileData: {
					areaCode: "+86",
					phone: "",
					name: "",
					sex: "",
					weChat: "",
					qq: '',
					headImg: ""
				},
				sexList: [
					{name: "男", value: "M"},
					{name: "女", value: "F"},
					{name: "保密", value: "N"},
				],
				profileValid: {
					weChat: [
						{required: false, validator: checkWeChat, trigger: "blur"}],
					qq: [
						{required: false, validator: checkQQ, trigger: "blur"}
					],
				}
			};
		},
		created() {
			this.getProfileDetail();
		},
		methods: {
			handleFormatError() {
				this.$Message.error('仅支持：JPEG/JPG/PNG 格式');
			},
			
			handleMaxSize() {
				this.$Message.error('最大支持1MB');
			},
			handlerError() {
				this.$Message.error('上传失败');
			},
			handleSuccess(res) {
				if (res && res.code === 10000) {
					this.$Message.success(res.message);
					this.$set(this.profileData, 'headImg', res.data);
				}
			},
			handleRemove() {
				this.$set(this.profileData, 'headImg', '');
			},
			getProfileDetail(isCheck) {
				const id = localStorage.getItem('userId');
				if (!isCheck) this.isLoading = true;
				this.$ajaxGet('/user-server/platform/employee/detail', {
					id
				})
					.then(res => {
						if (res&&res.code === 10000) {
							this.profileData = res.data || {};
							localStorage.setItem('name', res.data.name || '');
							localStorage.setItem('headImg', res.data.headImg || '');
							this.isLoading = false;
							this.$store.dispatch("setAvatar");
							this.$store.dispatch("setUsername");
						}
					});
			},
			onChangePhone() {
				this.$emit('whatComponent', {
					name: "ChangePhone", accountObj: {
						phone: this.profileData.phone,
						areaCode: this.profileData.areaCode
					}
				});
			},
			onSaveProfile() {
				this.$refs['profile'].validate(valid => {
					if (valid) this.updateProfile();
				});
				
			},
			updateProfile() {
				const {name = "", sex = "", qq = "", weChat = "", headImg = ""} = this.profileData;
				const id = localStorage.getItem('userId') || "";
				this.isLoading = true;
				this.$ajaxPost('/user-server/platform/employee/updateBaseInfo', {
					name, sex, qq, weChat, headImg, id
				})
					.then(res => {
						if (res&&res.code === 10000) {
							this.$Message.success(res.message || '操作成功');
							this.getProfileDetail(1);
						}
					});
			},
			onCancelAccount() {
				this.$emit('whatComponent', {name: 'CancelAccount', accountObj: this.profileData});
			}
		}
	};
</script>

<style scoped lang="scss">
		.account-profile {
				background: #fff;
				padding-bottom: 48px;
		}
		
		.upload-class {
				width: 100%;
				height: 100%;
				text-align: center;
				line-height: 72px;
				background: rgba(23, 23, 37, 0.8);
				
		}
		
		.profile-form {
				position: relative;
				width: 340px;
				margin: 0 auto;
		}
		
		.avatar-item {
				width: 80px;
				height: 80px;
				margin: 0 auto;
				border: 4px solid #fff;
				border-radius: 50%;
				overflow: hidden;
				
				.image-position {
						position: relative;
						width: 72px;
						height: 72px;
						text-align: center;
						border: 1px solid #E1E7EB;
						cursor: pointer;
						background: rgba(0, 0, 0, .2);
						line-height: 72px;
						
						img {
								height: 72px;
								width: 72px;
								border-radius: 50%;
						}
						
						&:hover .demo-upload-list-cover {
								position: absolute;
								background: rgba(0, 0, 0, 0.18);
								color: #fff;
								left: 0;
								top: 0;
								width: 80px;
								height: 80px;
						}
				}
		}
</style>
