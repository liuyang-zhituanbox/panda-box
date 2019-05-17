<template>
    <div>
        <el-row>
            <el-col :span="24">
                <el-input placeholder="请输入" v-model="inputValue" class="input-with-select">
                    <el-button slot="append" icon="el-icon-search" @click="searchHandler"></el-button>
                </el-input>
            </el-col>
        </el-row>
        <el-divider content-position="left">{{displaySearchText}}</el-divider>
        <template v-for="chunkImg in chunkImgResult">
            <el-row :gutter="10">
                <el-col :span="colSpan" v-for="img in chunkImg" :key="img.out_id">
                    <transition name="el-fade-in-linear">
                        <el-card shadow="hover">
                            <el-image :src="img.image_url" style="width: 100%; height: 196px" :lazy="true"></el-image>
                        </el-card>
                    </transition>
                </el-col>
            </el-row>
        </template>
    </div>
</template>

<script>
    import _ from 'lodash';
    import axios from 'axios';

    export default {
        name: "Search",
        data() {
            return {
                inputValue: '金馆长',
                displaySearchText: '',
                imgResult: [],
                chunkCount: 4
            }
        },
        computed: {
            chunkImgResult() {
                return _.chunk(this.imgResult, this.chunkCount);
            },
            colSpan() {
                return 24 / this.chunkCount;
            }
        },
        methods: {
            searchHandler() {
                if (_.isEmpty(this.inputValue) || this.inputValue.trim() === '') {
                    //不搜索
                    this.imgResult = [];
                    this.displaySearchText = '';
                } else {
                    //搜索
                    this.imgResult = [];
                    this.displaySearchText = this.inputValue;
                    let promise = axios.get(`https://www.doutula.com/api/search?keyword=${this.displaySearchText}&mime=0&page=1`);
                    promise.then(({data}) => {
                        if (data.status === 1) {
                            this.imgResult = data.data.list;
                        }
                        console.debug(this.imgResult);
                    })
                }
            }
        }
    }
</script>

<style scoped>

</style>
