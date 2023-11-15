<template>
  <div
    class="relative flex justify-center flex-col items-center h-[100vh] bg-[#262D34]"
  >
    <!-- <img src="~/assets/img/Cool-Background-GIF.gif" alt="" class="absolute w-full h-full"> -->
    <div class="absolute w-full h-full bg-[#2C353D]"></div>
    <div class="z-10">
      <form
        action=""
        class="flex flex-col form gap-7 py-[60px] justify-center items-center w-[400px] rounded-md bg-[#1E252B] px-10"
      >
        <div class="text-3xl font-bold text-white">Sign up</div>
        <div class="w-full flex flex-col gap-2">
          <label for="" class="text-sm text-white font-medium">Email</label>
          <input
            v-model="email"
            type="text"
            class="focus:outline-0 text-white h-[50px] w-full rounded-md pl-5 bg-[#2C353D]"
            placeholder="Email"
            :class="{ invalid: !validateEmail }"
          />
        </div>
        <div class="w-full relative flex flex-col gap-2">
          <label for="" class="text-sm text-white font-medium">Password</label>
          <input
            v-model="password"
            :type="isShowPassword ? 'text' : 'password'"
            class="focus:outline-0 text-white h-[50px] w-full rounded-md pl-5 pr-8 bg-[#2C353D]"
            placeholder="Password"
          />
          <img
            v-if="isShowPassword"
            src="~assets/icon/eye.svg"
            class="absolute w-[20px] h-[16px] right-[10px] bottom-[17px] cursor-pointer"
            alt=""
            @click="toggleShowPassword()"
          />
          <img
            v-else
            src="~assets/icon/eye-slash.svg"
            class="absolute w-[20px] h-[16px] right-[10px] bottom-[17px] cursor-pointer"
            alt=""
            @click="toggleShowPassword()"
          />
        </div>
        <div class="w-full relative flex flex-col gap-2">
          <label for="" class="text-sm text-white font-medium"
            >Password confirm</label
          >
          <input
            v-model="password_confirm"
            :type="isShowPasswordConfirm ? 'text' : 'password'"
            class="focus:outline-0 text-white h-[50px] w-full rounded-md pl-5 pr-8 bg-[#2C353D]"
            placeholder="Password confirm"
            :class="{ invalid: !validateMatchPassword }"
          />
          <img
            v-if="isShowPasswordConfirm"
            src="~assets/icon/eye.svg"
            class="absolute w-[20px] h-[16px] right-[10px] bottom-[17px] cursor-pointer"
            alt=""
            @click="toggleShowPasswordConfirm()"
          />
          <img
            v-else
            src="~assets/icon/eye-slash.svg"
            class="absolute w-[20px] h-[16px] right-[10px] bottom-[17px] cursor-pointer"
            alt=""
            @click="toggleShowPasswordConfirm()"
          />
        </div>
        <hr />
        <div class="flex justify-between w-full">
          <button
            class="text-sm text-[#FF4B26] font-medium"
            @click.prevent="toSignin"
          >
            Go to sign in
          </button>
        </div>
        <div class="flex justify-center items-center w-full">
          <button
            type="button"
            @click="submit"
            class="text-[16px] font-bold text-white bg-[#FF4401] rounded-[25px] py-[8px] px-[40px]"
          >
            Sign up
          </button>
        </div>
      </form>
    </div>
    <modal-alert
      v-if="alert.isShowModal"
      v-bind="alert"
      @close="onCloseModal"
    />
  </div>
</template>

<script>
import ModalAlert from '~/components/Modal/ModalAlert.vue'
export default {
  components: { ModalAlert },
  layout: 'empty',
  data() {
    return {
      email: '',
      password: '',
      password_confirm: '',
      isShowPassword:false,
      isShowPasswordConfirm: false,
      alert: {
        isShowModal: false,
        title: 'Xác nhận',
        type: 'failed',
        content: 'Cần được xác nhận',
        buttonCancelContent: '',
        buttonOkContent: 'Ok',
        typeSubmit: '',
      },
    }
  },
  computed: {
    validateEmail() {
      const regex =
        /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|.(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
      if (this.email) return this.email.toLowerCase().match(regex)
      else return true
    },
    validateMatchPassword() {
      if (this.password_confirm) return this.password_confirm === this.password
      else return true
    },
    checkData() {
      return this.email && this.password && this.password_confirm
    },
  },
  methods: {
    submit() {
      if (
        !this.checkData ||
        !this.validateEmail ||
        !this.validateMatchPassword
      ) {
        if (!this.checkData) {
          this.$notify({
            title: 'Lỗi',
            text: 'Vui lòng điền đầy đủ thông tin',
            type: 'error',
            group: 'foo',
          })
        } else if (!this.validateEmail) {
          this.$notify({
            group: 'foo',
            title: 'Lỗi',
            text: 'Vui lòng điền email đúng định dạng',
            type: 'error',
          })
        } else {
          this.$notify({
            group: 'foo',
            title: 'Lỗi',
            text: 'Password confirm không khớp',
            type: 'error',
          })
        }
      } else {
        this.$axios.post('/auth/register', {
          email: this.email.toLowerCase().trim(),
          password: this.password,
        }).then(res => {
          this.alert = {... this.alert,
            ...{
              isShowModal : true,
              title: 'Thành công',
              content:'Bạn đã đăng kí thành công, hãy đăng nhập để sử dụng trang web',
              type:'success',
              buttonOkContent:'Đến sign in',
              typeSubmit:'gotologin'
          }}
        }).catch((res) => {
          this.alert = {... this.alert,
            ...{
              isShowModal : true,
              title: 'Thất bại',
              content: res.response.data.message[0].message,
              type:'failed',
              buttonOkContent:'Đóng'
          }}
        })
      }
    },
    toSignin() {
      this.$router.push('/auth/login')
    },
    toggleShowPassword() {
      this.isShowPassword = !this.isShowPassword
    },
    toggleShowPasswordConfirm() {
      this.isShowPasswordConfirm = !this.isShowPasswordConfirm
    },
    toFogotPassword() {
      this.$router.push('/auth/forgot-password')
    },
    onCloseModal(typeSubmit) {
      switch (typeSubmit) {
        case 'gotologin':
          this.$router.push('/auth/login')
          break
        default:
          this.resetAlert()
          break
      }
    },
    resetAlert() {
      this.alert = {
        isShowModal: false,
        title: '',
        type: 'failed',
        content: '',
        buttonCancelContent: '',
        buttonOkContent: '',
        typeSubmit: '',
      }
    },
  },
}
</script>
