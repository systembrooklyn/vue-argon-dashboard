<script setup>
import { ref, computed, onBeforeMount, onBeforeUnmount, watch } from "vue";
import { useStore } from "vuex";
import { useRouter } from "vue-router";

import Navbar from "@/examples/PageLayout/Navbar.vue";
import ArgonInput from "@/components/ArgonInput.vue";
import ArgonButton from "@/components/ArgonButton.vue";
import ArgonAlert from "@/components/ArgonAlert.vue"; // لإضافة تنبيه في حالة الخطأ

const store = useStore();
const router = useRouter(); // لاستخدام التوجيه بعد تسجيل الدخول
const email = ref(""); // البريد الإلكتروني المدخل
const password = ref(""); // كلمة المرور المدخلة
const showPassword = ref(false); // للتحكم في عرض/إخفاء كلمة المرور
const showAlert = ref(false); // حالة التحكم في عرض التنبيه
const errorMessage = ref(""); // رسالة الخطأ التي ستظهر عند الفشل
const passwordValid = ref(false); // للتحقق من صحة كلمة المرور

const body = document.getElementsByTagName("body")[0];
const isLoading = ref(false);


onBeforeMount(() => {
  store.state.hideConfigButton = true;
  store.state.showNavbar = false;
  store.state.showSidenav = false;
  store.state.showFooter = false;
  body.classList.remove("bg-gray-100");

  const isRTL = store.getters.currentLanguage === "ar";
  document
    .querySelector("html")
    .setAttribute("lang", store.getters.currentLanguage);
  document.querySelector("html").setAttribute("dir", isRTL ? "rtl" : "ltr");
  document.querySelector("#app").classList.toggle("rtl", isRTL);
});

onBeforeUnmount(() => {
  store.state.hideConfigButton = false;
  store.state.showNavbar = true;
  store.state.showSidenav = true;
  store.state.showFooter = true;
  body.classList.add("bg-gray-100");
});

