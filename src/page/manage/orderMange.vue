<template>
    <div class="fillcontain">
        <HeadTop></HeadTop>
        <div class="table_container">
            <el-table
                :data="tableData"
                @expand='expand'
                :expand-row-keys='expendRow'
                :row-key="row => row.index"
                :default-sort="{prop:'total_amount',rating:'desc'}">
                <el-table-column type="expand">
                    <template slot-scope="props">
                        <el-form label-position="left" inline class="demo-table-expand">
                            <el-form-item label="用户名">
                                <span>{{ props.row.user_name }}</span>
                            </el-form-item>
                            <el-form-item label="店铺名称">
                                <span>{{ props.row.restaurant_name }}</span>
                            </el-form-item>
                            <el-form-item label="收货地址">
                                <span>{{ props.row.address }}</span>
                            </el-form-item>
                            <el-form-item label="店铺 ID">
                                <span>{{ props.row.restaurant_id }}</span>
                            </el-form-item>
                            <el-form-item label="店铺地址">
                                <span>{{ props.row.restaurant_address }}</span>
                            </el-form-item>
                        </el-form>
                    </template>
                </el-table-column>
                <el-table-column
                    label="订单 ID"
                    prop="id">
                </el-table-column>
                <el-table-column
                    label="总价格"
                    prop="total_amount"
                    sortable>
                </el-table-column>
                <el-table-column
                    label="订单状态"
                    prop="status">
                </el-table-column>
            </el-table>
            <div class="Pagination">
                <el-pagination
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :current-page="currentPage"
                    :page-size="20"
                    :page-sizes="[20,50,100,200]"
                    layout="total,sizes, prev, pager, next,jumper"
                    :total="count">
                </el-pagination>
            </div>
        </div>
    </div>
</template>

<script>
    import HeadTop from '../../components/HeadTop'
    import {getOrderCount} from '@/api/home'
    import {getOrderList, getResturantDetail, getUserInfo, getAddressById} from '@/api/getData'

    export default {
        data() {
            return {
                tableData: [],
                currentRow: null,
                offset: 0,
                limit: 20,
                count: 0,
                currentPage: 1,
                restaurant_id: null,
                expendRow: [],
            }
        },
        components: {
            HeadTop,
        },
        created() {
            this.restaurant_id = this.$route.query.restaurant_id;
            this.initData();
        },
        mounted() {

        },
        methods: {
            async initData() {
                try {
                    const countData = await getOrderCount({restaurant_id: this.restaurant_id});
                    if (countData.status == 1) {
                        this.count = countData.count;
                    } else {
                        throw new Error('获取数据失败');
                    }
                    this.getOrders();
                } catch (err) {
                    console.log('获取数据失败', err);
                }
            },
            handleSizeChange(val) {
                this.limit = val
                this.getOrders()
            },
            handleCurrentChange(val) {
                this.currentPage = val;
                this.offset = (val - 1) * this.limit;
                this.getOrders()
            },
            async getOrders() {
                const Orders = await getOrderList({
                    offset: this.offset,
                    limit: this.limit,
                    restaurant_id: this.restaurant_id
                });
                this.tableData = [];
                Orders.forEach((item, index) => {
                    const tableData = {};
                    tableData.id = item.id;
                    tableData.total_amount = item.total_amount;
                    tableData.status = item.status_bar.title;
                    tableData.user_id = item.user_id;
                    tableData.restaurant_id = item.restaurant_id;
                    tableData.address_id = item.address_id;
                    tableData.index = index;
                    this.tableData.push(tableData);
                })
            },
            async expand(row, status) {
                if (status) {
                    const restaurant = await getResturantDetail(row.restaurant_id);
                    const userInfo = await getUserInfo(row.user_id);
                    const addressInfo = await getAddressById(row.address_id);

                    this.tableData.splice(row.index, 1, {
                        ...row, ...{
                            restaurant_name: restaurant.name,
                            restaurant_address: restaurant.address,
                            address: addressInfo.address,
                            user_name: userInfo.username
                        }
                    });
                    this.$nextTick(() => {
                        this.expendRow.push(row.index);
                    })
                } else {
                    const index = this.expendRow.indexOf(row.index);
                    this.expendRow.splice(index, 1)
                }
            },
        },
    }
</script>

<style lang="less">
    @import '../../style/mixin';

    .demo-table-expand {
        font-size: 0;
    }

    .demo-table-expand label {
        width: 90px;
        color: #99a9bf;
    }

    .demo-table-expand .el-form-item {
        margin-right: 0;
        margin-bottom: 0;
        width: 50%;
    }
</style>
