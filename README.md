# messchow-email-input

类似QQ邮箱的邮箱输入框，增删改查、根据输入给邮箱自动排序。

## install

```js
npm i messchow-email-input -s
```

## use

```js
import emailInput from 'messchow-email-input'
import 'messchow-email-input/lib/messchow-email-input.css'
Vue.use(emailInput)
```
## props parameter

| props | type | introduce |
|----------------|----------------|----------------|
|`value.sync`|Array|邮箱初始化数组|
|`contactsFieldName`|String|接口返回数据中,`联系人的字段名`|
|`optionFormat`|String|input中下拉框`option`的`label`的数据格式，列如`contactsName (email)`中，`contactsName`、`email`都为接口返回对象的字段名|
|`tagFormat`|String|input中`tag`的`label`的数据格式，同理`optionFormat`|
|`options`|Array|下拉框的数组,如果`value.sync`中的邮箱和`options`匹配，会自动选中|

### `options`的格式
```js
//后台返回的接口数据
apiDatas:[
  {
    email:"quincy@freecodecamp.org", //邮箱地址
    contactsName:'Kane', //联系人名称
    contactsNameInitials:'KANE', //联系人名称大写（自选）
    company:'company', //联系人公司名称 （自选）
    companyInitials:'COMPANY', //公司名称大写 （自选）
  }，
  ....
]

//转换数据格式
let options = this.apiDatas.map(o => ({
  value: o.email,
  keyWord: [o.email, o.contactsName, o.contactsNameInitials, o.company, o.companyInitials] //关键字搜索，input 输入会根据 keyWord 字段去排序
}))

```
