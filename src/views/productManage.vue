<style scoped>
.header {
    margin-bottom: 30px;
    margin-top: 20px
}
</style>

<template>
    <div class="content">

        <div class="header" style="margin-top:20px">
            <Row>
                <Col span="2">
                <h4 style="margin-top:10px">分类:</h4>
                </Col>
                <Col span="8">
                <Select v-model="showTypeSelected" style="width:200px" @on-change="showTypeChange">
                    <Option v-for="item in showTypes" :value="item.value" :key="item.value">{{ item.label }}</Option>
                </Select>
                </Col>
                <Col span="2">
                <h4 style="margin-top:10px">过滤:</h4>
                </Col>
                <Col span="9" style="margin:0.5rem">
                <CheckboxGroup v-model="sortArray" @on-change="checkGroupChange">
                    <Checkbox label="isSort">
                        <Icon type="arrow-up-a"></Icon>
                        <span>置顶</span>
                    </Checkbox>
                    <Checkbox label="isRecommend">
                        <Icon type="navigate"></Icon>
                        <span>推荐</span>
                    </Checkbox>
                    <Checkbox label="isSpecialPrice">
                        <Icon type="star"></Icon>
                        <span>特价</span>
                    </Checkbox>
                    <Checkbox label="isFreeTravel">
                        <Icon type="navigate"></Icon>
                        <span>特色</span>
                    </Checkbox>
                    <Checkbox label="isFollowTeam">
                        <Icon type="flag"></Icon>
                        <span>百安丽</span>
                    </Checkbox>
                </CheckboxGroup>
                </Col>
            </Row>
        </div>
        <Table height="500" :columns="columns" :data="dataArray"></Table>
        <Modal v-model="modal1" title="删除操作" @on-ok="removeProdcut">
            <p>确认将编号为{{productNumber}}的产品删除吗</p>
        </Modal>
        <Modal v-model="modal2" title="产品分类" @on-ok="confirmOperation">

            <label>是否置顶:</label>
            <i-switch v-model="modifyObj.isSort"></i-switch>
            <label>是否特价:</label>

            <i-switch v-model="modifyObj.isSpecialPrice"></i-switch>
            <label>是否推荐:</label>

            <i-switch v-model="modifyObj.isRecommend"></i-switch>

            <label>是否特色:</label>
            <i-switch v-model="modifyObj.isFreeTravel"></i-switch>

            <label>是否百安丽:</label>
            <i-switch v-model="modifyObj.isFollowTeam"></i-switch>
        </Modal>

    </div>
</template>

