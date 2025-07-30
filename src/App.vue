<template>
  <div class="auth-page">
    <div class="container container-sm section-padding">
      <div class="row justify-content-center">
        <div class="col-md-6">
          <div class="auth-card">
            <div class="card shadow">
              <div class="card-header bg-white border-bottom-0 text-center pt-4">
                <img src="/images/logo.png" alt="لوگوی فردا موتورز" class="auth-logo mb-3" />
                <h2 class="card-title">{{ isOtpSent ? 'تایید کد یکبار مصرف' : 'ورود / ثبت نام' }}</h2>
                <p class="card-subtitle text-muted">
                  {{ isOtpSent ? 'کد ارسال شده به شماره موبایل خود را وارد کنید' : 'برای ورود یا ثبت نام اطلاعات زیر را
                  تکمیل کنید' }}
                </p>
              </div>

              <div class="card-body p-4">
                <form class="auth-form">

                  <div class="mb-3">
                    <label for="nationalId" class="form-label">کد ملی</label>
                    <input type="text" class="form-control ltr-input" id="nationalId" v-model="form.nationalId"
                      placeholder="کد ملی خود را وارد کنید" :disabled="isOtpSent"
                      :class="{ 'is-invalid': formErrors.nationalId }" maxlength="10" @input="validateNationalId" />
                    <div v-if="formErrors.nationalId" class="invalid-feedback">
                      {{ formErrors.nationalId }}
                    </div>
                  </div>
                  <div class="mb-3">
                    <label for="mobileNumber" class="form-label">شماره موبایل</label>
                    <div class="input-group">
                      <input type="text" class="form-control ltr-input" id="mobileNumber" v-model="form.mobileNumber"
                        placeholder="09xxxxxxxxx" :disabled="isOtpSent"
                        :class="{ 'is-invalid': formErrors.mobileNumber }" maxlength="11"
                        @input="validateMobileNumber" />
                    </div>
                    <div v-if="formErrors.mobileNumber" class="invalid-feedback d-block">
                      {{ formErrors.mobileNumber }}
                    </div>
                  </div>

                  <!-- <div v-if="!isOtpSent" class="arcaptcha" :data-site-key="rCapchaSiteKey" ></div> -->


                  <div v-if="isOtpSent" class="mb-3">
                    <label for="otpCode" class="form-label">کد تایید</label>
                    <div class="otp-input-container">
                      <input type="text" class="form-control ltr-input text-center" id="otpCode" v-model="form.otpCode"
                        placeholder="کد 6 رقمی" maxlength="6" @input="validateOtpCode"
                        :class="{ 'is-invalid': formErrors.otpCode }" />
                      <div v-if="formErrors.otpCode" class="invalid-feedback d-block">
                        {{ formErrors.otpCode }}
                      </div>
                    </div>

                    <div class="timer-container text-center mt-3">
                      <template v-if="timer > 0">
                        <div class="countdown-text">
                          <span>زمان باقیمانده:</span>
                          <span class="countdown">{{ formatTime(timer) }}</span>
                        </div>
                      </template>
                      <button v-else type="button" class="btn btn-link p-0" @click="resendOtp" :disabled="isLoading">
                        ارسال مجدد کد
                      </button>
                    </div>
                  </div>

                  <div class="d-grid gap-2 mt-5">


                    <div v-if="withRender">
                      <button :data-site-key="config.public.rCapchaSiteKey" :data-callback="handleSubmit" type="button"
                        class="btn btn-primary btn-lg arcaptcha" :disabled="isLoading || !isFormValid">
                        <span v-if="isLoading" class="spinner-border spinner-border-sm me-2" role="status"
                          aria-hidden="true"></span>
                        {{ isOtpSent ? 'ورود به سیستم' : 'ادامه' }}
                      </button>
                    </div>
                    <div v-else id="captcha"></div>

                  </div>
                </form>
              </div>

              <div class="card-footer bg-white border-top-0 text-center pb-4">
                <p class="mb-0">
                  <router-link to="/" class="link-primary">بازگشت به صفحه اصلی</router-link>
                </p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>


<script setup lang="ts">
import { ref, reactive, computed, onMounted } from 'vue';
import { useRouter, useNuxtApp, useRuntimeConfig, useRoute } from '#app';
import { useUserStore } from '~/store/userStore';
// import ArcaptchaVue from "arcaptcha-vue";

// Router and Store
const router = useRouter();
const userStore = useUserStore();
const { $swal } = useNuxtApp();

const config = useRuntimeConfig();

// State
const isOtpSent = ref(false);
const isLoading = ref(false);
const timer = ref(0);
const countdownInterval = ref<any>(null);
const withRender = ref(true);

// Form data
const form = reactive({
  nationalId: '',
  mobileNumber: '',
  otpCode: '',
  recaptchaToken: '',
});

// Form validation errors
const formErrors = reactive({
  nationalId: '',
  mobileNumber: '',
  otpCode: ''
});

// Computed properties
const isFormValid = computed(() => {
  if (isOtpSent.value) {
    return isValidNationalId(form.nationalId) &&
      isValidMobileNumber(form.mobileNumber) &&
      isValidOtpCode(form.otpCode);
  } else {
    return isValidNationalId(form.nationalId) &&
      isValidMobileNumber(form.mobileNumber);
  }
});

// Validation methods
function isValidNationalId(id: string): boolean {
  const regex = /^\d{10}$/;
  return regex.test(id);
}

function validateNationalId() {
  if (!form.nationalId) {
    formErrors.nationalId = 'کد ملی الزامی است';
  } else if (!isValidNationalId(form.nationalId)) {
    formErrors.nationalId = 'کد ملی باید 10 رقم باشد';
  } else {
    formErrors.nationalId = '';
  }
}

function onSuccess(token) {
  console.log("Captcha Solved! token:", token);
}

function isValidMobileNumber(mobile: string): boolean {
  const regex = /^09\d{9}$/;
  return regex.test(mobile);
}

function validateMobileNumber() {
  if (!form.mobileNumber) {
    formErrors.mobileNumber = 'شماره موبایل الزامی است';
  } else if (!isValidMobileNumber(form.mobileNumber)) {
    formErrors.mobileNumber = 'شماره موبایل باید با 09 شروع شود و 11 رقم باشد';
  } else {
    formErrors.mobileNumber = '';
  }
}

function isValidOtpCode(code: string): boolean {
  const regex = /^\d{6}$/;
  return regex.test(code);
}

function validateOtpCode() {
  if (!form.otpCode) {
    formErrors.otpCode = 'کد تایید الزامی است';
  } else if (!isValidOtpCode(form.otpCode)) {
    formErrors.otpCode = 'کد تایید باید 6 رقم باشد';
  } else {
    formErrors.otpCode = '';
  }
}

// Show alert message using toast notifications
function showAlert(type: 'success' | 'error', message: string) {


  if (process.client && window.Swal) {
    window.Swal.fire({
      icon: type,
      title: type === 'success' ? 'موفق' : 'خطا',
      text: message,
      timer: 3000,
      showConfirmButton: false,
      toast: true,
      position: 'top-end',
      customClass: {
        popup: 'rtl-alert'
      }
    });
  } else {
    // Fallback to alert
    if (type === 'success') {
      alert(`✅ ${message}`);
    } else {
      alert(`❌ ${message}`);
    }
  }
}

// Format timer for display
function formatTime(seconds: number): string {
  const minutes = Math.floor(seconds / 60);
  const remainingSeconds = seconds % 60;
  return `${minutes.toString().padStart(2, '0')}:${remainingSeconds.toString().padStart(2, '0')}`;
}

// Start countdown timer
function startTimer() {
  if (countdownInterval.value) {
    clearInterval(countdownInterval.value);
  }

  timer.value = 120; // 2 minutes in seconds

  countdownInterval.value = setInterval(() => {
    if (timer.value > 0) {
      timer.value--;
    } else {
      clearInterval(countdownInterval.value);
    }
  }, 1000);
}


const getTokenValue = (): string | null => {
  const tokenElement = document.getElementsByName('arcaptcha-token')[0];
  if (tokenElement) {
    return tokenElement.value;
  } else {
    console.warn('المان arcaptcha-token پیدا نشد');
    return null; // اگر المان وجود نداشت، null برمی‌گرداند
  }
};
// API calls
async function sendOtp() {

  try {
    isLoading.value = true;

    var _recaptchaToken = getTokenValue();

    const response = await fetch(`${useRuntimeConfig().public.apiBaseUrl}/api/UserCustomer/send-otp`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        [useRuntimeConfig().public.gwKeyName]: useRuntimeConfig().public.gwKeyValue,
      },
      body: JSON.stringify({
        nationalId: form.nationalId,
        mobileNumber: form.mobileNumber,
        recaptchaToken: _recaptchaToken,
      })
    });

    const data = await response.json();

    if (data.success) {
      isOtpSent.value = true;
      userStore.setUserInfo(form.mobileNumber);
      startTimer();
      showAlert('success', data.message);
    } else {
      showAlert('error', data.errorMessages?.join('\n') || 'خطا در ارسال کد تایید');
    }
  } catch (error) {
    console.error('Error sending OTP:', error);
    showAlert('error', 'خطا در ارتباط با سرور');
  } finally {
    isLoading.value = false;
  }
}

