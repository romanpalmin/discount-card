<template>
  <div>
    <div class="password-page-content">
      <div class="row-item title">
        ВВЕДИТЕ КОД ИЗ SMS:
      </div>
      <div class="row-item input">
        <input id="password" type="number" class="password-input" v-model="currentPassword"/>
      </div>
      <div class="row-item button">
        <button :class="continueButtonClass" @click="send()">продолжить</button>
      </div>
      <div class="row-item prompt">
        <span v-if="showPrompt">Неправильный код из СМС</span>
      </div>
      <modal v-if="showModal"
             :isLoader=true
             :alertText="modalText"
             :alertTitle="modalTitle"
             :showButton="modalButton"
             :showAlert="modalAlert"
             :showPreloader="modalPreloader"
      />
    </div>
  </div>
</template>
<style scoped lang="less">
  .password-page-content {
    margin-top: 42%;
    .row-item {
      &.title {
        font-size: 30pt;
        font-weight: 700;
        letter-spacing: 1px;
        font-family: IntroHeader;
      }
      &.input {
        input {
          height: 40pt;
          border: 3px solid #cbcbcb;
          border-radius: 8px;
          width: 230px;
          background-color: black;
          margin-top: 50px;
          text-align: center;
          color: white;
          font-size: 25pt;
          font-weight: 900;
        }
      }
      &.button {
        button {
          height: 40pt;
          border: 3px solid white;
          border-radius: 8px;
          width: 230px;
          margin-top: 30px;
          text-align: center;
          font-size: 20pt;
          font-weight: 900;
          font-family: IntroHeader;
          &.ready {
            background-color: #ffffff;
            color: #000000;
          }
          &.not-ready {
            background-color: #000000;
            color: #484848;
          }
        }
      }
      &.prompt {
        span {
          color: #f30000;
          font-size: 18pt;
          font-weight: 900;
        }
      }
    }
  }
</style>
<script>
  import InputMask from 'inputmask';
  import MaskedInput from 'vue-masked-input';
  import modal from '../components/modal.vue';
  import store from '../store';
  export default {
    data() {
      return {
        name: 'this component',
        showPrompt: false,
        passwordLength: 4,
        currentPassword: '',
        modalText: '',
        modalTitle: '',
        modalButton: false,
        modalAlert: false,
        modalPreloader: false,
      }
    },
    computed: {
      isReadyButton: function () {
        return this.smsCode.length !== '';//=== this.passwordLength || this.smsCode.length === this.passwordLength - 1;
      },
      continueButtonClass: function () {
        return this.isReadyButton ? 'ready' : 'not-ready';
      },
      smsCode: function () {
        let password = this.currentPassword;
        password = password.replace(/\s+/g, '');
        password = password.replace(/_+/g, '');
        return password;
      }
    },
    mounted() {
      const passwordMask = new InputMask("9 9 9 9", {colorMask: true, inputEventOnly: true});
      InputMask.extendDefaults({androidHack: "rtfm"});
      const password = document.getElementById("password");
      //passwordMask.mask(password);
    },
    components:{
      'masked-input': MaskedInput,
      modal
    },
    methods: {
      send() {
        if (!(isNaN(this.smsCode)) && (this.smsCode.toString().length !== '')){
        //=== this.passwordLength || this.smsCode.toString().length === this.passwordLength - 1)) {
          this.ajaxSend(this.smsCode);
        } else {
          console.log('Введено недостаточно символов');
        }
      },

      ajaxSend(smsCode) {
        this.modalPreloader = true;
        this.showModal = true;
        this.modalAlert = true;
        const numADM = this.$router.currentRoute.params.numADM;
        //http://10.100.50.248/planshet_kl/hs/cardreg?numADM=11112&check=1
        const params = {codSMS: smsCode, 'check': 1, numADM};
        const ip = store.state.ip;
        const ws = store.state.ws;

        let url = `http://planshet:planshet@${ip}/${ws}/hs/cardreg?`;
        for (let prm in params) {
          url += prm + '=' + params[prm] + '&';
        }

        this.showModal = true;
        this.axios.post(url, {data: ''})
          .then(resp => {
            console.log(resp);
            if (resp.status === 200) {
              this.modalPreloader = false;
              this.showModal = false;
              this.modalAlert = false;
              let pathName = 'Thanks';
              this.$router.replace({name: pathName, params: {lang: 'ru'}});
            }
          })
          .catch(err => {
            console.log(err);
            this.modalPreloader = false;
            this.showModal = false;
            this.modalAlert = false;
            // todo удалить
            let pathName = 'Thanks';
            this.$router.replace({name: pathName, params: {lang: 'ru'}});
          })

      }
    }
  }

</script>
