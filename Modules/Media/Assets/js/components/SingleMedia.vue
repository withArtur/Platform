<template>
    <div>
        <label class="el-form-item__label">{{ getFieldLabel() }}</label>
        <div class="jsThumbnailImageWrapper jsSingleThumbnailWrapper" v-if="hasSelectedMedia">
            <figure>
                <img :src="this.selectedMedia.medium_thumb" alt="" v-if="this.selectedMedia.is_image"/>
                <i :class="`fa ${this.selectedMedia.fa_icon}`" style="font-size: 60px;" v-if="! this.selectedMedia.is_image"></i>
                <span v-if="! this.selectedMedia.is_image" style="display:block;">{{ this.selectedMedia.filename }}</span>
            </figure>
            <div class="clearfix"></div>
            <el-button type="button" @click="unSelectMedia">{{ trans('media.remove media') }}</el-button>
        </div>
        <div class="" v-else>
            <el-button type="button" @click="dialogVisible = true">{{ trans('media.Browse') }}</el-button>
        </div>

        <el-dialog
            :visible.sync="dialogVisible"
            width="75%"
            :before-close="handleClose">

            <media-list single-modal :event-name="this.eventName"></media-list>

            <span slot="footer" class="dialog-footer">
                <el-button @click="dialogVisible = false">{{ trans('core.button.cancel') }}</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
    import axios from 'axios';
    import _ from 'lodash';
    import UploadZone from './UploadZone.vue';
    import MediaList from './MediaList.vue';
    import StringHelpers from '../../../../Core/Assets/js/mixins/StringHelpers.vue';
    import RandomString from '../mixins/RandomString';

    export default {
        mixins: [StringHelpers, RandomString],
        props: {
            zone: { type: String, required: true },
            entity: { type: String, required: true },
            entityId: { default: null },
            label: { type: String },
        },
        components: {
            'upload-zone': UploadZone,
            'media-list': MediaList,
        },
        watch: {
            entityId() {
                if (this.entityId) {
                    this.fetchMedia();
                }
            },
        },
        data() {
            return {
                dialogVisible: false,
                selectedMedia: {},
                eventName: '',
            };
        },
        computed: {
            hasSelectedMedia() {
                return !_.isEmpty(this.selectedMedia);
            },
        },
        methods: {
            handleClose(done) {
                done();
            },
            unSelectMedia() {
                this.selectedMedia = {};
                this.$emit('single-file-selected', { id: null, zone: this.zone });
            },
            fetchMedia() {
                axios.get(route('api.media.find-first-by-zone-and-entity', {
                        zone: this.zone,
                        entity: this.entity,
                        entity_id: this.entityId,
                    }))
                    .then((response) => {
                        this.$emit('single-file-selected', _.merge(response.data.data, { zone: this.zone }));
                        this.selectedMedia = response.data.data;
                    });
            },
            getFieldLabel() {
                return this.label || this.ucwords(this.zone.replace('_', ' '));
            },
        },
        mounted() {
            if (this.entityId) {
                this.fetchMedia();
            }
            this.eventName = `file-was-selected${this.randomString()}${Math.floor(Math.random() * 999999)}`;

            this.$events.listen(this.eventName, (mediaData) => {
                this.dialogVisible = false;
                this.selectedMedia = mediaData;
                this.$emit('single-file-selected', _.merge(mediaData, { zone: this.zone }));
            });
        },
    };
</script>
