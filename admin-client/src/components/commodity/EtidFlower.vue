<template>
    <div class="edit-flower">
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>编辑商品</el-breadcrumb-item>
        </el-breadcrumb>

        <el-card class="box-card">

            <el-form inline ref="flowerRef" :rules="flowerinfoRules" :model="flowerinfo" label-width="100px">
                <el-form-item label="鲜花标题：" prop="title">
                    <el-input v-model="flowerinfo.title" placeholder="请输入鲜花标题"></el-input>
                </el-form-item>
                <el-form-item label="花语：" prop="flowerLanguage">
                    <el-input v-model="flowerinfo.flowerLanguage" placeholder="请输入花语"></el-input>
                </el-form-item>
                <el-form-item label="上下架：">
                    <el-switch
                            @change="flowerStatusChange"
                            v-model="status"
                            active-color="#13ce66"
                            inactive-color="#ff4949"
                            active-text="上架"
                            inactive-text="下架">
                    </el-switch>
                </el-form-item>
                <el-form-item label="材料：" prop="material">
                    <el-input v-model="flowerinfo.material" placeholder="请输入鲜花材料"></el-input>
                </el-form-item>
                <el-form-item label="包装：" prop="pack">
                    <el-input v-model="flowerinfo.pack" placeholder="请输入包装材料"></el-input>
                </el-form-item>
                <el-form-item label="存放仓库：">
                    <el-select size="small" v-model="flowerinfo.warehouseId" placeholder="请选择">
                        <el-option
                                v-for="item in warehouses"
                                :key="item.id"
                                :label="item.wareName"
                                v-model="item.id">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="入库数量：">
                    <el-input-number v-model="flowerinfo.stock" :min="0" :step="1"></el-input-number>
                </el-form-item>


                <el-form-item label="销售价格：" label-width="110px" prop="price">
                    <el-input-number v-model="flowerinfo.price" :precision="2" :min="0" :step="1.00"></el-input-number>
                </el-form-item>


                <el-form-item label="鲜花用途：" label-width="110px">
                    <el-select size="small" v-model="flowerinfo.purposeId" placeholder="请选择">
                        <el-option
                                v-for="item in flowerPurpose"
                                :key="item.purposeId"
                                :label="item.purposeName"
                                :value="item.purposeId">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="鲜花正图：">
                    <div class="upload" @click="$refs.uploadRef.click()">
                        <img v-if="imageUrl!=''" :src="imageUrl" class="cover">
                        <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                        <input type="file" ref="uploadRef" style="display: none" @change="inputFileChange">
                    </div>
                </el-form-item>

            </el-form>
            <!--上传详细图片表单-->
            <el-form inline label-width="100px">
                <el-form-item label="详细图片：">
                    <div class="pictureInfo">
                        <div class="picture" v-for="(item,index) in picturepreviews" :key="index">
                            <img :src="item.pictureSrc" class="cover">
                        </div>
                        <div @click="$refs.pictureInfo.click()" class="upload">
                            <i class="el-icon-plus avatar-uploader-icon"></i>
                        </div>
                        <input name="picture" id="picture" style="display: none"
                               @change="flowerPictureChange" type="file" ref="pictureInfo"/>
                    </div>
                </el-form-item>
            </el-form>
            <!--产品说明-->
            <el-form label-width="100px">
                <el-form-item label="产品说明：">
                    <el-input type="textarea" v-model="flowerinfo.details" placeholder="请输入产品说明" rows="4"></el-input>
                </el-form-item>
            </el-form>
            <div style="text-align: center">
                <el-button type="primary" @click="editFflowerinfo">编辑</el-button>
                <el-button @click="$router.push('/flower/list')">取消</el-button>
            </div>
        </el-card>

    </div>
</template>

