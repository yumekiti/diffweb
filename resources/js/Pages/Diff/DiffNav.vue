<template>
    <div class="flex space-x-4" v-if="diff">
        <jet-nav-link :href="route('diffs.show', diff.id)" :active="route().current('diffs.show')">Diff</jet-nav-link>
        <jet-nav-link v-if="member.authority <= 0" :href="route('diffs.members', diff.id)" :active="route().current('diffs.members')">メンバー</jet-nav-link>
        <jet-nav-link v-if="member.authority <= 0" :href="route('diffs.invitations', diff.id)" :active="route().current('diffs.invitations')">招待</jet-nav-link>
    </div>
</template>
<script>
import JetNavLink from '@/Jetstream/NavLink'

export default {
    components: {
        JetNavLink
    },
    props: {
        diff: {
            type: Object,
            required: true
        },
        member: {
            type: Object,
            required: true
        }
    }
}
</script>