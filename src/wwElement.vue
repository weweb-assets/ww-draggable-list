<template>
    <div v-if="!items">Please bind variable</div>
    <draggable v-else v-model="items" :group="group" :item-key="content.itemKey" ghost-class="ghost" :disabled="isEditing">
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
        wwProps: { type: Object, required: true },
        /* wwEditor:start */
        wwEditorState: { type: Object, required: true },
        /* wwEditor:end */
    },
    emits: ["update:content", "update:content:effect", "trigger-event"],
    setup() {
        return { internalGroup: wwLib.wwUtils.getUid() };
    },
    computed: {
        isEditing() {
            /* wwEditor:start */
            return this.wwEditorState.editMode === wwLib.wwEditorHelper.EDIT_MODES.EDITION;
            /* wwEditor:end */
            // eslint-disable-next-line no-unreachable
            return false;
        },
        items: {
            get() {
                if (!this.content.variableId) {
                    return null;
                }
                const data = wwLib.wwVariable.getValue(this.content.variableId);
                if (!Array.isArray(data)) {
                    return null;
                } else {
                    return data;
                }
            },
            set(items) {
                this.$emit("trigger-event", { name: "change", event: { value: items } });
                if (this.content.variableId) wwLib.wwVariable.updateValue(this.content.variableId, items);
            },
        },
        group() {
          return wwProps.group || this.internalGroup
        }
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