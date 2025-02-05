<!-- Edited Version for src/components/UserDropdown.vue -->
<template>
	<Dropdown class="p-2" :options="userDropdownOptions">
		<template v-slot="{ open }">
			<button
				class="flex h-12 py-2 items-center rounded-md duration-300 ease-in-out text-sm font-medium text-gray-900 leading-none"
				:class="
					isCollapsed
						? 'px-0 w-auto'
						: open
						? 'bg-white shadow-sm px-2'
						: 'hover:bg-gray-200 px-2'
				"
			>
                    <UserRound class="size-5 mr-1" /> Profile
            </button>
		</template>
	</Dropdown>
	<SettingsModal
		v-if="userResource.data?.is_moderator"
		v-model="showSettingsModal"
	/>
</template>

<script setup>
import LMSLogo from '@/components/Icons/LMSLogo.vue'
import { sessionStore } from '@/stores/session'
import { Dropdown } from 'frappe-ui'
import Apps from '@/components/Apps.vue'
import { ChevronDown, LogIn, LogOut, User, Settings, UserRound } from 'lucide-vue-next'
import { useRouter } from 'vue-router'
import { convertToTitleCase } from '@/utils'
import { usersStore } from '@/stores/user'
import { useSettings } from '@/stores/settings'
import { markRaw, watch, ref } from 'vue'
import SettingsModal from '@/components/Modals/Settings.vue'

const router = useRouter()
const { logout, branding } = sessionStore()
let { userResource } = usersStore()
const settingsStore = useSettings()
let { isLoggedIn } = sessionStore()
const showSettingsModal = ref(false)

const props = defineProps({
	isCollapsed: {
		type: Boolean,
		default: false,
	},
})

watch(
	() => settingsStore.isSettingsOpen,
	(value) => {
		showSettingsModal.value = value
	}
)

const userDropdownOptions = [
	{
		icon: User,
		label: 'My Profile',
		onClick: () => {
			router.push(`/user/${userResource.data?.username}`)
		},
		condition: () => {
			return isLoggedIn
		},
	},
	{
		component: markRaw(Apps),
		condition: () => {
			let cookies = new URLSearchParams(document.cookie.split('; ').join('&'))
			let system_user = cookies.get('system_user')
			if (system_user === 'yes') return true
			else return false
		},
	},
	{
		icon: Settings,
		label: 'Settings',
		onClick: () => {
			settingsStore.isSettingsOpen = true
		},
		condition: () => {
			return userResource.data?.is_moderator
		},
	},
	{
		icon: LogOut,
		label: 'Log out',
		onClick: () => {
			logout.submit().then(() => {
				isLoggedIn = false
			})
		},
		condition: () => {
			return isLoggedIn
		},
	},
	{
		icon: LogIn,
		label: 'Log in',
		onClick: () => {
			window.location.href = '/login'
		},
		condition: () => {
			return !isLoggedIn
		},
	},
]
</script>
