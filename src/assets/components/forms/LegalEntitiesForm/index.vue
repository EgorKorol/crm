<template lang="pug">
	v-form(ref='form', v-model='valid')
		v-card.elevation-6
			v-card-title.title Данные о сертификате #[v-subheader(v-if='parameters.dateCreation') (создан: {{ parameters.dateCreation | formateDate }})]
			v-container.mb-4(fill-height, fluid, grid-list-xl)
				v-layout(fill-height, wrap, row)
					v-flex(lg6, sm12)
						v-text-field(v-model='parameters.fullName',
												:rules="[v => !!v || 'Введите ФИО']",
												label='ФИО',
												:readonly='isEditCertificate',
												required)
						v-text-field(v-model='parameters.post',
												:rules="[v => !!v || 'Введите должность']",
												label='Должность',
												:readonly='isEditCertificate',
												required)
						v-text-field(v-model='parameters.inn',
												:rules='innRules',
												label='ИНН', type='number',
												:readonly='isEditCertificate',
												required)
						v-text-field(v-model='parameters.kpp',
												:rules="[v => v.length === 9 || 'Введите правильный КПП']",
												label='КПП', max='9'
												:readonly='isEditCertificate',
												required)
						v-text-field(v-model='parameters.ogrn',
												:rules='ogrnRules',
												label='ОГРН',
												:readonly='isEditCertificate',
												required)
						v-text-field(v-model='parameters.companyName',
												:rules="[v => !!v || 'Введите название компании']",
												label='Наименование компании',
												:readonly='isEditCertificate'
												required)
						v-text-field(v-model='parameters.fss',
												:rules="fssRules",
												label='ФСС', type='number',
												:readonly='isEditCertificate')
						v-text-field(v-model='parameters.pfr',
												:rules="pfrRules",
												label='ПФР', type='number',
												:mask="'###-###-######'",
												:readonly='isEditCertificate')
						v-text-field(v-model='parameters.kno',
												label='Код ИФНС (КНО)',
												:readonly='isEditCertificate')
						v-text-field(v-model='parameters.snils',
												:rules='snilsRules',
												label='СНИЛС',
												:readonly='isEditCertificate',
												required)
					v-flex(lg6, sm12)
						v-text-field(v-model='parameters.passport',
												:rules="[v => !!v || 'Введите паспортные данные']",
												hint='Серия, номер, кем выдан, дата выдачи, код подразделения',
												label='Паспортные данные',
												:readonly='isEditCertificate',
												required)
						v-text-field(v-model='parameters.bDate',
												label='Дата рождения',
												:rules="[v => !!v || 'Выберите дату рождения']",
												:readonly='isEditCertificate',
												required)
						v-text-field(v-model='parameters.bAddress',
												:rules="[v => !!v || 'Введите место рождения']",
												label='Место рождения',
												:readonly='isEditCertificate',
												required)
						v-text-field(v-model='parameters.registrationAddress',
												:rules="[v => !!v || 'Введите юридический адрес']",
												label='Юридический адрес',
												:readonly='isEditCertificate',
												required)
						v-text-field(v-model='parameters.email',
												:rules='emailRules',
												type='email'
												label='E-mail',
												:readonly='isEditCertificate',
												required)
						v-text-field(v-model='parameters.phone',
												:rules="[v => !!v || 'Телефон обязателен']",
												label='Контактный телефон', type='tel',
												:readonly='isEditCertificate',
												required)
						file-loader.mb-4(:files='files',
														@setFiles='setFiles',
														@deleteFile='deleteFile')
						file-loader-certificate(v-if='isFileLoaderCertificate')
		v-card.elevation-6.mb-4
			v-card-title.title Параметры изготовления
			v-container(fill-height, fluid, grid-list-xl)
				v-layout(fill-height, wrap)
					v-flex(lg6, sm12)
						v-select(v-model='parameters.groupType',
										:items='items.groupTypes',
										:rules="[v => !!v || 'Выберите ТИП группы']",
										:readonly='isLeadership',
										item-value='id'
										item-text='title'
										label='ТИП группы',
										required)
						v-text-field(v-model='parameters.innGroup',
												:readonly='isLeadership', type='number',
												v-show='isGroup',
												label='ИНН группы')
						v-select(v-model='parameters.numberInGroup',
										:items='items.numberInGroup',
										:readonly='isLeadership',
										item-value='id'
										item-text='title'
										v-show='isGroup',
										label='Количество в группе')
						v-select(v-model='parameters.certType',
										:items='items.certTypes',
										:rules="[v => !!v || 'Выберите тип сертификата']",
										:readonly='isLeadership',
										item-value='id'
										item-text='title'
										label='Тип сертификата',
										required)
						v-select(v-model='parameters.cryptoProvider',
										:items='items.cryptoProviders',
										:rules="[v => !!v || 'Выберите крипто провайдера']",
										:readonly='isLeadership',
										item-value='id'
										item-text='title'
										label='Крипто провайдер',
										required)
						v-select(v-model='parameters.reasonMaking',
										:items='items.reasonsMaking',
										:rules="[v => !!v || 'Выберите причину изготовления']",
										:readonly='isLeadership',
										item-value='id'
										item-text='title'
										label='Причина изготовления',
										required)
						v-select(v-model='parameters.issuance',
										:items='items.issuanceCert',
										:rules="[v => !!v || 'Выберите выдачу сертификата']",
										:readonly='isLeadership',
										item-value='id'
										item-text='title'
										label='Выдача сертификата',
										required)
					v-flex(lg6, sm12)
						v-select(v-model='parameters.status',
										:items='items.statusCert',
										:rules="[v => !!v || 'Выберите текущий статус сертификата']",
										:readonly='isLeadership && isExecutors',
										item-value='id'
										item-text='title'
										label='Текущий статус сертификата',
										required)
						v-select(v-model='parameters.executorsId',
										:items='items.executors',
										:rules="[v => !!v || 'Выберите исполнителя']",
										:readonly='isLeadership',
										item-value='_id'
										item-text='fullName'
										label='Исполнитель',
										required)
						v-select(v-model='parameters.installersId',
										:items='items.installers',
										:rules="[v => !!v || 'Выберите установщика']",
										:readonly='isLeadership',
										item-value='_id'
										item-text='fullName'
										label='Установщик',
										required)
						v-select(v-model='parameters.clientId',
										:items='items.clients',
										:readonly='isLeadership',
										item-value='_id', clearable,
										item-text='fullName'
										label='Контактное лицо')
						v-text-field(v-model='parameters.note',
												:readonly='isLeadership',
												label="Примечание"
												textarea)
						v-checkbox(v-model='checkbox',
											color='teal darken-1',
											v-if='role !== "installer"',
											:rules="[v => !!v || 'Вы должны быть уверены в правильности введенных данных']",
											label='Вы уверены в правильности введенных данных?',
											required)
		v-layout
			v-spacer
			v-btn(:disabled='!valid',
						color='teal lighten-2', large,
						v-if='role !== "installer"',
						@click="$emit('submit', parameters)") Сохранить
