<template>
    <default-field @keydown.native.stop :field="field" :errors="errors" :full-width-content="true">
        <template slot="field">
            <div v-bind:class="editorClass">
                <froala
                    v-if="!loading"
                    :id="field.name"
                    :tag="'textarea'"
                    :config="options"
                    :placeholder="field.name"
                    v-model="value"
                ></froala>
            </div>
        </template>
    </default-field>
</template>

<script>
import { FormField, HandlesValidationErrors } from 'laravel-nova';

import MediaConfigurator from '../MediaConfigurator';
import PluginsLoader from '../PluginsLoader';

export default {
    mixins: [HandlesValidationErrors, FormField],

    beforeDestroy() {
        this.mediaConfigurator.cleanUp();
    },

    mounted() {
        if (typeof window.froala !== 'undefined' && typeof window.froala.events !== 'undefined') {
            _.forEach(window.froala.events, value => {
                value.apply(this);
            });
        }

        const pluginLoader = new PluginsLoader(this.options, this.$toasted);
        Promise.all([
            pluginLoader.registerCustomButtons(),
            pluginLoader.registerPlugins()
        ]).then((data) => {
            this.loading = false;
        });
    },

    data() {
        return {
            loading: true,
            mediaConfigurator: new MediaConfigurator(this.resourceName, this.field, this.$toasted),
            editorClass: 'editor'
        };
    },

    computed: {
        options() {
            this.editorClass = this.field.options.editorClass;
            return _.merge(this.field.options, this.defaultConfig(), window.froala || {});
        },
    },

    methods: {
        fill(formData) {
            formData.append(this.field.attribute, this.value || '');
            formData.append(this.field.attribute + 'DraftId', this.field.draftId);
        },

        /**
         * Additional configurations
         */
        defaultConfig() {
            return this.mediaConfigurator.getConfig();
        },
    },
};
</script>
<style>
    .fr-view {
        border-left: 1px solid #CCCCCC;
        border-right: 1px solid #CCCCCC;
        margin: 0 auto;
    }

    .editor-small .fr-view {
        max-width: 720px;
    }

    .editor-medium .fr-view {
        max-width: 1000px;
    }
</style>
