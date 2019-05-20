<template>
    <div>
        <el-row>
            <el-col :span="24">
                <el-input placeholder="请输入" v-model="inputValue" @keyup.enter.native="searchHandler"
                          class="" id="box-search">
                    <el-button slot="append" icon="el-icon-search" @click="searchHandler"></el-button>
                </el-input>
            </el-col>
        </el-row>
        <el-divider content-position="left">{{displaySearchText}}</el-divider>
        <template v-for="chunkImg in chunkImgResult">
            <el-row :gutter="space" :style="`margin-bottom: ${space}px`" class="box-el-row">
                <el-col :span="colSpan" v-for="img in chunkImg" :key="img.out_id">
                    <el-badge v-if="img.number<10" :value="img.number" type="primary">
                        <el-card shadow="hover" :body-style="{ padding: '0px' }" class="box-el-card">
                            <el-image :src="img.image_url" class="box-image" :lazy="true"></el-image>
                            <div>
                                <el-row>
                                    <el-button type="text" @click="copyImage(img)">复制</el-button>
                                </el-row>
                            </div>
                        </el-card>
                    </el-badge>
                    <el-card v-else shadow="hover" :body-style="{ padding: '0px' }" class="box-el-card">
                        <el-image :src="img.image_url" class="box-image" :lazy="true"></el-image>
                        <div>
                            <el-row>
                                <el-button type="text" @click="copyImage(img)">复制</el-button>
                            </el-row>
                        </div>
                    </el-card>
                </el-col>
            </el-row>
        </template>
    </div>
</template>

<script>
    import _ from 'lodash';
    import axios from 'axios';
    import Mousetrap from 'mousetrap';

    const {nativeImage, clipboard} = require('electron');

    export default {
        name: "Search",
        data() {
            return {
                inputValue: '金馆长',
                displaySearchText: '',
                imgResult: [],
                chunkCount: 4,
                /*间距*/
                space: 10,
                /*图片二进制*/
                imgBlobMap: {}
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
        mounted() {
            document.getElementById('box-search').classList.add('mousetrap');

            let keys = [];
            for (let i = 1; i < 10; i++) {
                keys.push(`alt+${i}`);
            }
            Mousetrap.bind(keys, (keyboardEvent) => {
                // console.log(keys, keyboardEvent);
                let img = this.imgResult.find(value => value.number + '' === keyboardEvent.key);
                if (_.isUndefined(img)) {
                    this.$message.warning('不存在');
                } else {
                    this.copyImage(img);
                }
                // return false to prevent default browser behavior
                // and stop event from bubbling
                return false;
            });

            Mousetrap.bind(['ctrl+f'], (keyboardEvent) => {
                document.getElementById('box-search').focus();
                // return false to prevent default browser behavior
                // and stop event from bubbling
                return false;
            });
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
                    let promise = axios.post(`https://www.doutula.com/api/search?keyword=${this.displaySearchText}&mime=1&page=1`);
                    promise.then(({data}) => {
                        if (data.status === 1) {
                            if (Array.isArray(data.data.list)) {
                                let array = data.data.list;
                                array = array.filter(value => !value.image_url.endsWith('.gif'));
                                for (let i = 1; i <= array.length; i++) {
                                    array[(i - 1)].number = i;
                                }
                                // array.filter(value => !value.image_url.endsWith('.gif'));
                                this.imgResult = array;
                            }
                        }
                        console.debug(this.imgResult);
                    })
                }
            },
            copyImage(img) {
                let imgUrl = img.image_url;
                axios.get(imgUrl, {
                    responseType: "blob"
                }).then(({data}) => {
                    let reader = new FileReader();
                    reader.onloadend = (e) => {
                        debugger;
                        const ni = nativeImage.createFromDataURL(e.target.result);
                        clipboard.writeImage(ni);
                        this.$message.success('复制成功');
                    };
                    reader.readAsDataURL(data);
                });
            }
        }
    }
</script>

<style scoped>
    .box-image {
        width: 185px;
        height: 185px;
    }

    .box-el-row {

    }

    .box-el-card {
        width: 185px;
    }
</style>