</template>

<script>
import fileLoader from '@/assets/components/FileLoader/forFiles'
import fileLoaderCertificate from '@/assets/components/FileLoader/forCertificate'
import axios from 'axios'
import { mapGetters } from 'vuex'
import moment from 'moment'

export default {
	name: 'LegalEntitiesForm',
	components: {
		fileLoader,
		fileLoaderCertificate
	},
	props: {
		parameters: {
			type: Object
		},
		files: {
			type: Array
		}
	},
	data () {
		return {
			valid: false,
			checkbox: false,
			menu: false,
			items: {},
			emailRules: [
				v => !!v || 'E-mail обязателен',
				v => /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/.test(v) || 'E-mail должен быть валидный'
			],
			innRules: [
				v => !!v || 'Введите ИНН',
				v => this.validateInn(v) || 'Не прошло проверку контрольного числа'
			],
			ogrnRules: [
				v => v.length === 13 || 'Не хватает символов',
				v => this.validateOgrn(v) || 'Не прошло проверку контрольного числа'
			],
			snilsRules: [
				v => v.length === 11 || 'Не хватает символов',
				v => this.validateSnils(v) || 'Не прошло проверку контрольного числа'
			],
			fssRules: [
				v => (v.length === 0 || v.length === 10) || 'Введите правильный ФСС'
			],
			pfrRules: [
				v => (v.length === 0 || v.length === 12) || 'Введите правильный ПФР'
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
		isEditCertificate () {
			return this.$route.name === 'EditСertificate'
		},
		isAddCertificate () {
			return this.$route.name === 'NewСertificate'
		},
		isLeadership () {
			if ((this.isEditCertificate || this.isAddCertificate) && (this.role === 'manager' || this.role === 'admin')) {
				return false
			} else {
				return true
			}
		},
		isExecutors () {
			if (this.isEditCertificate && (this.role === 'remoteExecutor' || this.role === 'executor')) {
				return false
			} else {
				return true
			}
		},
		isFileLoaderCertificate () {
			if (this.role !== 'installer' && this.$route.name === 'EditСertificate') {
				return true
			} else {
				return false
			}
		},
		isGroup () {
			if (this.parameters.groupType !== 'none') {
				return true
			} else {
				return false
			}
		}
	},
	methods: {
		async getItemsForSelect () {
			const res = await axios.post(`${this.$root.SERVER_HOST}/certificates.getParametrs`)
			return res.data.items
		},
		validateInn (inn) {
			const checkDigit = (inn, coefficients) => {
				let n = 0
				for (let i in coefficients) {
					n += coefficients[i] * inn[i]
				}
				return parseInt(n % 11 % 10)
			}
			switch (inn.length) {
			case 10:
				let n10 = checkDigit(inn, [2, 4, 10, 3, 5, 9, 4, 6, 8])
				if (n10 === parseInt(inn[9])) {
					return true
				}
				break
			case 12:
				let n11 = checkDigit(inn, [7, 2, 4, 10, 3, 5, 9, 4, 6, 8])
				let n12 = checkDigit(inn, [3, 7, 2, 4, 10, 3, 5, 9, 4, 6, 8])
				if ((n11 === parseInt(inn[10])) && (n12 === parseInt(inn[11]))) {
					return true
				}
				break
			}
			return false
		},
		validateOgrn (ogrn) {
			let n13 = parseInt((parseInt(ogrn.slice(0, -1)) % 11).toString().slice(-1))
			if (n13 === parseInt(ogrn[12])) {
				return true
			} else {
				return false
			}
		},
		validateSnils (snils) {
			let sum = 0
			for (let i = 0; i < 9; i++) {
				sum += parseInt(snils[i]) * (9 - i)
			}
			let checkDigit = 0
			if (sum < 100) {
				checkDigit = sum
			} else if (sum > 101) {
				checkDigit = parseInt(sum % 101)
				if (checkDigit === 100) {
					checkDigit = 0
				}
			}
			if (checkDigit === parseInt(snils.slice(-2))) {
				return true
			} else {
				return false
			}
		},
		setFiles (files) {
			this.$emit('setFiles', files)
		},
		deleteFile (fileIndex) {
			this.$emit('deleteFile', fileIndex)
		}
	},
	async mounted () {
		try {
			this.items = await this.getItemsForSelect()
		} catch (err) {
			this.$store.commit('setError', err)
		}
	}
}
</script>
