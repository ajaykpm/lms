<template>
	<Dialog
		v-model="show"
		:options="{
			title: __('Create a Live Class'),
			size: 'xl',
			actions: [
				{
					label: 'Submit',
					variant: 'solid',
					onClick: (close) => submitLiveClass(close),
				},
			],
		}"
	>
		<template #body-content>
			<div class="flex flex-col gap-4">
				<div class="grid grid-cols-2 gap-4">
					<div>
						<div class="mb-4">
							<div class="mb-1.5 text-sm text-gray-600">
								{{ __('Title') }}
							</div>
							<Input type="text" v-model="liveClass.title" />
						</div>
						<div class="mb-4">
							<div class="mb-1.5 text-sm text-gray-600">
								<Tooltip
									class="flex items-center"
									:text="
										__(
											'Time must be in 24 hour format (HH:mm). Example 11:30 or 22:00'
										)
									"
								>
									<span>
										{{ __('Time') }}
									</span>
									<Info class="stroke-2 w-3 h-3 ml-1" />
								</Tooltip>
							</div>
							<Input v-model="liveClass.time" />
						</div>
						<div>
							<div class="mb-1.5 text-sm text-gray-600">
								{{ __('Timezone') }}
							</div>
							<Select
								v-model="liveClass.timezone"
								:options="getTimezoneOptions()"
							/>
						</div>
					</div>
					<div>
						<div class="mb-4">
							<div class="mb-1.5 text-sm text-gray-600">
								{{ __('Date') }}
							</div>
							<DatePicker v-model="liveClass.date" inputClass="w-full" />
						</div>
						<div class="mb-4">
							<div class="mb-1.5 text-sm text-gray-600">
								<Tooltip
									class="flex items-center"
									:text="__('Duration of the live class in minutes')"
								>
									<span>
										{{ __('Duration') }}
									</span>
									<Info class="stroke-2 w-3 h-3 ml-1" />
								</Tooltip>
							</div>
							<Input type="number" v-model="liveClass.duration" />
						</div>
						<div>
							<div class="mb-1.5 text-sm text-gray-600">
								{{ __('Auto Recording') }}
							</div>
							<Select
								v-model="liveClass.auto_recording"
								:options="getRecordingOptions()"
							/>
						</div>
					</div>
				</div>
				<div>
					<div class="mb-1.5 text-sm text-gray-600">
						{{ __('Description') }}
					</div>
					<Textarea v-model="liveClass.description" />
				</div>
			</div>
		</template>
	</Dialog>
</template>
<script setup>
import {
	Input,
	DatePicker,
	Select,
	Textarea,
	Dialog,
	createResource,
	Tooltip,
} from 'frappe-ui'
import { reactive, inject } from 'vue'
import { getTimezones, createToast } from '@/utils/'
import { Info } from 'lucide-vue-next'

const liveClasses = defineModel('reloadLiveClasses')
const show = defineModel()
const user = inject('$user')
const dayjs = inject('$dayjs')

const props = defineProps({
	batch: {
		type: String,
		default: null,
	},
})

let liveClass = reactive({
	title: '',
	description: '',
	date: '',
	time: '',
	duration: '',
	timezone: '',
	auto_recording: 'No Recording',
	batch: props.batch,
	host: user.data.name,
})

const getTimezoneOptions = () => {
	return getTimezones().map((timezone) => {
		return {
			label: timezone,
			value: timezone,
		}
	})
}

const getRecordingOptions = () => {
	return [
		{
			label: 'No Recording',
			value: 'No Recording',
		},
		{
			label: 'Local',
			value: 'Local',
		},
		{
			label: 'Cloud',
			value: 'Cloud',
		},
	]
}

const createLiveClass = createResource({
	url: 'lms.lms.doctype.lms_batch.lms_batch.create_live_class',
	makeParams(values) {
		return {
			doctype: 'LMS Live Class',
			batch_name: values.batch,
			...values,
		}
	},
})

const submitLiveClass = (close) => {
	createLiveClass.submit(liveClass, {
		validate() {
			if (!liveClass.title) {
				return 'Please enter a title.'
			}
			if (!liveClass.date) {
				return 'Please select a date.'
			}
			if (dayjs(liveClass.date).isSameOrBefore(dayjs(), 'day')) {
				return 'Please select a future date.'
			}
			if (!liveClass.time) {
				return 'Please select a time.'
			}
			if (!valideTime()) {
				return 'Please enter a valid time in the format HH:mm.'
			}
			if (!liveClass.duration) {
				return 'Please select a duration.'
			}
			if (!liveClass.timezone) {
				return 'Please select a timezone.'
			}
		},
		onSuccess() {
			liveClasses.value.reload()
			close()
		},
		onError(err) {
			createToast({
				title: 'Error',
				text: err.messages?.[0] || err,
				icon: 'x',
				iconClasses: 'bg-red-600 text-white rounded-md p-px',
				position: 'top-center',
				timeout: 10,
			})
		},
	})
}

const valideTime = () => {
	let time = liveClass.time.split(':')
	if (time.length != 2) {
		return false
	}
	if (time[0] < 0 || time[0] > 23) {
		return false
	}
	if (time[1] < 0 || time[1] > 59) {
		return false
	}
	return true
}
</script>
