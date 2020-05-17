<template>
    <section class="article-wrapper" v-html="articleContent"></section>
</template>
<script>
import { mapState, mapGetters } from 'vuex'
import { getAriticleSrcById } from '../../../tools/column-tools'
import { uid } from '../../../tools/index'
import axios from 'axios'
export default {
    data() {
        return {
            articleContent: ''
        }
    },
    computed: {
        ...mapState('column', ['curArticleId']),
        ...mapGetters('column', ['curContents'])
    },
    methods: {
        async handlerArticle(src) {
            src = src.replace('./', '')
            let reqUrl = window.location.origin + '/api/' + src
            let res = await axios.get(reqUrl)
            if (res.status !== 200) {
                return
            }
            this.articleContent = this.filterWaterMark(res.data)
            this.$nextTick(() => {
                this.generateOutline()
                this.polyfillPage()
            })
        },
        filterWaterMark(content) {
            //去除水印
            return content
                .replace('<p class="x">更多课程请加QQ群170701297</p>', '')
                .replace(
                    '<p class="x">所有最新极客时间课程请加QQ群170701297</p>',
                    ''
                )
        },
        generateOutline() {
            let wrapper = document.querySelector('.article-wrapper')
            let childrens = wrapper.getElementsByTagName('*')
            let treeArray = []

            for (let i = 0; i < childrens.length - 1; i++) {
                let nodeName = childrens[i].nodeName
                if (nodeName == 'H1' || nodeName == 'H2' || nodeName == 'H3') {
                    childrens[i].id = uid()
                    treeArray.push({
                        id: childrens[i].id,
                        name: childrens[i].innerText,
                        tag: childrens[i].nodeName,
                        top: childrens[i].offsetTop
                    })
                }
            }
            this.$store.commit('column/setOutline', treeArray)
        },
        polyfillPage() {}
    },
    watch: {
        curArticleId: {
            immediate: true,
            handler(id) {
                let src = getAriticleSrcById(this.curContents, id)
                if (!src) return
                this.handlerArticle(src)
            }
        }
    }
}
</script>
<style lang="scss" scoped>
.article-wrapper {
    padding: 80px 40px;
    overflow-y: auto;
    box-sizing: border-box;
    height: 100%;
}
</style>