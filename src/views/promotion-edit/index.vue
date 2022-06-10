<template>
    <div class="app-container">
        <el-table v-loading="listLoading" :data="list" border fit highlight-current-row style="width: 100%">
            <!-- <el-table-column align="center" label="ID" width="80">
                <template slot-scope="{row}">
                    <span>{{ row.id }}</span>
                </template>
            </el-table-column> -->

            <el-table-column width="180px" align="center" label="推广活动名称">
                <template slot-scope="{row}">
                    <!-- <template v-if="row.edit">
                        <el-input v-model="row.title" class="edit-input" size="small" /> -->
                    <!-- <el-button class="cancel-btn" size="small" icon="el-icon-refresh" type="warning"
                            @click="cancelEdit(row)">
                            取消
                        </el-button> -->
                    <!-- </template> -->

                    <span>{{ row.title }}</span>
                </template>
            </el-table-column>

            <el-table-column width="120px" align="center" label="推广活动时间">
                <template slot-scope="{row}">
                    <!-- <template v-if="row.edit">
                        <el-input v-model="row.title1" class="edit-input" size="small" />
                        <el-button class="cancel-btn" size="small" icon="el-icon-refresh" type="warning"
                            @click="cancelEdit(row)">
                            取消
                        </el-button> -->
                    <!-- </template> -->
                    <span>{{ row.title1 }}</span>
                </template>
            </el-table-column>

            <el-table-column min-width="160px" label="活动简介">
                <template slot-scope="{row}">
                    <!-- <template v-if="row.edit">
                        <el-input v-model="row.title2" class="edit-input" size="small" /> -->
                    <!-- <el-button class="cancel-btn" size="small" icon="el-icon-refresh" type="warning"
                            @click="cancelEdit(row)">
                            取消
                        </el-button> -->
                    <!-- </template> -->
                    <span>{{ row.title2 }}</span>
                </template>
            </el-table-column>

            <el-table-column class-name="status-col" label="状态" width="110">
                <template slot-scope="{row}">
                    <el-tag :type="row.status | statusFilter">
                        {{ row.status }}
                    </el-tag>
                    <!-- <span>{{ row.status }}</span> -->
                </template>
            </el-table-column>

            <!-- <el-table-column min-width="300px" label="Title">
                <template slot-scope="{row}">
                    <template v-if="row.edit">
                        <el-input v-model="row.title" class="edit-input" size="small" />
                        <el-button class="cancel-btn" size="small" icon="el-icon-refresh" type="warning"
                            @click="cancelEdit(row)">
                            取消
                        </el-button>
                    </template>
                    <span v-else>{{ row.title }}</span>
                </template>
            </el-table-column> -->

            <el-table-column align="center" label="操作" width="180">
                <template slot-scope="{row}">
                    <!-- <el-button v-if="row.edit" type="success" size="small" icon="el-icon-circle-check-outline"
                        @click="confirmEdit(row)">
                        确定
                    </el-button> -->
                    <!-- <el-button type="primary" size="small" icon="el-icon-edit" @click="row.edit = !row.edit">
                        编辑
                    </el-button> -->

                    <el-button type="primary" size="small" @click="dialogVisible = true, row.edit = !row.edit"
                        icon="el-icon-edit">编辑</el-button>
                    <el-dialog title="添加活动" :visible.sync="dialogVisible" width="50%" :before-close="handleClose">

                        <el-form :model="form" label-position="left">
                            <el-form-item label="推广活动名称" :label-width="formLabelWidth">
                                <el-input v-model="row.title" class="edit-input" size="small" />
                            </el-form-item>

                            <el-form-item label="推广活动时间" :label-width="formLabelWidth">
                                <el-input v-model="row.title1" class="edit-input" size="small" />
                            </el-form-item>

                            <el-form-item label="活动简介" :label-width="formLabelWidth">
                                <el-input v-model="row.title2" class="edit-input" size="small" />
                            </el-form-item>


                            <el-form-item label="状态" :label-width="formLabelWidth">
                                <el-select v-model="row.status" placeholder="请选择活动区域">
                                    <el-option v-for="item in statusOptions" :key="item" :label="item" :value="item" />
                                </el-select>
                            </el-form-item>

                            <el-form-item label="推广编辑" :label-width="formLabelWidth">
                                <tinymce v-model="content" :height="300" />
                                <div class="editor-content" v-html="content" />
                            </el-form-item>




                        </el-form>


                        <span slot="footer" class="dialog-footer">
                            <el-button @click="cancelEdit(row), dialogVisible = false">取 消</el-button>
                            <el-button type="primary" @click="confirmEdit(row), dialogVisible = false">确 定</el-button>
                        </span>
                    </el-dialog>

                </template>
            </el-table-column>
        </el-table>
        <el-pagination small layout="prev, pager, next" :total="50">
        </el-pagination>

        
    </div>

</template>

<script>
import { fetchList } from '@/api/article'
import Tinymce from '@/components/Tinymce'

export default {

    name: 'InlineEditTable',
    components: { Tinymce },
    filters: {
        statusFilter(status) {
            const statusMap = {
                published: 'success',
                draft: 'info',
                deleted: 'danger'
            }
            return statusMap[status]
        }
    },
    data() {
        return {
            input: '',
            dialogVisible: false,
            statusOptions: ['已启用', '已停止'],
            list: null,
            listLoading: true,
            listQuery: {
                page: 1,
                limit: 10
            }
        }
    },
    created() {
        this.getList()
    },
    methods: {
        async getList() {
            this.listLoading = true
            const { data } = await fetchList(this.listQuery)
            const items = data.items
            this.list = items.map(v => {
                this.$set(v, 'edit', false) // https://vuejs.org/v2/guide/reactivity.html
                v.originalTitle = v.title //  will be used when user click the cancel botton
                return v
            })
            this.listLoading = false
        },
        cancelEdit(row) {
            row.title = row.originalTitle
            row.edit = false
            this.$message({
                message: 'The title has been restored to the original value',
                type: 'warning'
            })
        },
        confirmEdit(row) {
            row.edit = false
            row.originalTitle = row.title
            this.$message({
                message: 'The title has been edited',
                type: 'success'
            })
        }
    }
}
</script>

<style scoped>
.edit-input {
    padding-right: 100px;
}

.cancel-btn {
    position: absolute;
    right: 15px;
    top: 10px;
}
</style>
