<template>
    <div class="blog-horizontal-cards">
        <!-- wwManager:start -->
        <wwSectionEditMenu :sectionCtrl="sectionCtrl" :options="openOptions"></wwSectionEditMenu>
        <!-- wwManager:end -->
        <wwObject class="background" :ww-object="section.data.background" ww-category="background"></wwObject>

        <!--TOP WWOBJS-->
        <div class="top-ww-objs">
            <wwLayoutColumn tag="div" ww-default="ww-image" :ww-list="section.data.topWwObjs" class="top-ww-obj" @ww-add="add(section.data.topWwObjs, $event)" @ww-remove="remove(section.data.topWwObjs, $event)">
                <wwObject v-for="topWwObj in section.data.topWwObjs" :key="topWwObj.uniqueId" :ww-object="topWwObj"></wwObject>
            </wwLayoutColumn>
        </div>

        <!--THUMBNAILS-->
        <div class="thumbnail-container">
            <div class="thumbnail" v-for="(thumbnail, index) in section.data.thumbnails" :key="thumbnail.uniqueId">
                <!-- wwManager:start -->
                <wwContextMenu tag="div" class="contextmenu" v-if="sectionCtrl.getEditMode() == 'CONTENT'" @ww-add-before="addCardBefore(index)" @ww-add-after="addCardAfter(index)" @ww-remove="removeCard(index)">
                    <div class="wwi wwi-config"></div>
                </wwContextMenu>
                <!-- wwManager:end -->

                <div class="content-container" :style="{'order': isPair(index)}">
                    <wwObject class="background" :ww-object="thumbnail.background" ww-category="background" ww-default-object-type="ww-color"></wwObject>
                    <wwLayoutColumn tag="div" ww-default="ww-image" :ww-list="thumbnail.contents" class="content" @ww-add="add(thumbnail.contents, $event)" @ww-remove="remove(thumbnail.contents, $event)">
                        <wwObject v-for="content in thumbnail.contents" :key="content.uniqueId" :ww-object="content"></wwObject>
                    </wwLayoutColumn>
                </div>
                <div class="image-container">
                    <wwObject class="background" :ww-object="thumbnail.image" ww-category="background" ww-default-object-type="ww-image" :style="{'border-top-right-radius': '7px', 'border-bottom-right-radius': '7px'}"></wwObject>
                </div>
            </div>
        </div>

        <!--BOTTOM WWOBJS-->
        <div class="bottom-ww-objs">
            <wwLayoutColumn tag="div" ww-default="ww-image" :ww-list="section.data.bottomWwObjs" class="top-ww-obj" @ww-add="add(section.data.bottomWwObjs, $event)" @ww-remove="remove(section.data.bottomWwObjs, $event)">
                <wwObject v-for="bottomWwObj in section.data.bottomWwObjs" :key="bottomWwObj.uniqueId" :ww-object="bottomWwObj"></wwObject>
            </wwLayoutColumn>
        </div>
    </div>
</template>

<script>

/* wwManager:start */
import featureCColumnPerLine from './featureCColumnPerLine.vue'
wwLib.wwPopups.addPopup('featureCColumnPerLine', featureCColumnPerLine);
wwLib.wwPopups.addStory('FEATURE_C_COLUMN_COUNT', {
    title: {
        en: 'Column per line',
        fr: 'Nombre de colonnes par ligne'
    },
    type: 'featureCColumnPerLine',
    buttons: {
        FINISH: {
            text: {
                en: 'Finish',
                fr: 'Terminer'
            },
            next: false
        }
    }
});
/* wwManager:end */

