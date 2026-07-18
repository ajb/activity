<!--
  - SPDX-FileCopyrightText: 2022 Nextcloud GmbH and Nextcloud contributors
  - SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
	<NcSettingsSection :name="t('activity', 'Notification')">
		<NcCheckboxRadioSwitch
			v-model="emailEnabled"
			type="checkbox"
			@update:modelValue="toggleEmailEnabled({ emailEnabled: $event })">
			{{ t('activity', 'Enable notification emails') }}
		</NcCheckboxRadioSwitch>
	</NcSettingsSection>
	<NcSettingsSection
		:name="t('activity', 'Excluded file extensions')"
		:description="t('activity', 'Files with these extensions will not appear in the activity log. Enter one extension per line, without the leading dot.')">
		<NcTextArea
			:modelValue="excludedFileExtensions"
			:label="t('activity', 'Excluded file extensions')"
			labelOutside
			:error="hasError"
			:success="isSuccess"
			placeholder="lock&#10;tmp"
			@update:modelValue="onExcludedFileExtensionsInput" />
	</NcSettingsSection>
</template>

<script>
import { t } from '@nextcloud/l10n'
import debounce from 'debounce'
import { mapActions, mapMutations, mapState } from 'vuex'
import NcCheckboxRadioSwitch from '@nextcloud/vue/components/NcCheckboxRadioSwitch'
import NcSettingsSection from '@nextcloud/vue/components/NcSettingsSection'
import NcTextArea from '@nextcloud/vue/components/NcTextArea'
import logger from '../utils/logger.ts'

export default {
	name: 'AdminSettings',
	components: {
		NcCheckboxRadioSwitch,
		NcSettingsSection,
		NcTextArea,
	},

	data() {
		return {
			isSuccess: false,
			hasError: false,
		}
	},

	computed: {
		...mapState({
			emailEnabled: 'emailEnabled',
			excludedFileExtensions: 'excludedFileExtensions',
		}),

		settingDescription() {
			if (this.emailEnabled) {
				return t('activity', 'Choose for which activities you want to get an email or push notification.')
			} else {
				return t('activity', 'Choose for which activities you want to get a push notification.')
			}
		},

		debouncedSaveExcludedFileExtensions() {
			return debounce(async (value) => {
				try {
					await this.setExcludedFileExtensions({ excludedFileExtensions: value })
					this.hasError = false
					this.isSuccess = true
					setTimeout(() => {
						this.isSuccess = false
					}, 2000)
				} catch (error) {
					this.hasError = true
					logger.error('An error occurred while saving the activity settings', { error })
				}
			}, 1000)
		},
	},

	mounted() {
		this.setEndpoint({ endpoint: '/apps/activity/settings/admin' })
	},

	methods: {
		...mapActions([
			'setEndpoint',
			'toggleEmailEnabled',
			'setExcludedFileExtensions',
		]),

		...mapMutations({
			setExcludedFileExtensionsLocally: 'SET_EXCLUDED_FILE_EXTENSIONS',
		}),

		onExcludedFileExtensionsInput(value) {
			this.setExcludedFileExtensionsLocally({ excludedFileExtensions: value })
			this.debouncedSaveExcludedFileExtensions(value)
		},

		t,
	},
}

</script>
