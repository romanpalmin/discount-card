<template>
  <div>
    <div class="password-page-content">
      <div class="row-item title">
        ВВЕДИТЕ ПАРОЛЬ:
      </div>
      <div class="row-item input">
        <!-- <input id="password" type="number" class="password-input" v-mask="'####'" placeholder="_ _ _ _"
                v-model="currentPassword"/>-->
        <div class="password-input block" @click="setFocus()" v-model="shownPass">{{shownPass}}</div>
        <input type="number" class="hidden-input" id="passHidden" v-model="hiddenPass" maxlength="4"/>
      </div>
      <div class="row-item button">
        <button :class="continueButtonClass" @click="send()">продолжить</button>
      </div>
      <div class="row-item prompt">
        <span v-if="showPrompt">Неправильный пароль</span>
      </div>
      <modal v-if="showModal"
             :isLoader=true
             :alertText="modalText"
             :alertTitle="modalTitle"
             :showButton="modalButton"
             :showAlert="modalAlert"
             :showPreloader="showPreloader"
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
      .hidden {
        opacity: 0;
      }
      &.input {
        div {
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
          .hidden {
            height: 0px;
            margin-top: 0px;
          }
          &.block {
            margin: 20px auto 0;
            line-height: 60px;
          }

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
      .hidden-input {
        height: 0;
        margin-top: 0;
        opacity: 0;
      }
    }
  }
</style>
<script>
  import modal from '../components/modal.vue';
  import store from '../store';

  export default {
    data() {
      return {
        name: 'this component',
        showPrompt: false,
        passwordLength: 4,
        /*currentPassword: '',*/
        hiddenPass: '',
        showModal: false,
        modalText: '',
        modalTitle: '',
        modalButton: false,
        modalAlert: false,
        showPreloader: false
      }
    },
    computed: {
      isReadyButton: function () {
        return this.numADM.length === this.passwordLength || this.numADM.length === this.passwordLength - 1;
      },
      continueButtonClass: function () {
        return this.isReadyButton ? 'ready' : 'not-ready';
      },
      numADM: function () {
        this.hiddenPass = this.hiddenPass.substr(0, 4);
        this.hiddenPass = this.hiddenPass.replace(/\s+/g, '');
        this.hiddenPass = this.hiddenPass.replace(/_+/g, '');
        let password = this.hiddenPass;
        password = password.replace(/\s+/g, '');
        password = password.replace(/_+/g, '');
        return password;
      },
      shownPass: function () {
        const currentPass = this.hiddenPass;
        let resPass = '';
        switch (currentPass.length) {
          case 0:
            resPass = `_ _ _ _`;
            break;
          case 1:
            resPass = `${currentPass[0]} _ _ _`;
            break;
          case 2:
            resPass = `${currentPass[0]} ${currentPass[1]} _ _`;
            break;
          case 3:
            resPass = `${currentPass[0]} ${currentPass[1]} ${currentPass[2]} _`;
            break;
          case 4:
            resPass = `${currentPass[0]} ${currentPass[1]} ${currentPass[2]} ${currentPass[3]}`;
            break;
          default:
            resPass = `_ _ _ _`;
            break;
        }
        return resPass;
      }
    },
    mounted() {
          },
    components: {
      modal
    },
    methods: {
      formatPass(pass) {

      },
      setFocus() {
        const el = document.getElementById('passHidden');
        el.focus();
      },
      send() {
        if (!(isNaN(this.numADM)) && (
            this.numADM.toString().length === this.passwordLength || this.numADM.toString().length === this.passwordLength - 1)) {
          this.ajaxSend(this.numADM);
        } else {
          console.log('Введено недостаточно символов');
        }
      },

      ajaxSend(numADM) {
        const params = {numADM, 'check': 1};
        const ip = store.state.ip;
        const ws = store.state.ws;
        store.setNumADM(numADM);
        let url = `http://planshet:planshet@${ip}/${ws}/hs/cardreg?`;
        for (let prm in params) {
          url += prm + '=' + params[prm] + '&';
        }
        url += 'image=1';
        this.showPreloader = true;
        this.showModal = true;
        this.modalAlert = true;
        this.axios.post(url, {data: ''})
          .then(resp => {
            this.showModal = false;
            this.showPreloader = false;
            console.log(resp);
            if (resp.status === 200) {
              let pathName = 'InputForm';
              if (resp.data === 1) {
                this.$router.replace({name: pathName, params: {lang: 'ru', numADM: numADM}});
              } else {
                this.modalAlert = true;
                this.modalText = 'Неправильный пароль';
                this.modalTitle = 'Ошибка';
                this.modalButton = true;
                this.showModal = true;
                this.showPreloader = false;
              }
            }
          })
          .catch(err => {
            console.log(err);
            this.modalAlert = true;
            this.modalText = 'Обратитесь к администратору';
            this.modalTitle = 'Ошибка';
            this.modalButton = true;
            this.showModal = true;
            this.showPreloader = false;
            // todo удалить
            //let pathName = 'InputForm';
            //this.$router.replace({name: pathName, params: {lang: 'ru', numADM: numADM}});
          })

      }
    }
  }
</script>