<script>
import network from '../tools/network.js'
export default {
    data() {
        return {
            uid: '',
            productNumber: '',
            modal1: false,
            modal2: false,
            //显示类型过滤
            showTypeSelected: 0,
            sortArray: [],
            rawDataArray: [],
            modifyObj: {
                isSort: false,
                isFollowTeam: false,
                isFreeTravel: false,
                isSpecialPrice: false,
                isRecommend: false,
            },
            showTypes: [{
                label: '全部',
                value: 0
            }],
            columns: [{
                title: '名称',
                key: 'name',
                width: 150,
            },
            {
                title: '置顶',
                key: 'isSort'
            },
            {
                title: '特价',
                key: 'isSpecialPrice'
            },
            {
                title: '特色',
                key: 'isFreeTravel'
            },
            {
                title: '推荐',
                key: 'isRecommend'
            },
            {
                title: '百安丽',
                key: 'isFollowTeam'
            },
            {
                title: '分类',
                key: 'action',
                width: 150,
                align: 'center',
                render: (h, params) => {
                    return h('div', [
                        h('Button', {
                            props: {
                                type: 'info',
                                size: 'small'
                            },
                            style: {
                                marginRight: '5px'
                            },
                            on: {
                                click: () => {
                                    // this.setFirst(params)
                                    this.sortOperation(params)
                                }
                            }
                        }, '操作')
                    ]);
                }
            },
            {
                title: '操作',
                key: 'action',
                width: 150,
                align: 'center',
                render: (h, params) => {
                    return h('div', [
                        h('Button', {
                            props: {
                                type: 'primary',
                                size: 'small'
                            },
                            style: {
                                marginRight: '5px'
                            },
                            on: {
                                click: () => {
                                    this.modfifyProduct(params)
                                }
                            }
                        }, '修改'),
                        h('Button', {
                            props: {
                                type: 'error',
                                size: 'small'
                            },
                            on: {
                                click: () => {
                                    this.deleteProduct(params)
                                }
                            }
                        }, '配置')
                    ]);
                }
            }
            ],
            dataArray: [],
        }
    },
    created() {
        this.configData()
        this.congfigChooseType()
    },
    methods: {
        // 表格数据过滤
        checkGroupChange(checkGroups) {
            let _self = this
            let filterArray = new Set()
            for (let myObj of _self.rawDataArray) {
                let isSign = true
                for (let obj of checkGroups) {
                    if (myObj[obj] != true) {
                        isSign = false
                    }
                }
                if (isSign) {
                     filterArray.add(myObj) 
                }
            }
            _self.dataArray = [...filterArray]
            if (!checkGroups.length) {
                _self.dataArray = _self.rawDataArray
            }
        },
        configData() {
            var _self = this

            network.getProductList((data) => {
                _self.dataArray = data
                _self.rawDataArray = data

            }, (error) => {
                _self.$Message.error('获取数据失败请重试');
            })

        },

        congfigChooseType() {
            var _self = this
            network.getThemelist().then(data => {
                if (data.length) {
                    _self.showTypes = [{
                        label: '全部',
                        value: 0
                    }]
                    // _self.showTypes.push({ label: '全部', value: 0 })
                    // _self.showTypeSelected = 0
                    data.forEach(obj => {
                        _self.showTypes.push({
                            label: obj.attributes.name,
                            value: obj.attributes.type
                        })
                    })
                }
            })
        },
        sortOperation(params) {
            this.modal2 = true
            this.productNumber = params.row.onleyId
            this.uid = params.row.uid
            let rowObj = params.row
            this.modifyObj.isSort = rowObj.isSort || false
            this.modifyObj.isSpecialPrice = rowObj.isSpecialPrice || false
            this.modifyObj.isFollowTeam = rowObj.isFollowTeam || false
            this.modifyObj.isFreeTravel = rowObj.isFreeTravel || false
            this.modifyObj.isRecommend = rowObj.isRecommend || false
        },
        //修改
        modfifyProduct(params) {
            let productId = params.row.uid
            this.$router.push({
                name: 'upload',
                params: {
                    'productId': productId
                }
            })
        },
        //删除
        deleteProduct(params) {
            this.modal1 = true
            this.productNumber = params.row.onleyId
            this.uid = params.row.uid
        },
        //确认修改分类
        confirmOperation() {
            this.setFirst()
        },
        removeProdcut(uid) {
            var _self = this
            network.deleteProductWithId(this.uid, () => {

                _self.configData()
            })
        },
        //选中值改变
        showTypeChange(myValue) {
            var _self = this
            myValue = this.showTypeSelected
            if (myValue == 0) {
                _self.configData()
            } else {

                network.getDetailItemList(myValue).then(data => {
                    var netDataArray = [];
                    for (var model of data) {
                        // model.attributes.endDate = model.attributes.endDate.toISOString().slice(0, 10)
                        // model.attributes.startDate = model.attributes.startDate.toISOString().slice(0, 10)
                        model.attributes.uid = model.id;
                        netDataArray.push(model.attributes);
                    }

                    _self.dataArray = netDataArray
                }, (error) => {


                });


            }
        },
        //置顶
        setFirst() {
            var _self = this

            network.setFirstProduct(this.uid, this.modifyObj).then(data => {
                this.$Message.success('操作成功')
                _self.showTypeChange()
            }, error => {
                this.$Message.error('操作失败')

            })
        },
    },

}
</script>