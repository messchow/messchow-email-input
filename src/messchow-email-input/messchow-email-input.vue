<template>
	<div id="email-Input" @mousedown.stop="mousedownEditHandler(undefined)">
		<b class="placeholder" v-if="!isWriteAndHasVal && !list.length">{{ placeholder }}</b>
		<button v-show="false" :class="clipboardClassName" :data-clipboard-text="clipboardText"></button>
		<div class="email-Input">
			<template v-for="(o, i) in renderlist">
				<span
					v-if="i % 2"
					class="tag"
					:class="{ select: hasIn(selectList, o.value) }"
					:ref="'tag' + i"
					contenteditable="false"
					@click.stop
					@mousedown.stop
					@keydown.stop.enter.prevent="inputChange($event, i, o)"
					@blur="blurChange($event, i, o)"
					@dblclick.stop="dbclickHandler($event, o)"
					@mouseover.stop="hoverHandler(i)"
					:key="o.value"
					:value="o.value"
				>
					{{ o.tagLabel }}
				</span>
				<span
					v-else
					class="edit"
					:ref="'edit' + i"
					:key="'edit' + i"
					contenteditable="true"
					@click.stop
					@keydown.stop.enter.prevent="editEnterHandler($event, i)"
					@keydown.stop.delete="deleteHandler($event, i)"
					@keydown.stop.left="leftOrRightHandler(i, 0)"
					@keydown.stop.right="leftOrRightHandler(i, 1)"
					@keydown.stop.up="upOrDownHandler($event, 1)"
					@keydown.stop.down="upOrDownHandler($event, 0)"
					@copy.stop.prevent="copyEditHandler(i)"
					@paste.stop.prevent="pasteEditHandler($event, i)"
					@cut.stop.prevent="cutEditHandler"
					@keydown.stop.ctrl.65.prevent="selectAllEditHandler"
					@mousedown.stop="mousedownEditHandler(i)"
					@focus.stop="focusEditHandler(i)"
					@blur.stop="blurEidtHandler($event, i)"
					@input="editTagChange($event)"
				></span>
			</template>
		</div>
		<div class="email-select" :class="{ show: isShow && renderOptionList.length }">
			<div class="email-opt-select-wrap">
				<ul class="email-opt-select">
					<li
						class="opt"
						v-selectScroll="{ index: i, selectIndex }"
						:class="{ select: hasIn(list, o.value), hover: selectIndex == i }"
						v-for="(o, i) in renderOptionList"
						:key="i"
						@mousedown.stop="selectChange(o, i)"
					>
						<div class="opt-cnt">{{ o.optionLabel }}</div>
						<svg viewBox="64 64 896 896" data-icon="check" width="1em" height="1em" fill="currentColor" aria-hidden="true" focusable="false" class="">
							<path
								d="M912 190h-69.9c-9.8 0-19.1 4.5-25.1 12.2L404.7 724.5 207 474a32 32 0 0 0-25.1-12.2H112c-6.7 0-10.4 7.7-6.3 12.9l273.9 347c12.8 16.2 37.4 16.2 50.3 0l488.4-618.9c4.1-5.1.4-12.8-6.3-12.8z"
							></path>
						</svg>
					</li>
				</ul>
			</div>
		</div>
	</div>