async function resendOtp() {
  await sendOtp();
}

async function verifyOtp() {
  try {
    isLoading.value = true;

    const response = await fetch(`${useRuntimeConfig().public.apiBaseUrl}/api/UserCustomer/login`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        [useRuntimeConfig().public.gwKeyName]: useRuntimeConfig().public.gwKeyValue,
      },
      body: JSON.stringify({
        nationalId: form.nationalId,
        mobileNumber: form.mobileNumber,
        confirmCode: form.otpCode
      })
    });

    const data = await response.json();
    if (data.success && data.data) {
      // Save auth data to store and localStorage, including mobile number and customer name
      userStore.setAuthData(data.data.token, data.data.userId, form.mobileNumber, data.data.customerName);
      showAlert('success', data.message || 'ورود موفقیت آمیز');
      // Navigate to returnUrl (if present), else home
      const returnUrl = useRoute().query.returnUrl as string | undefined;
      setTimeout(() => {
        if (returnUrl) {
          router.push(returnUrl);
        } else {
          router.push('/');
        }
      }, 1500);
    } else {
      showAlert('error', data.message ?? (data.errorMessages?.join('\n') || 'خطا در احراز هویت'));
    }
  } catch (error) {
    console.error('Error verifying OTP:', error);
    showAlert('error', 'خطا در ارتباط با سرور');
  } finally {
    isLoading.value = false;
  }
}

