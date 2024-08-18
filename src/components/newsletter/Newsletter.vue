<template>
  <form name="newsletter-subscribes" class="relative inline-flex items-center gap-4 py-4" @submit.prevent="submit"
    data-netlify="true" method="POST" netlify-honeypot="bot-field">
    <input type="hidden" name="form-name" value="newsletter-subscribes" />
    <p class="hidden">
      <label>
        Don't fill this out if you're human: <input name="bot-field" />
      </label>
    </p>
    <input type="email" name="email" :placeholder="t('email')" v-model="form.email"
      class="relative block overflow-hidden rounded-full bg-dark px-5 py-2"
      :class="errorFields?.email?.length ? 'text-warning' : 'text-light'" />
    <button type="submit" :disabled="!canSubmit" class="btn"
      :class="canSubmit ? 'surface-primary' : 'surface-base opacity-50'">
      {{ t("subscribe") }}
    </button>
    <Loading :loading="loading" />
  </form>
</template>

<script setup>
import { ref, computed, reactive } from "vue";
import { t } from "@util/translate";
import { useAsyncValidator } from "@vueuse/integrations/useAsyncValidator";
import Loading from "@components/common/Loading.vue";
import "vue3-toastify/dist/index.css";
import { toast } from "vue3-toastify";

const props = defineProps({
  type: {
    type: String,
    required: false,
    default: "netlify",
  },
  list_id: String,
});

const loading = ref(false);
const message = ref(null);
const form = reactive({ email: "" });

const rules = {
  email: [
    {
      type: "email",
      required: true,
    },
  ],
};

const { pass, isFinished, errorFields } = useAsyncValidator(form, rules);

const canSubmit = computed(() => {
  return !loading.value && isFinished.value && pass.value;
});

const resetForm = () => {
  form.email = "";
};

const submit = () => {
  loading.value = true;
  const formElement = document.querySelector('form[name="newsletter-subscribes"]');
  const formData = new FormData(formElement);

  fetch("/", {
    method: "POST",
    headers: { "Content-Type": "application/x-www-form-urlencoded" },
    body: new URLSearchParams(formData).toString(),
  })
    .then((response) => {
      if (response.ok) {
        toast.success(t("newsletter_thanks"));
        resetForm();
      } else {
        throw new Error("Network response was not ok");
      }
    })
    .catch((error) => {
      console.error("Error:", error);
      toast.error(t("newsletter_error"));
    })
    .finally(() => {
      loading.value = false;
    });
};
</script>