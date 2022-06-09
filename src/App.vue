<template>
  <div id="app">
    <EmailInput 
    :value.sync="list" 
    :options="options"
    :contactsFieldName="contactsFieldName"
    :optionFormat="optionFormat"
    :tagFormat="tagFormat" style="width:1000px;"/>
    
    <div>下拉框数据格式：</div>
    <div>{{options}}</div>
  </div>
</template>

<script>
import EmailInput from '@/messchow-email-input/index.js'

export default {
  name: 'App',
  components: {
    EmailInput
  },
  data(){
    return {
      list:[ "quincy@freecodecamp.org", "newsletter@newsletter.aliyun.com", "mira@codeanywhere.net" ],
      contactsFieldName: 'contactsName',
			optionFormat: 'contactsName (email) company',
			tagFormat: 'contactsName (email)',
      apiDatas:[
        {
          email:"quincy@freecodecamp.org",
          contactsName:'Kane',
          contactsNameInitials:'KANE',
          company:'company',
          companyInitials:'COMPANY',
        }
      ]
    }
  },
  computed:{
    options() {
			return this.apiDatas.map(o => ({
				value: o.email,
				keyWord: [o.email, o.contactsName, o.contactsNameInitials, o.company, o.companyInitials]
			}))
		},
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