// Form submission handler
async function handleSubmit(token) {

  alert(token)

  // Validate all fields
  validateNationalId();
  validateMobileNumber();
  if (isOtpSent.value) validateOtpCode();

  // If not valid, return early
  if (!isFormValid.value) return;

  if (isOtpSent.value) {
    await verifyOtp();
  } else {
    await sendOtp();
  }
}

const loadScript = (src: string) => {
  return new Promise((resolve, reject) => {
    const script = document.createElement('script');
    script.src = src;
    script.async = true;
    script.defer = true;
    // script.onload = () => resolve(true);
    // script.onerror = () => reject(new Error(`Failed to load script: ${src}`));
    document.head.appendChild(script);
  });
};


onBeforeMount(() => {

  //  loadScript('https://widget.arcaptcha.ir/1/api.js');

  const script = document.createElement('script');
  script.src = 'https://widget.arcaptcha.ir/1/api.js';
  script.async = true;
  script.defer = true;
  // script.onload = () => resolve(true);
  // script.onerror = () => reject(new Error(`Failed to load script: ${src}`));
  document.head.appendChild(script);
})

// Check if user is already logged in
onMounted(() => {

  setTimeout(() => {


    if (!withRender.value) {
      window.arcaptcha.render('#captcha', {
        site_key: config.public.rCapchaSiteKey,
        size: 'invisible',
        //callback:this.
      });

      withRender.value = true;

      arcaptcha.execute();
    }
  }, 1000);


  if (userStore.checkAuth()) {
    // Only redirect to home if NOT coming from a returnUrl (i.e. direct /auth visit)
    const route = useRoute();
    const returnUrl = route.query.returnUrl as string | undefined;
    if (!returnUrl) {
      router.push('/');
    }
  }
});
</script>

<style lang="scss" scoped>
.auth-page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  padding: 40px 0;
  background-color: #f8f9fa;

  .auth-card {
    max-width: 500px;
    margin: 0 auto;

    .card {
      border-radius: 15px;
      border: none;
      overflow: hidden;
    }
  }

  .auth-logo {
    max-height: 60px;
    margin-bottom: 1rem;
  }

  .ltr-input {
    direction: ltr;
    text-align: left;
  }

  .rtl-input-group {
    direction: ltr;
    display: flex;

    .form-control {
      order: 1;
    }

    .input-group-text {
      order: 2;
    }
  }

  .timer-container {
    margin-top: 10px;

    .countdown-text {
      color: #6c757d;
      font-size: 0.9rem;

      .countdown {
        color: #dc3545;
        font-weight: 600;
        margin-right: 5px;
      }
    }
  }

  .otp-input-container {
    max-width: 200px;
    margin: 0 auto;
  }
}
</style>