</template>
<script>
import Clipboard from 'clipboard'
export default {
	name:'messchow-email-input',
	props: {
		value: {
			type: Array,
			default: () => []
		},
		options: {
			type: Array,
			default: () => []
		},
		optionFormat: {
			type: String,
			default: 'value'
		},
		optionFormatDef: {
			type: String,
			default: 'value'
		},
		tagFormat: {
			type: String,
			default: 'value'
		},
		tagFormatDef: {
			type: String,
			default: 'value'
		},
		contactsFieldName: {
			//联系人字段名
			type: String,
			default: ''
		},
		placeholder: {
			type: String,
			default: '请输入'
		}
	},
	data() {
		return {
			list: [],
			optionList: [],
			selectList: [],
			focusIndex: undefined,
			mousedownEdit: undefined,
			clipboardClassName: '',
			isCopyPasteCut: false, //当前操作正在剪切复制粘贴
			isTagEntered: false,
			isEditEnter: false,
			isSelectOption: false,
			isWriteAndHasVal: false,
			isShow: false,
			selectIndex: -1
		}
	},
	watch: {
		options:{
			handler(val) {
				this.optionList = val.slice(0)
			},
			deep:true,
			immediate:true
		},
		syncValue: {
			handler(val) {
				let list = val.value.slice()
				list.forEach((one, index) => {
					let obj = this.optionList.find(o => o.value === one)
					if (typeof one == 'string' && !obj) obj = this.newEmail(one)
					list.splice(index, 1, obj)
				})
				this.$nextTick(() => (this.list = list))
			},
			deep: true,
			immediate: true
		},
		isShow(val) {
			!val && (this.selectIndex = -1)
		}
	},
	computed: {
		syncValue() {
			let { value, optionList } = this
			return { value, optionList }
		},
		clipboardText() {
			return this.selectList.map(o => o && o.value).join(';')
		},
		listlastIndex() {
			return this.renderlist.length - 1
		},
		renderlist() {
			let list = JSON.parse(JSON.stringify(this.list))
			let len = list.length
			do {
				list.splice(len, 0, null)
				len--
			} while (len > -1)
			return list
		},
		renderOptionList() {
			let arr = this.optionList.slice(0)
			arr.forEach(o => this.prodLabel(o))
			return arr
		}
	},
	directives: {
		selectScroll: {
			update(el, binding) {
				let { index, selectIndex } = binding.value
				if (index !== selectIndex) return
				let elHeight = el.offsetHeight
				let elOffsetTop = el.offsetTop
				let parentEl = el.parentElement
				let parentElHeight = parentEl.offsetHeight
				let scrollTop = parentEl.scrollTop
				if (elOffsetTop < scrollTop) {
					parentEl.scrollTop = elOffsetTop
				} else if (elOffsetTop + elHeight > scrollTop + parentElHeight) {
					parentEl.scrollTop = elOffsetTop + elHeight - 250
				}
			}
		}
	},
	mounted() {
		this.clipboardInit()
	},
	methods: {
		inputChange(e, index, obj) {
			this.tagsDataUpDate(e, index, obj, e.currentTarget.innerText)
			this.isTagEntered = false
			e.currentTarget.blur()
		},
		blurChange(e, index, obj) {
			e.currentTarget.setAttribute('contenteditable', false)
			this.tagsDataUpDate(e, index, obj, this.isTagEntered ? e.currentTarget.innerText : obj.value)
			this.isTagEntered = false
		},
		tagsDataUpDate(e, index, one, txt) {
			let listIndex = this.rIndexToIndex(index)
			//为空
			// eslint-disable-next-line vue/no-mutating-props
			if (!txt) return this.value.splice(listIndex, 1)
			//不改变值
			if (one.value === txt) {
				e.currentTarget.innerText = one.tagLabel
				return this.editTagFocus(index + 1)
			}
			let obj = this.optionList.find(o => o.value === txt)
			if (obj) {
				//optionList中存在
				let email = this.value.find(o => o === obj.value) //当前已选中
				if (!email) {
			// eslint-disable-next-line vue/no-mutating-props
					this.value.splice(listIndex, 1, txt)
					this.editTagFocus(index - 1)
				}
			} else {
				//optionList不存在
			// eslint-disable-next-line vue/no-mutating-props
				this.value.splice(listIndex, 1, txt)
			}
		},
		dbclickHandler(e, o) {
			let el = e.currentTarget
			el.innerText = o.value
			el.setAttribute('contenteditable', true)
			this.contentableFocusLast(el)
			this.isTagEntered = true
		},
		deleteHandler(e, index) {
			if (e.currentTarget.innerText) return
			let firstIndex = this.renderlist.indexOf(this.selectList[0]) - 1
			let fIndex = this.rIndexToIndex(firstIndex)
			// 全选一次性删除
			if (this.selectList.length) {
			// eslint-disable-next-line vue/no-mutating-props
				this.value.splice(fIndex, this.rIndexToIndex(index) - fIndex)
				this.selectList = []
				this.$forceUpdate()
				this.editTagFocus(firstIndex)
			} else {
				let i = index - 1
				// 点击删除选择前面的标签
				i > 0 && this.selectList.push(this.renderlist[i])
			}
		},
		upOrDownTagHandler(e) {
			this.isShow && e.preventDefault()
		},
		leftOrRightHandler(index, direct) {
			index = direct ? (index < this.listlastIndex ? index + 2 : index) : index ? index - 2 : 0
			this.editTagFocus(index)
		},
		upOrDownHandler(e, direct) {
			this.isShow && e.preventDefault()
			if (direct) {
				this.selectIndex <= -1 ? (this.selectIndex = -1) : this.selectIndex--
			} else {
				let lastIndex = this.optionList.length - 1
				this.selectIndex >= lastIndex ? (this.selectIndex = lastIndex) : this.selectIndex++
			}
		},
		focusEditHandler(i) {
			this.focusIndex = i
		},
		mousedownEditHandler(i) {
			this.selectList = [] //清除选中 场景 点击当前focus
			// 兼容点击input空白处
			if (i === undefined) {
				i = this.listlastIndex
				this.setTimer(this.editTagFocusLast)
			}
			this.mousedownEdit = i
			document.onmouseup = () => {
				this.mousedownEdit = undefined
				document.onmouseup = null
			}
		},
		editEnterHandler(e, index, isFromBlur) {
			let el = e.currentTarget
			let txt = el.innerText
			// 上下选择options
			if (this.isShow && this.selectIndex != -1) {
				return this.selectChange(this.optionList[this.selectIndex], this.selectIndex)
			} else if (!txt) {
				return
			} else if (!this.list.find(o => o.value == txt)) {
			// eslint-disable-next-line vue/no-mutating-props
				this.value.splice(this.rIndexToIndex(index), 0, txt)
				index = index + 2
			}
			el.innerText = ''
			this.isEditEnter = true
			el.blur()
			!isFromBlur && this.setTimer(() => this.editTagFocus(index))
		},
		newEmail(email) {
			let o = {
				value: email,
				tagLabel: '',
				optionLabel: '',
				keyWord: [email]
			}
			this.prodLabel(o)
			return o
		},
		blurEidtHandler(e, i) {
			console.log('blurEidtHandler')
			//当前正在复制粘贴剪切操作 则不清除
			if (!this.isCopyPasteCut) {
				this.focusIndex = undefined
				this.selectList = []
				this.isCopyPasteCut = false
			}
			let txt = e.target.innerText
			//列表选择造成失焦
			if (this.isSelectOption) {
				e.target.innerText = ''
			} else {
				if (!this.isEditEnter) {
					if (txt) {
						//有内容，点击外部失焦
						this.editEnterHandler(e, i, true)
						e.target.innerText = ''
					}
					this.isShow = false
				}
			}
			this.isSelectOption = false
			this.isEditEnter = false
		},
		selectChange(o, i) {
			let index = this.value.findIndex(one => one == o.value)
			// eslint-disable-next-line vue/no-mutating-props
			index >= 0 ? this.value.splice(index, 1) : this.value.push(o.value)
			this.selectIndex = i
			this.isSelectOption = true
			this.setTimer(() => {
				this.editTagFocusLast()
				this.isShow = true
			})
			//点击外部隐藏
			window.onmousedown = () => {
				this.isShow = false
				window.onmousedown = null
			}
		},
		editTagChange(e) {
			let txt = e.target.innerText
			txt && (this.isShow = true)
			this.isWriteAndHasVal = !!txt
			this.optionList.sort((o, n) => {
				let l = o.keyWord.find(w => w.includes(txt)) ? 1 : 0
				let r = n.keyWord.find(w => w.includes(txt)) ? 1 : 0
				return r - l
			})
			this.$forceUpdate()
		},
		copyEditHandler(i) {
			let selectList = JSON.parse(JSON.stringify(this.selectList))
			this.triggerClipboard()
			this.selectList = selectList
			this.editTagFocus(i)
			this.isCopyPasteCut = true
		},
		pasteEditHandler(e, curIndex) {
			if (e.clipboardData || e.originalEvent) {
				let clipboardData = e.clipboardData || window.clipboardData
				let arr = clipboardData
					.getData('text')
					.split(/;|；|,|，/)
					.filter(o => o)
				let diffLen = this.value.length
			// eslint-disable-next-line vue/no-mutating-props
				this.value.splice(this.rIndexToIndex(this.focusIndex + 1), 0, ...arr)
				let value = Array.from(new Set(this.value))
				this.$emit('update:value', value)
				let firstIndex
				if (diffLen != value.length) {
					diffLen = value.length - diffLen
					firstIndex = curIndex + diffLen * 2
				} else {
					firstIndex = curIndex
				}
				setTimeout(() => this.editTagFocus(firstIndex), 0)
			}
		},
		cutEditHandler() {
			let firstIndex = this.renderlist.findIndex(o => o && o.value === this.selectList[0].value) - 1
			this.editTagFocus(firstIndex)
			this.$emit(
				'update:value',
				this.value.filter(o => !this.selectList.map(one => one.value).includes(o))
			)
			this.triggerClipboard()
		},
		selectAllEditHandler() {
			this.selectList = JSON.parse(JSON.stringify(this.list))
		},
		hoverHandler(index) {
			// 长按状态
			if (this.mousedownEdit !== undefined) {
				let fIndex = this.rIndexToIndex(this.focusIndex + 1)
				let rIndex = this.rIndexToIndex(index)
				let firstIndex = 0
				let lastIndex = 0
				if (this.focusIndex > index) {
					firstIndex = rIndex
					lastIndex = fIndex
				} else {
					firstIndex = fIndex
					lastIndex = rIndex + 1
				}
				this.selectList = this.list.slice(firstIndex, lastIndex)
			}
		},
		rIndexToIndex(index) {
			if (index % 2 == 1) index--
			return (index = index >= 2 ? index / 2 : 0)
		},
		contentableFocusLast(el) {
			el.focus()
			document.execCommand('selectAll', false, null)
			document.getSelection().collapseToEnd()
		},
		hasIn(list, value) {
			return !!list.find(o => o.value === value)
		},
		setTimer(func, wait = 0) {
			let timer = setTimeout(() => {
				func()
				clearTimeout(timer)
			}, wait)
		},
		prodLabel(o) {
			let hasContacts = this.contactsFieldName && o[this.contactsFieldName]
			let option = hasContacts ? this.optionFormat : this.optionFormatDef
			let tag = hasContacts ? this.tagFormat : this.tagFormatDef
			Object.keys(o).forEach(key => {
				option = option.replace(key, o[key])
				tag = tag.replace(key, o[key])
			})
			o.optionLabel = option
			o.tagLabel = tag
		},
		editTagFocus(index) {
			this.$nextTick(() => this.$refs[`edit${index}`][0].focus())
		},
		editTagFocusLast() {
			this.editTagFocus(this.listlastIndex)
		},
		editTagBlur(index) {
			this.$refs[`edit${index}`][0].blur()
		},
		ClearCurEditTagCnt() {
			this.$refs[`edit${this.focusIndex}`][0].innerText = ''
		},
		clipboardInit() {
			this.clipboardClassName = 'clipboard' + +new Date()
			let clipboard = new Clipboard(`.${this.clipboardClassName}`)
			clipboard.on('success', e => console.log('success', e))
			clipboard.on('error', e => console.log('success', e))
		},
		triggerClipboard() {
			document.querySelector(`.${this.clipboardClassName}`).click()
		}
	}
}
</script>
<style lang="scss" scoped>
#email-Input {
	position: relative;
	box-sizing: border-box;
	text-align: left;
	ul,
	li {
		list-style: none;
		outline: none;
		padding: 0;
		margin: 0;
	}
	.placeholder {
		position: absolute;
		height: 28px;
		line-height: 28px;
		padding-left: 4px;
		color: #bfbfbf;
		font-weight: normal;
		z-index: 1;
		select: none;
	}
	.email-Input {
		min-height: 20px;
		max-height: 80px;
		font-size: 14px;
		border: 1px solid #d9d9d9;
		border-radius: 4px;
		vertical-align: middle;
		padding: 0 5px;
		user-select: none;
		text-align: left;
		padding-bottom: 3px;
		overflow: auto;
	}

	.tag {
		margin-top: 3px;
		height: 20px;
		line-height: 18px;
		color: rgba(0, 0, 0, 0.65);
		padding: 0 8px;
		display: inline-block;
		white-space: nowrap;
		background: #fafafa;
		border: 1px solid #d9d9d9;
		box-sizing: border-box;
		cursor: pointer;
		&:focus-visible {
			border: 1px solid #3385ff;
			outline: #3385ff solid 1px;
			cursor: text;
		}
		&.select {
			border: 1px solid #3385ff;
			background-color: #3385ff;
			color: #fff;
		}
	}

	.edit {
		margin-top: 3px;
		height: 20px;
		line-height: 20px;
		color: rgba(0, 0, 0, 0.65);
		display: inline-block;
		padding: 0 3px;
		margin-top: 3px;
		&:focus-visible {
			outline: 0;
		}
		&:first-of-type {
			margin-left: -6px;
		}
	}

	.email-select {
		position: absolute;
		bottom: 0;
		left: 0;
		right: 0;
		max-height: 0;
		overflow: hidden;
		transform: translateY(100%);
		z-index: 100;
		background-color: #fff;
		box-shadow: 0 2px 8px rgb(0 0 0 / 15%);
		border-radius: 4px;
		font-size: 14px;
		transition: max-height 0.5s ease-in-out;
		&.show {
			max-height: 250px;
		}
	}

	.email-opt-select-wrap {
		padding: 4px 0;
	}
	.email-opt-select {
		overflow: auto;
		max-height: 242px;
	}
	.opt {
		position: relative;
		display: flex;
		align-items: center;
		padding: 5px 12px;
		overflow: hidden;
		color: rgba(0, 0, 0, 0.65);
		font-weight: normal;
		font-size: 14px;
		line-height: 22px;
		white-space: nowrap;
		text-overflow: ellipsis;
		cursor: pointer;
		transition: background 0.3s ease;
		&.select {
			font-weight: bold;
			color: rgba(0, 0, 0, 0.65);
			font-weight: 600;
			background-color: #fafafa;
		}
		&.select {
			svg {
				display: inline-block !important;
				color: #1890ff;
			}
		}
		&.hover,
		&:hover {
			background-color: #e6f7ff;
		}
		&:not(:hover) svg {
			display: none;
		}
		svg {
			transition: all 0.2s;
			font-weight: bold;
			font-size: 12px;
			text-shadow: 0 0.1px 0, 0.1px 0 0, 0 -0.1px 0, -0.1px 0;
		}
	}
	.opt-cnt {
		flex: 1;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}
}
</style>
