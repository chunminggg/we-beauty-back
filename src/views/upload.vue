<style scoped>
.product {
    margin-top: 10px;
}

.myAlert {
    margin-top: 30px;
}

.imageUpload {
    margin-bottom: 30px
}

</style>

<template>
    <div class="content">
        <h2>产品发布</h2>
        <Input v-model="productNumber" placeholder="" class="product">
        <span slot="prepend">产品编号</span>
        </Input>
        <Input v-model="productName" placeholder="" class="product">
        <span slot="prepend">产品名称</span>
        </Input>
        <!--<Input v-model="productPlace" placeholder="" class="product">
        <span slot="prepend">出发城市</span>
        </Input>-->
        <Input v-model="productPrice" placeholder="" class="product">
        <span slot="prepend">产品价格</span>
        </Input>
        <Input v-model="productDes" placeholder="" class="product">
        <span slot="prepend">产品描述</span>
        </Input>
        <!--<Input v-model="productStartDate" placeholder="" class="product">
        <span slot="prepend">开始时间</span>
        </Input>-->
        <div class="product">
            <Button type="info" class="product" @click="priceSelect">价格添加</Button>
            <div class="priceTag">
            <Tag class="tagView" v-for="(item,index) in tagArray"  closable @on-close="handleClose(index)">{{item.date}} ￥{{item.price}}</Tag>
            </div>
        </div>
        <div class="product">
            <label>是否特价:</label>
            <i-switch v-model="isRecommend"></i-switch>
            <label>是否店长推荐:</label>
            <i-switch v-model="isSpecialPrice"></i-switch>
            <label>是否为特色项目</label>
            <i-switch v-model="isFollowTeam"></i-switch>
            <label>百安丽</label>
            <i-switch v-model="isFreeTravel"></i-switch>
        </div>
        <Select v-model="productTypeSelected" class="product" placeholder="请选择产品类型">
            <Option v-for="item in productTypes" :value="item.value" :key="item">{{ item.label }}</Option>
        </Select>
        <Alert class="myAlert" type="warning">产品详情滚动图（至少传一张)</Alert>
        <image-upload class="product imageUpload" @send-image="getImageArray" :uploadList="imageArray">

        </image-upload>
        <div v-for="(richItem, index) in richItems" class="myProduct">
            <Alert>{{richItem.placeHolder}}</Alert>
            <rich-editor class="product" :richContent="richItem" :richIndex="index" @send-text="getRichTextArray"></rich-editor>
        </div>

        <Button type="success" long @click="submitData" class="product">确认提交</Button>
        <Button type="error" long @click="elseSubmitData" class="product">另存为</Button>

        <Modal v-model="priceModal" title="价格添加" @on-ok="priceAdd" >
            <Input v-model="singlePrice" placeholder="请输入价格" style="width: 300px"></Input>
            <!--<DatePicker v-model="singleDate" type="date" placeholder="选择日期" style="width: 300px" class="product"></DatePicker>-->
             <Input v-model="singleDate" placeholder="请输入备注" style="width: 300px"></Input>

        </Modal>
    </div>
</template>

<script>
import imageUpload from './imageUpload'
import network from '../tools/network.js'
import richEditor from './richEditor'

export default {
    components: {
        imageUpload,
        richEditor,

    },
    data() {
        return {
            richItems: [{ content: '', placeHolder: "套餐特色" }, { content: '', placeHolder: "套餐内容" }, { content: '', placeHolder: "价格说明" }, { content: '', placeHolder: "预订须知" },],

            productId: '',
            //产品编号
            productNumber: '',
            //产品名称
            productName: '',
            //产品描述
            productDes: '',
            //产品价格
            productPrice: '',
            // 出发城市
            productPlace: '',
            productStartDate: '',
            productEndDate: '',
            //是否推荐
            isRecommend: false,
            isSpecialPrice: false,
            isFollowTeam: false,
            isFreeTravel: false,
            // 模态框
            priceModal: false,
            //是否特价
            //价格添加
            singlePrice:'',
            singleDate:'疗程(5次)',
            tagArray:[],
            //日期格式设置
            dateOption1: {
                disabledDate(date) {
                    return date && date.valueOf() < Date.now() - 86400000
                }
            },
            dateOption2: {
                disabledDate(date) {
                    return date && date.valueOf() < Date.now() - 86400000
                }
            },
            productTypes: [
            ],
            productTypeSelected: '',
            imageArray: [],

        }
    },
    created() {
        var _self = this
        _self.productId = this.$route.params.productId

        network.getThemelist().then(data => {
            if (data.length) {
                _self.productTypes = []
                data.forEach(obj => {
                    _self.productTypes.push({ label: obj.attributes.name, value: obj.attributes.type })
                })
            }
        })
        network.getTodoDetail(_self.productId, 'Product', (data) => {
            _self.richItems = data.detailContent
            _self.productNumber = data.onleyId
            _self.productPlace = data.place
            _self.productDes = data.describe
            _self.productName = data.name
            _self.productPrice = data.price
            _self.imageArray = data.imageArray
            _self.productStartDate = data.startDate
            _self.isFollowTeam = data.isFollowTeam || false
            _self.isFreeTravel = data.isFreeTravel || false
            _self.tagArray = data.tagArray || []
            if (data.isRecommend != undefined) {
                _self.isRecommend = data.isRecommend

            }
            if (data.isSpecialPrice != undefined) {
                _self.isSpecialPrice = data.isSpecialPrice

            }

            // _self.productEndDate = data.endDate
            _self.productTypeSelected = data.type
        }, (error) => {
            _self.$Message.error('获取信息失败,请重试')
        })
    },
    methods: {
        handleClose(index){
            this.tagArray.splice(index,1)
        },
        // 价格添加
        priceAdd() {
           let dict = {
               date:this.singleDate,
               price:this.singlePrice
           }

           this.tagArray.push(dict)
        },
        //价格选择
        priceSelect() {
            this.priceModal = true
        },
        getRichTextArray(data) {
            this.richItems[data.index].content = data.content
        },
        getImageArray(data) {
            this.imageArray = data
        },
        getNowData(){
            var _self = this
              var dict = {
                startDate: _self.productStartDate,
                // endDate: _self.productEndDate,
                name: _self.productName,
                describe: _self.productDes,
                type: _self.productTypeSelected,
                place: _self.productPlace,
                onleyId: _self.productNumber,
                price: _self.productPrice,
                imageArray: _self.imageArray,
                detailContent: _self.richItems,
                isRecommend: _self.isRecommend,
                isSpecialPrice: _self.isSpecialPrice,
                isFollowTeam: _self.isFollowTeam,
                isFreeTravel: _self.isFreeTravel,
                tagArray:_self.tagArray,
            }
            return dict
        },
        // 另存为
        elseSubmitData(){
            var _self = this
            let dict = this.getNowData()

             network.uploadProdut('', dict, function() {
                _self.$Message.success('另存为成功');
                // _self.$router.push('/productManage')
            })
        },
        submitData() {
            var _self = this
            var dict = this.getNowData()
            if (!this.imageArray.length) {
                _self.$Message.error('请上传图片至少一张');
            }

            network.uploadProdut(_self.productId, dict, function() {
                _self.$Message.success('保存成功');
                setTimeout(function() {
                    _self.$router.go(0)
                }, 1000)
            })
        }
    }
}
</script>


