<template>
    <div class="widget-panel" v-if="context=='editor'">
        <div class="editor widget-label">{{widget.label}}</div>
        <model-select
            v-model="selectedOption"
            :options="options"
            :placeholder="placeholder"
            @input="onChange">
        </model-select>
    </div>
    <ons-row class="report-widget" v-else-if="context=='report'">
        <ons-col>
            <span class="report widget-label">{{widget.label}}</span>
            <span class="report widget-value">{{displayValue}}</span>
        </ons-col>
    </ons-row>
    <span class="flex tile-data" v-else-if="context=='nav'">
        <div v-if="!!displayValue">{{displayValue}}</div>
        <div v-else>no data</div>
        <div class="widget-label">{{widget.label}}</div>
    </span>
</template>

<script>

export default {
    name: 'NodeValueWidget',
    props: ['value', 'widget', 'node', 'tiles', 'context'],
    data() {
        return {
            placeholder: this.widget.config.placeholder,
            local_value: this.value,
            user: this.$store.getters.activeServer.user,
            allWidgets: this.$store.getters.activeGraph.widgets
        };
    },
    computed: {
        options() {
            var self = this;
            var ret = [];

            this.tiles.forEach(function(tile){
                var found = false;
                if (!!tile.provisionaledits && tile.provisionaledits.hasOwnProperty(self.user.id) && !!tile.provisionaledits[self.user.id]) {
                    var provisionaledit = tile.provisionaledits[self.user.id]['value'];
                    if (provisionaledit.hasOwnProperty(self.node.config.nodeid)) {
                        ret.push({
                            value: tile.tileid,
                            text: self.displayLabel + ': ' + provisionaledit[self.node.config.nodeid],
                            depth: 1
                        })
                        found = true;
                    }
                }
                if (!found) {
                    if (tile.data.hasOwnProperty(self.node.config.nodeid)) {
                        ret.push({
                            value: tile.tileid,
                            text: self.displayLabel + ': ' + tile.data[self.node.config.nodeid],
                            depth: 1
                        })
                    }
                }
            })
            return ret;
        },
        selectedOption: {
            get: function() {
                var self = this;
                var ret = {};
                this.options.forEach(function(option){
                     if(option.value === self.local_value){
                        ret = option;
                    }
                })
                return ret;

            },
            set: function() {

            }
        },
        displayValue: function() {
            return this.selectedOption.text;
        },
        displayLabel: function() {
            var self = this;
            var widget = this.$underscore.filter(this.allWidgets, function(widget) {
                return widget.node_id === self.node.config.nodeid;
            }, this);
            return widget[0].label;
        }
    },
    methods: {
        onChange(option) {
            var ret = null
            if(!!option) {
                ret = option.value;
            }
            this.local_value = ret;
            this.$emit('update:value', ret);
        }
    }
};
</script>

<style scoped>
.widget-value {
  padding-left: 5px;
}

.widget-panel {
    padding-bottom: 25px;
    background: #fbfbfb;
    border-bottom: 1px solid #ddd;
}

.widget-panel.widget {
    padding-bottom: 25px;
}

.widget-label {
  font-weight: 400;
  color: #271F4C;
  padding-right: 5px;
}

.report-widget {
    padding-bottom: 3px;
}

.tile-data {
    background: #fafafa;
    color: #888;
    margin-left: -5px;
    padding-left: 5px;
}

.tile-data .widget-label {
    color: #271F4C;
    font-size: 13px;
    padding-bottom: 10px;
}
</style>
