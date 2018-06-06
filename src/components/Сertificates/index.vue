<template lang="pug">
	v-container(fluid, align-content-start, justify-start)
		v-expansion-panel.mb-3
			v-expansion-panel-content
				div.title(slot="header") Фильтр
				certificate-filter(:statuses='statuses'
													@update='updateCertificates')
		v-card.elevation-1.pa-2
			v-card-title.title Сертификаты
				v-layout(row, wrap, justify-space-between, align-center)
					v-flex(lg4, md5, sm12)
						v-menu(transition='slide-x-transition', bottom, right, v-if='isDisable')
							v-btn(small, slot='activator', color='teal', dark) Добавить
								v-icon(dark) add
							v-list
								v-list-tile(@click='$router.push({name: "NewСertificate", params: {type: "individual"} })')
									v-list-tile-title Физическое лицо
								v-list-tile(@click='$router.push({name: "NewСertificate", params: {type: "legal"} })')
									v-list-tile-title Юридическое лицо
					v-flex(lg4, md5, sm12)
						v-text-field(v-model='search', append-icon='search', label='Поиск...')
			v-data-table(:headers='headers',
									:items='certificates',
									:loading='isLoading',
									:search='search',
									no-results-text='Ничего не найдено',
									no-data-text='Нет данных',
									:rows-per-page-items='[10,25,{"text":"Все","value":-1}]'
									rows-per-page-text='Строк на странице',
									disable-initial-sort)
				v-progress-linear(slot='progress', color='blue darken-3', indeterminate)
				template(slot='items', slot-scope='props')
					td {{ props.item.fullName }}
					td.text-xs-center {{ props.item.companyName ? props.item.companyName : 'Физ.лицо'}}
					td.text-xs-center {{ props.item.inn }}
					td.text-xs-center {{ props.item.dateCreation | formateDate }}
					td.text-xs-center {{ props.item.client[0].phone ? props.item.client[0].phone : '' }}
					td.text-xs-center {{ props.item.client[0].email ? props.item.client[0].email : '' }}
					td.text-xs-center
						v-chip(:color='setColorStatus(props.item.status)', small) {{ formatStatuses(props.item.status) }}
					td.text-xs-center
						v-btn.mx-0(icon, @click='editCertificate(props.item)')
							v-icon(color='teal darken-1') edit
				template(slot='no-data')
					.headline.text-xs-center.pa-2(v-if='!certificates.length') Нет добавленных сертификатов
					v-alert(:value='true', v-if='isError', color='error', icon='warning') Похоже проихошла ошибка, обновите страницу :(
				template(slot='pageText', slot-scope='{ pageStart, pageStop }') c {{ pageStart }} по {{ pageStop }}
</template>

<script>
import axios from 'axios'
import { mapGetters } from 'vuex'
import moment from 'moment'

export default {
	name: 'Certificates',
	components: {
		CertificateFilter: () => import('@/assets/components/forms/CertificateFilter')
	},
	data () {
		return {
			search: '',
			isLoading: true,
			isError: false,
			statuses: [],
			headers: [
				{ text: 'ФИО владельца', align: 'left', value: 'fullName' },
				{ text: 'Организация', value: 'companyName', align: 'center' },
				{ text: 'ИНН', value: 'inn', align: 'center' },
				{ text: 'Дата создания', value: 'dateCreation', align: 'center' },
				{ text: 'Контактный телефон', sortable: false, value: 'phone', align: 'center' },
				{ text: 'Контактный email', sortable: false, value: 'email', align: 'center' },
				{ text: 'Статус', value: 'status', align: 'center' },
				{ text: 'Действия', sortable: false, align: 'center' }
			],
			certificates: [],
			statusesColor: [
				{ id: 'work', color: 'blue lighten-4' },
				{ id: 'reg', color: 'purple lighten-4' },
				{ id: 'done', color: 'pink lighten-4' },
				{ id: 'issued', color: 'green lighten-4' },
				{ id: 'problem', color: 'deep-orange lighten-4' },
				{ id: 'is_formed', color: 'yellow lighten-4' }
			]
		}
	},
	filters: {
		formateDate (value) {
			if (value) {
				return moment(String(value)).format('DD.MM.YYYY, HH:mm')
			}
		}
	},
	computed: {
		...mapGetters(['role']),
		isDisable () {
			if (this.role === 'remoteExecutor' || this.role === 'executor' || this.role === 'installer') {
				return false
			} else {
				return true
			}
		}
	},
	methods: {
		editCertificate (item) {
			this.$store.commit('setItemSelectedCertificate', item)
			this.$router.push({name: 'EditСertificate'})
		},
		formatStatuses (value) {
			for (const iterator of this.statuses) {
				if (iterator.id === value) return iterator.title
			}
		},
		setColorStatus (value) {
			for (const iterator of this.statusesColor) {
				if (iterator.id === value) return iterator.color
			}
		},
		async getItemsForSelect () {
			const res = await axios.post(`${this.$root.SERVER_HOST}/certificates.getParametrs`)
			return res.data.items.statusCert
		},
		async getCertificates () {
			const res = await axios.post(`${this.$root.SERVER_HOST}/certificates.get`)
			return res.data.certificates
		},
		async updateCertificates (parameters) {
			try {
				this.isLoading = true
				const res = await axios.post(`${this.$root.SERVER_HOST}/certificates.get`, parameters)
				this.certificates = res.data.certificates
				this.isLoading = false
			} catch (err) {
				this.$store.commit('setError', err)
				this.$store.commit('showErorrMessage', 'Произошла ошибка при обновлении')
				this.isLoading = false
			}
		}
	},
	async mounted () {
		try {
			this.isLoading = true
			this.certificates = await this.getCertificates()
			this.statuses = await this.getItemsForSelect()
			this.isLoading = false
		} catch (err) {
			this.$store.commit('setError', err)
			this.$store.commit('showErorrMessage', 'Произошла ошибка при загрузке')
			this.isError = true
			this.isLoading = false
		}
	}
}
</script>
