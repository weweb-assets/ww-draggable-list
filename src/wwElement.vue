<template>
    <div v-if="!items">Please bind variable</div>
    <draggable
        v-else
        :list="items"
        :group="{ name: group }"
        :item-key="itemKey"
        ghost-class="ghost"
        :disabled="isEditing"
    >
        <template #item="{ element, index }">
            <div>
                <wwLayoutItemContext :index="index" :item="{}" is-repeat :data="element">
                    <wwLayout path="itemContent"></wwLayout>
                </wwLayoutItemContext>
            </div>
        </template>
    </draggable>
</template>

<script>
import draggable from "vuedraggable";

export default {
    components: {
        draggable,
    },
    props: {
        content: { type: Object, required: true },
        wwFrontState: { type: Object, required: true },
        wwElementState: { type: Object, required: true },
        /* wwEditor:start */
        wwEditorState: { type: Object, required: true },
        /* wwEditor:end */
    },
    emits: ["update:content", "update:content:effect", "trigger-event", "element-event"],
    setup() {
        return { internalGroup: wwLib.wwUtils.getUid(), items: ref([]) };
    },
    watch: {
        internalItems(value) {
            if (!_.isEqual(value, this.items)) {
                this.items = value;
            }
        },
        items(value) {
            if (!_.isEqual(value, this.internalItems)) {
                this.internalItems = value;
            }
        },
    },
    computed: {
        isEditing() {
            /* wwEditor:start */
            return this.wwEditorState.editMode === wwLib.wwEditorHelper.EDIT_MODES.EDITION;
            /* wwEditor:end */
            // eslint-disable-next-line no-unreachable
            return false;
        },
        internalItems: {
            get() {
                if (!this.content.variableId && !(this.wwElementState.props && this.wwElementState.props.items)) {
                    return null;
                }
                const data =
                    (this.wwElementState.props && this.wwElementState.props.items) ||
                    wwLib.wwVariable.getValue(this.content.variableId);
                if (!Array.isArray(data)) {
                    return null;
                } else {
                    return data;
                }
            },
            set(items) {
                this.$emit("trigger-event", { name: "change", event: { value: items } });
                this.$emit("element-event", { name: "change", event: { value: items } });
                if (this.content.variableId) wwLib.wwVariable.updateValue(this.content.variableId, items);
            },
        },
        group() {
            return (this.wwElementState.props && this.wwElementState.props.group) || this.internalGroup;
        },
        itemKey() {
            return (this.wwElementState.props && this.wwElementState.props.itemKey) || this.content.itemKey;
        },
    },
};
</script>

<style scoped>
.ghost {
    opacity: 0.5;
    background: #c8ebfb;
}
.not-draggable {
    cursor: no-drop;
}
</style>
