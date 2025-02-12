<template>
    <div class="d-flex flex-column gap-4">
        <slot v-if="description !== undefined" name="markdown" :content="description"/>
        <!-- Root plugin page with subgroups -->
        <template v-if="subGroup === undefined && plugins.length > 1">
            <div class="d-flex flex-column">
                <RowLink v-for="subGroupWrapper in subGroupsWrappers"
                         :id="`group-${slugify(subGroupName(subGroupWrapper))}`"
                         :iconB64Svg="'data:image/svg+xml;base64,' + icons[subGroupWrapper.subGroup]"
                         :text="subGroupName(subGroupWrapper)"
                         :href="subGroupHref(subGroupName(subGroupWrapper))"
                         @click="$emit('goTo', {targetSubGroup: subGroupWrapper.subGroup})"
                />
            </div>
        </template>
        <template v-else>
            <div class="d-flex flex-column elements-section" v-for="(elements, elementType) in elementsByType">
                <h4 :id="`section-${slugify(elementType)}`">{{ elementType }}</h4>
                <div class="d-flex flex-column">
                    <RowLink v-for="element in elements"
                             :id="slugify(element)"
                             :iconB64Svg="'data:image/svg+xml;base64,' + icons[element]"
                             :text="elementName(element)"
                             :href="elementHref(element)"
                             @click="$emit('goTo', {targetElement: element})"
                    />
                </div>
            </div>
        </template>
    </div>
</template>
<script setup lang="ts">
    import RowLink from "./RowLink.vue";
    import type {Plugin} from "~/utils/plugins";
    import {isEntryAPluginElementPredicate, subGroupName} from "~/utils/plugins";
    import {slugify} from "~/utils/url.js";

    const props = defineProps<{
        plugins: Plugin[],
        pluginName: string,
        subGroup?: string | undefined,
        icons: Record<string, string>
    }>();

    const plugin = computed(() => props.plugins.find(p => props.subGroup === undefined ? true : (slugify(subGroupName(p)) === props.subGroup)));

    const description = computed(() => plugin.value?.longDescription ?? plugin.value?.description);

    const subGroupsWrappers = computed(() => {
        return props.plugins
            .filter(p => p.name.toLowerCase() === props.pluginName.toLowerCase() && p.subGroup !== undefined);
    });

    const elementName = (qualifiedName) => {
        let split = qualifiedName.split(".");
        return split?.[split.length - 1]?.capitalize();
    }

    const {path} = useRoute();

    const subGroupHref = (targetSubGroup) => `${path}/${slugify(targetSubGroup)}`;

    const elementHref = (element) => `${path}/${element}`;

    const extractPluginElements: Record<string, string[]> = (plugin: Plugin) => {
        return Object.fromEntries(
            Object.entries(plugin).filter(([key, value]) => isEntryAPluginElementPredicate(key, value))
                .map(([key, value]) => [key.replaceAll(/[A-Z]/g, match => ` ${match}`).capitalize(), value])
        );
    };

    const elementsByType = computed(() => extractPluginElements(plugin.value));

    defineEmits<{
        goTo: [targetElement: string, targetSubGroup?: string]
    }>()
</script>