// دالة التحقق من قوة كلمة المرور
const validatePassword = (password) => {
  const regex = /^(?=.*[A-Z])(?=.*[a-z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/;
  return regex.test(password);
};

// مراقبة إدخال كلمة المرور والتحقق منها
watch(password, (newPassword) => {
  passwordValid.value = validatePassword(newPassword);
});

// دالة التحقق من البريد الإلكتروني وكلمة المرور
const signIn = async () => {
  isLoading.value = true;
  if (!passwordValid.value) {
    errorMessage.value = store.getters.currentLanguage === "ar"
      ? "كلمة المرور غير مطابقة للشروط."
      : "Password does not meet the requirements.";
    showAlert.value = true;

    // إخفاء التنبيه بعد 3 ثوانٍ
    setTimeout(() => {
      isLoading.value = false;
      showAlert.value = false;

    }, 3000);
    return;
  }

  try {

    const formData = {
      email: email.value,
      password: password.value
    }

    console.log("formData:", formData);
    // إرسال البيانات إلى المسار للتحقق من تسجيل الدخول
    const response = await store.dispatch("signIn", formData);
    console.log("response:", response);
    if (response.status === 200) {
      isLoading.value = false;
      router.push("/routine-task");
    } else {
      isLoading.value = false;
      errorMessage.value = t("loginError");
      showAlert.value = true;
      setTimeout(() => {
        showAlert.value = false;
      }, 3000);
    }
  } catch (error) {
    isLoading.value = false;
    console.error("Error logging in:", error);
    errorMessage.value = t("loginError");
    showAlert.value = true;

    // إخفاء التنبيه بعد 3 ثوانٍ
    setTimeout(() => {
      showAlert.value = false;
    }, 3000);
  }
};

const showForgotPasswordForm = () => {
  router.push("/forgot-password");
};

const translations = {
  en: {
    signIn: "Sign In",
    enterEmailAndPassword: "Enter your email and password to sign in",
    email: "Email",
    password: "Password",
    rememberMe: "Remember me",
    dontHaveAccount: "Don't have an account?",
    signUp: "Sign Up",
    attentionIsNewCurrency: '"Attention is the new currency"',
    effortlessWriting:
      "The more effortless the writing looks, the more effort the writer actually put into the process.",
    forgotPassword: "Forgot password?",
    passwordRequirements: "Password must be at least 8 characters, include an uppercase letter, a lowercase letter, a number, and a special character.",
    passwordValid: "Password is valid ✅",
    showPassword: "Show Password",
    hidePassword: "Hide Password",
    showForgotPasswordForm: "Show Forgot Password Form",
    loginError: "Email or password is incorrect.",
    generalError: "There was an error logging in. Please try again.",
  },
  ar: {
    signIn: "تسجيل الدخول",
    enterEmailAndPassword: "أدخل بريدك الإلكتروني وكلمة المرور لتسجيل الدخول",
    email: "البريد الإلكتروني",
    password: "كلمة المرور",
    rememberMe: "تذكرني",
    dontHaveAccount: "ليس لديك حساب؟",
    signUp: "تسجيل",
    attentionIsNewCurrency: '"الانتباه هو العملة الجديدة"',
    effortlessWriting:
      "كلما بدت الكتابة أكثر سهولة، كلما بذل الكاتب مجهودًا أكبر في العملية.",
    forgotPassword: "نسيت كلمة المرور؟",
    passwordRequirements: "يجب أن تتكون كلمة المرور من 8 أحرف على الأقل، وتشمل حرفًا كبيرًا، وحرفًا صغيرًا، ورقمًا، وحرفًا خاصًا.",
    passwordValid: "كلمة المرور مطابقة للشروط ✅",
    showPassword: "إظهار كلمة المرور",
    hidePassword: "إخفاء كلمة المرور",
    showForgotPasswordForm: "اظهار نموذج نسيت كلمة المرور",
    loginError: "البريد الإلكتروني او كلمة المرور غير صحيح",
    generalError: "حدث خطأ أثناء تسجيل الدخول. الرجاء المحاولة مرة أخرى.",
  },
};

const currentLanguage = computed(() => store.getters.currentLanguage);

const t = (key) => {
  return translations[currentLanguage.value][key];
};
</script>

<template>
  <div class="container top-0 position-sticky z-index-sticky">
    <div class="row">
      <div class="col-12">
        <navbar isBlur="blur  border-radius-lg my-3 py-2 start-0 end-0 mx-4 shadow" :darkMode="true"
          isBtn="bg-gradient-success" style="display: none;" />
      </div>
    </div>
  </div>
  <main class="mt-0 main-content">
    <section>
      <div class="page-header min-vh-100">
        <div class="container">
          <div class="row">
            <!-- قسم النموذج -->
            <div class="col-xl-4 col-lg-5 col-md-7 d-flex flex-column mx-lg-0 mx-auto form-container"
              :class="currentLanguage === 'ar' ? 'order-2' : ''">
              <div class="card card-plain">
                <div class="pb-0 card-header text-start">
                  <h4 class="font-weight-bolder">{{ t("signIn") }}</h4>
                  <p class="mb-0">{{ t("enterEmailAndPassword") }}</p>
                </div>
                <div class="card-body">
                  <form role="form" @submit.prevent="signIn">
                    <div class="mb-3">
                      <argon-input v-model="email" id="email" type="email" :placeholder="t('email')" name="email"
                        size="lg" />
                    </div>
                    <div class="mb-3 position-relative">
                      <div class=" position-relative">
                        <argon-input v-model="password" id="password" :type="showPassword ? 'text' : 'password'"
                          :placeholder="t('password')" name="password" size="lg" />
                        <span @click="showPassword = !showPassword"
                          class="position-absolute end-0 top-50 translate-middle-y me-3 cursor-pointer">
                          <i :class="showPassword ? 'fas fa-eye' : 'fas fa-eye-slash'"></i>
                        </span>
                      </div>
                      <p v-if="passwordValid" class="text-success mt-2">
                        {{ t("passwordValid") }}
                      </p>
                      <p v-if="!passwordValid" class="text-danger mt-2">
                        {{ t("passwordRequirements") }}
                      </p>
                    </div>
                    <p class="mx-auto mb-4 text-sm">
                      <a href="javascript:;" @click="showForgotPasswordForm"
                        class="text-success text-gradient font-weight-bold">
                        {{ t("forgotPassword") }}
                      </a>
                    </p>
                    <argon-alert v-if="showAlert" color="danger">
                      {{ errorMessage }}
                    </argon-alert>
                    <div class="text-center">
                      <argon-button class="mt-4 d-flex justify-content-center" variant="gradient" color="success" fullWidth size="lg"
                        :disabled="!passwordValid">
                        <span class="me-2 mx-2">{{ t("signIn") }}</span>
                        <div v-if="isLoading" class="text-center">
                          <span class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
                        </div>
                      </argon-button>
                    </div>
                  </form>
                </div>
                <div class="px-1 pt-0 text-center card-footer px-lg-2">
                  <p class="mx-auto mb-4 text-sm">
                    {{ t("dontHaveAccount") }}
                    <a href="/" class="text-success text-gradient font-weight-bold">{{ t("signUp") }}</a>
                  </p>
                </div>
              </div>
            </div>
            <!-- قسم الصورة والنص -->
            <div
              class="col-6 d-none d-lg-flex h-100 justify-content-center flex-column text-center my-auto top-0 position-absolute"
              :class="currentLanguage === 'ar' ? 'start-0 ps-0' : 'end-0 pe-0'">
              <div
  class="position-relative bg-gradient-primary w-100  px-7 border-radius-lg d-flex flex-column justify-content-center overflow-auto"
  style="
    background-image: url('https://ik.imagekit.io/dimpx0s2v/Copy%20of%201task%20(1).gif');
    background-size: contain; 
    background-position: center;
    background-repeat: no-repeat;
    height: 480px; 
    margin-left: -100px;
    max-height: 90vh; 
  ">
  <span class=""></span>
  <!-- المحتويات الأخرى يمكن إضافتها هنا -->
  <!--
  <h4 class="mt-5 text-white font-weight-bolder position-relative">
    {{ t("attentionIsNewCurrency") }}
  </h4>
  <p class="text-white position-relative">
    {{ t("effortlessWriting") }}
  </p>
  -->
</div>

            </div>
            <!-- نهاية قسم الصورة والنص -->
          </div>
        </div>
      </div>
    </section>
  </main>
</template>

<style scoped>
.form-container {
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2); /* ظل أعمق وأكثر وضوحًا */
  border: 2px solid rgba(255, 255, 255, 0.5); /* حدود أكثر سمكًا وشفافية */
  border-radius: 10px;
  padding: 20px;
}
</style>