export default {
    name: "__COMPONENT_NAME__",
    props: {
        sectionCtrl: Object
    },
    data() {
        return {
            maxThumbnailsPerLine: 4,
        }
    },
    computed: {
        section() {
            return this.sectionCtrl.get();
        }
    },
    methods: {
        initData() {
            let needUpdate = false;

            //Init objects
            if (!this.section.data.background) {
                this.section.data.background = wwLib.wwObject.getDefault({ type: 'ww-color', data: { color: 'white' } });
                needUpdate = true;
            }
            if (_.isEmpty(this.section.data.topWwObjs)) {
                this.section.data.topWwObjs = [];
                needUpdate = true;
            }
            if (_.isEmpty(this.section.data.bottomWwObjs)) {
                this.section.data.bottomWwObjs = [];
                needUpdate = true;
            }
            if (_.isEmpty(this.section.data.thumbnails)) {
                this.section.data.thumbnails = [];
                needUpdate = true;
            }
            if (_.isEmpty(this.section.data.thumbnails)) {
                this.section.data.thumbnails.push({
                    background: wwLib.wwObject.getDefault({ type: 'ww-color', data: { color: 'white' } }),
                    image: wwLib.wwObject.getDefault({ type: 'ww-image', data: {} }),
                    contents: []
                })
                needUpdate = true;
            }
            if (!this.section.data.thumbnailsPerLine) {
                this.section.data.thumbnailsPerLine = 4;
                needUpdate = true;
            }
            if (needUpdate) {
                this.sectionCtrl.update(this.section);
            }
        },
        init() {
            window.addEventListener("resize", this.setThumbnailsPerLine);
        },
        isPair(index) {
            if (index % 2) {
                return 1;
            } else {
                return 0;
            }
        },

        /* wwManager:start */
        add(list, options) {
            list.splice(options.index, 0, options.wwObject);

            this.sectionCtrl.update(this.section);
        },
        remove(list, options) {
            list.splice(options.index, 1);

            this.sectionCtrl.update(this.section);
        },
        getNewCard() {
            const card = JSON.parse(JSON.stringify(this.section.data.thumbnails[0]))
            wwLib.wwUtils.changeUniqueIds(card)
            return card
        },
        addCardBefore(index) {
            const newCard = this.getNewCard()
            this.section.data.thumbnails.splice(index, 0, newCard);
            this.sectionCtrl.update(this.section);
        },
        addCardAfter(index) {
            const newCard = this.getNewCard()
            this.section.data.thumbnails.splice(index + 1, 0, newCard);
            this.sectionCtrl.update(this.section);
        },
        removeCard(index) {
            if (!this.section.data.thumbnails.length) {
                return;
            }
            this.section.data.thumbnails.splice(index, 1);

            this.sectionCtrl.update(this.section);
        },

        async openOptions() {
            let options = {
                firstPage: 'FEATURE_C_COLUMN_COUNT',
                data: {
                    columnPerLine: this.section.data.thumbnailsPerLine
                },
            }

            const result = await wwLib.wwPopups.open(options)

            if (result.columnPerLine) {
                this.section.data.thumbnailsPerLine = result.columnPerLine;

                this.sectionCtrl.update(this.section);

                this.setThumbnailsPerLine();
            }


            console.log(result);
        }
        /* wwManager:end */
    },
    created() {
        this.initData();
    },
    mounted: function () {
        this.init();
    },
    beforeDestroy() {
        window.removeEventListener("resize", this.setThumbnailsPerLine);
    }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.blog-horizontal-cards {
    position: relative;

    .background {
        position: absolute;
        width: 100%;
        height: 100%;
        top: 0;
        left: 0;
    }

    .top-ww-objs,
    .bottom-ww-objs {
        position: relative;

        .top-ww-obj,
        .bottom-ww-obj {
            position: relative;
        }
    }

    .thumbnail-container {
        display: flex;
        justify-content: center;
        flex-wrap: wrap;

        .thumbnail {
            display: flex;
            width: 75%;
            margin-right: 15px;
            position: relative;
            margin: 30px 15px;
            background-color: white;
            min-height: 50px;
            box-shadow: 0 10px 40px 0 rgba(113, 124, 137, 0.2);
            border-radius: 7px;
            transition: transform 0.4s ease-out, box-shadow 0.4s ease-out;

            &:hover {
                box-shadow: 0 16px 32px rgba(113, 124, 137, 0.4);
                transform: translateY(-10px);
            }

            .content {
                position: relative;
            }

            .content-container {
                position: relative;
                width: 50%;
            }

            .image-container {
                position: relative;
                width: 50%;
                .background {
                    overflow: hidden;
                }
            }

            /* wwManager:start */
            .contextmenu {
                position: absolute;
                top: 0;
                left: 0;
                transform: translate(-50%, -50%);
                width: 30px;
                height: 30px;
                color: white;
                background-color: #ef811a;
                border-radius: 100%;
                display: flex;
                justify-content: center;
                align-items: center;
                font-size: 1.2rem;
                cursor: pointer;
                z-index: 1;
            }
            /* wwManager:end */
        }
    }
}

/* wwManager:start */
.ww-editing {
    .thumbnail-container {
        .thumbnail {
            &:hover {
                transform: none !important;
            }
        }
    }
}
/* wwManager:end */
</style>
