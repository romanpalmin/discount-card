<template>
  <div>
    <div class="modal-page-wrapper">

    </div>
    <div class="main-modal" v-if="showAlert">
      <div class="modal-page-content">
        <div class="header1" v-if="title && title.length > 0">{{title}}</div>
        <div class="header2" v-if="isPreloader"><img class="img-spinner" :src="getImg('spinner')" /></div>
        <hr/>
        <div class="header2">
          {{message}}
        </div>
        <div class="modal-buttons" v-if="showCallbackButton" @click="accept()">
          <button>ОК</button>
        </div>
      </div>
    </div>
  </div>
</template>
<style scoped lang="less">
  .modal-page-wrapper {
    width: 100%;
    height: 100%;
    background-color: black;
    opacity: 0.5;
    z-index: 9990;
    position: absolute;
    top: 0;
    left: 0;
    display: block;
  }

  .main-modal {
    width: 100%;
    display: block;
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    z-index: 9991;
    .modal-page-content {
      opacity: 1;
      width: 70%;
      border-radius: 16px;
      border: 5px solid #383737;
      margin: 35% auto 0;
      background-color: white;
      color: black;
      font-family: IntroHeader;
      .header1 {
        font-size: 38pt;
        padding: 20px;
      }
      .header2 {
        font-size: 23pt;
        padding: 20px;
        img{
          width: 30px;
          height: 30px;
        }
      }
      .modal-buttons {
        button {
          width: 100px;
          height: 40px;
          margin: 10px;
          font-size: 20pt;
          font-weight: 600;
        }
      }
    }
  }

</style>
<script>
  export default {
    data() {
      return {
        name: 'this component',
      }
    },
    methods:{
      accept(){
        this.$parent.showModal = false;
        this.$parent.modalAlert = true;
        this.$parent.modalText = '';
        this.$parent.modalTitle = '';
        this.$parent.modalButton = false;
        this.$parent.showPreloader = false;
      },
      getImg(name) {
        let src = 'http://10.10.182.11/img/' + name + '.gif';
        return src;
      },
    },
    computed: {
      showCallbackButton: function () {
        return this.showButton;
      },
      title: function () {
        return this.alertTitle;
      },
      message: function(){
        return this.alertText;
      },
      isPreloader: function(){
        return this.showPreloader;
      }
    },
    mounted() {
      console.log(this.showPreloader);
    },
    props: ['isLoader', 'alertText', 'showButton', 'showAlert', 'alertTitle', 'showPreloader']
  }
</script>