<script>
    export default {
        data() {
            return {
                flowerId: null,
                pictures: [],
                picturepreviews: [],
                imageUrl: '',
                warehouses: [],  // 仓库列表
                flowerPurpose: [], // 鲜花用途
                flowerinfo: {price: 0, stock: 0, status: 2},
                status: false,
                flowerinfoRules: {
                    title: [{required: true, message: '请输入鲜花标题', trigger: 'blur'}],
                    price: [{required: true, message: '请输入销售价格', trigger: 'blur'}],
                    material: [{required: true, message: '请输入鲜花材料', trigger: 'blur'}],
                    flowerLanguage: [{required: true, message: '请输入花语', trigger: 'blur'}],
                    pack: [{required: true, message: '请输入包装材料', trigger: 'blur'}],
                }

            }
        },
        methods: {
            // 加载 详细图片
            loadFlowerPicture() {
                this.$axios({
                    url: '/api/flower/picture/list',
                    params: {flowerId: this.flowerId}
                }).then(resp => {
                    if (resp.data.code == 200) {
                        this.picturepreviews = resp.data.data
                    }
                })
            },
            // 加载 鲜花用途
            loadFlowerPurpose() {
                this.$axios({
                    url: '/api/purpose/list',
                }).then(result => {
                    if (result.data.code === 200) {
                        this.flowerPurpose = result.data.data
                    } else {
                        alert(result.data.message)
                    }
                })
            },
            // 加载鲜花信息
            loadFlowerInfo(flowerId) {
                this.$axios({
                    url: '/api/flower/info',
                    params: {flowerId}
                }).then(resp => {
                    if (resp.data.code === 200) {
                        this.flowerinfo = resp.data.data;
                        this.imageUrl = this.flowerinfo.coverSrc
                        this.status = this.flowerinfo.status === 1 ? true : false
                    } else {
                        alert(resp.data.message)
                    }
                })

            },
            // 加载仓库
            loadFlowerWarehouse() {
                this.$axios({
                    url: '/api/warehouse/list',
                }).then(result => {
                    if (result.data.code === 200) {
                        this.warehouses = result.data.data
                    } else {
                        alert(result.data.message)
                    }
                })
            },
            // 上下架
            flowerStatusChange() {
                this.flowerinfo.status = this.status ? 1 : 2

            },
            //封面图发生改变时
            inputFileChange() {
                this.flowerinfo.cover = this.$refs.uploadRef.files[0]
                this.imageUrl = URL.createObjectURL(this.$refs.uploadRef.files[0])  //创建一个虚拟路径,用来显示

            },
            // 详情图片
            // 图片预览
            flowerPictureChange() {
                let bbb = {
                    pictureSrc: URL.createObjectURL(this.$refs.pictureInfo.files[0])
                }
                this.picturepreviews.push(bbb)
            },
            // 修改鲜花信息
            editFflowerinfo() {
                // 表单验证
                this.$refs.flowerRef.validate(b => {
                    if (b == false) {
                        return this.$message.warning("必填项不能为空")
                    }
                    // 验证通过，开始修改
                    // 上架、下架
                    this.flowerinfo.status = this.status ? 1 : 2
                    let formData = new FormData();
                    if (this.$refs.uploadRef.files[0] !== undefined) {
                        formData.append("cover", this.flowerinfo.cover)
                        this.flowerinfo.cover = undefined
                    }
                    formData.append("flowerinfo", JSON.stringify(this.flowerinfo))

                    this.$axios({
                        url: '/api/flower/info/edit',
                        method: 'post',
                        data: formData
                    }).then(result => {
                        if (result.data.code == 200) {
                            if (this.pictures.length > 0) {
                                this.uploadPictures(result.data.data.flowerId)
                                this.loadFlowerPurpose()
                            }
                            this.$message.success(result.data.message)
                            // 刷新
                            this.loadFlowerInfo(this.flowerId)
                            this.loadFlowerPicture()
                            window.scrollTo(0, 0); // 滚动到顶部啊
                            this.$refs.uploadRef.value = ''

                        } else {
                            alert(result.data.message)
                        }

                    })
                })
            },
            // 上传详细图片
            uploadPictures(flowerId) {

                if (this.pictures.length == 0) {
                    return
                }
                this.pictures.forEach(s => {
                    let formData = new FormData();
                    formData.append("picture", s)
                    formData.append("flowerId", flowerId)
                    this.$axios({
                        url: '/api/upload/picture',
                        method: 'post',
                        data: formData
                    }).then(() => {
                        this.pictures = []
                        this.picturepreviews = []
                    })
                })
            },

        },
        created() {
            this.flowerId = this.$route.params.flowerId
            this.loadFlowerPurpose()
            this.loadFlowerWarehouse()
            this.loadFlowerPicture()
            this.loadFlowerInfo(Number.parseInt(this.flowerId))
        }
    }
</script>

<style scoped>
    .box-card {
        margin-top: 10px;
    }

    /*   鲜花正图样式 */
    .upload {
        border: 1px dashed #d9d9d9;
        border-radius: 6px;
        cursor: pointer;
        position: relative;
        overflow: hidden;
    }

    .upload:hover {
        border-color: #409EFF;
    }

    .avatar-uploader-icon {
        font-size: 28px;
        color: #8c939d;
        width: 128px;
        height: 128px;
        line-height: 128px;
        text-align: center;
    }

    .cover {
        width: 128px;
        height: 128px;
        display: block;
    }

    /*    详细图片 */
    .pictureInfo {
        display: flex;
    }

    .pictureInfo .picture {
        margin-right: 1px;
        padding: 1px;
        border: 1px solid #d9d9d9;
        border-radius: 6px;
    }
</style>