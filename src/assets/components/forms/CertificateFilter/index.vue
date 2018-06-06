<template lang="pug">
	v-card
		v-form(v-model='validFormDate')
			v-container(fluid, grid-list-xl)
				v-layout
					v-flex
						v-btn(small, color='blue', dark, @click='setYesterday') Вчера
						v-btn(small, color='blue', dark, @click='setToday') Сегодня
						v-btn(small, color='blue', dark, @click='setWeek') Неделя
						v-btn(small, color='blue', dark, @click='setMounth') Месяц
				v-layout(row, wrap)
					v-flex(lg3, md4, xs12)
						v-menu(:close-on-content-click='false',
									v-model='menu1',
									:nudge-right='40',
									lazy, offset-y, full-width,
									transition='scale-transition',
									max-width='290px', min-width='290px')
							v-text-field(slot='activator',
													v-model='computedDateFromFormatted',
													label='Начальная дата',
													clearable,
													persistent-hint, prepend-icon='event', readonly)
							v-date-picker(v-model='parameters.dateFrom',
														@input='handlerPickerFrom(parameters.dateFrom)',
														:max="new Date().toISOString().substr(0, 10)",
														locale='ru-ru')
					v-flex(lg3, md4, xs12)
						v-menu(:close-on-content-click='false',
									v-model='menu2',
									:nudge-right='40',
									lazy, offset-y, full-width,
									transition='scale-transition',
									max-width='290px', min-width='290px')
							v-text-field(slot='activator',
													v-model='computedDateToFormatted',
													clearable,
													label='Конечная дата',
													persistent-hint, prepend-icon='event', readonly)
							v-date-picker(v-model='parameters.dateTo',
														@input='handlerPickerTo(parameters.dateTo)',
														:min='parameters.dateFrom',
														:max="new Date().toISOString().substr(0, 10)",
														locale='ru-ru')
					v-flex(lg3, md4, xs12)
						v-select(v-model='parameters.status',
										:items='statuses',
										item-value='id',
										clearable,
										item-text='title',
										prepend-icon='timeline',
										label='Статус сертификата')
					v-flex(lg3, md4, xs12)
						v-text-field(v-model='parameters.inn',
												prepend-icon='list', clearable,
												label='ИНН')
					v-flex(xs12)
						v-btn.white--text(small, color='deep-purple darken-3',
															@click='sendDate', :disabled='isDisabled') Обновить
</template>

<script>
import moment from 'moment'

export default {
	name: 'CertificateFilter',
	props: {
		statuses: {
			type: Array
		}
	},
	data () {
		return {
			menu1: false,
			menu2: false,
			validFormDate: false,
			dateFrom: null,
			dateTo: null,
			parameters: {
				dateFrom: null,
				dateTo: null,
				status: null,
				inn: null
			}
		}
	},
	computed: {
		isDisabled () {
			if (this.parameters.dateFrom || this.parameters.dateTo || this.parameters.status || this.parameters.inn) {
				return false
			} else {
				return true
			}
		},
		computedDateFromFormatted: {
			get () {
				return this.formatDateFrom(this.parameters.dateFrom)
			},
			set (value) {
				return this.formatDateFrom(value)
			}
		},
		computedDateToFormatted: {
			get () {
				return this.formatDateTo(this.parameters.dateTo)
			},
			set (value) {
				return this.formatDateTo(value)
			}
		}
	},
	watch: {
		date (val) {
			this.dateFormatted = this.formatDate(this.date)
		}
	},
	methods: {
		formatDateFrom (date) {
			if (!date) {
				this.parameters.dateFrom = null
				return null
			}

			const [year, month, day] = date.split('-')
			return `${day}.${month}.${year}`
		},
		formatDateTo (date) {
			if (!date) {
				this.parameters.dateTo = null
				return null
			}

			const [year, month, day] = date.split('-')
			return `${day}.${month}.${year}`
		},
		sendDate () {
			let params = Object.assign({}, this.parameters)
			if (params.dateFrom) {
				params.dateFrom = this.dateFrom.toDate()
			}
			if (params.dateTo) {
				params.dateTo = this.dateTo.toDate()
			}
			this.$emit('update', params)
		},
		setToday () {
			const todayStart = moment().startOf('day')
			const todayNow = moment()
			this.dateFrom = todayStart
			this.dateTo = todayNow
			this.parameters.dateFrom = todayStart.format().substr(0, 10)
			this.parameters.dateTo = todayNow.format().substr(0, 10)
		},
		setYesterday () {
			const yesterdayStart = moment().subtract(1, 'days').startOf('day')
			const yesterdayEnd = moment().subtract(1, 'days').endOf('day')
			this.dateFrom = yesterdayStart
			this.dateTo = yesterdayEnd
			this.parameters.dateFrom = yesterdayStart.format().substr(0, 10)
			this.parameters.dateTo = yesterdayEnd.format().substr(0, 10)
		},
		setWeek () {
			const weekAgo = moment().subtract(7, 'days').startOf('day')
			const todayNow = moment()
			this.dateFrom = weekAgo
			this.dateTo = todayNow
			this.parameters.dateFrom = weekAgo.format().substr(0, 10)
			this.parameters.dateTo = todayNow.format().substr(0, 10)
		},
		setMounth () {
			const mounthAgo = moment().subtract(30, 'days').startOf('day')
			const todayNow = moment()
			this.dateFrom = mounthAgo
			this.dateTo = todayNow
			this.parameters.dateFrom = mounthAgo.format().substr(0, 10)
			this.parameters.dateTo = todayNow.format().substr(0, 10)
		},
		handlerPickerFrom (dateFrom) {
			this.menu1 = false
			this.dateFrom = moment(dateFrom)
		},
		handlerPickerTo (dateTo) {
			this.menu2 = false
			this.dateTo = moment(dateTo)
		}
	}
}
</script>
