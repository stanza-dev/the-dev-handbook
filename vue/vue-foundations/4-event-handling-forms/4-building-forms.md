---
source_course: "vue-foundations"
source_lesson: "vue-foundations-building-forms"
---

# Building Complete Forms

Let's combine everything to build real-world forms with validation, submission handling, and user feedback.

## A Complete Registration Form

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

// Form data
const form = ref({
  email: '',
  password: '',
  confirmPassword: '',
  name: '',
  agreeToTerms: false
})

// Form state
const isSubmitting = ref(false)
const submitError = ref('')
const submitSuccess = ref(false)

// Validation
const errors = computed(() => {
  const errs: Record<string, string> = {}
  
  // Email validation
  if (!form.value.email) {
    errs.email = 'Email is required'
  } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.value.email)) {
    errs.email = 'Please enter a valid email'
  }
  
  // Name validation
  if (!form.value.name) {
    errs.name = 'Name is required'
  } else if (form.value.name.length < 2) {
    errs.name = 'Name must be at least 2 characters'
  }
  
  // Password validation
  if (!form.value.password) {
    errs.password = 'Password is required'
  } else if (form.value.password.length < 8) {
    errs.password = 'Password must be at least 8 characters'
  }
  
  // Confirm password
  if (form.value.password !== form.value.confirmPassword) {
    errs.confirmPassword = 'Passwords do not match'
  }
  
  // Terms agreement
  if (!form.value.agreeToTerms) {
    errs.agreeToTerms = 'You must agree to the terms'
  }
  
  return errs
})

const isValid = computed(() => Object.keys(errors.value).length === 0)

// Track which fields have been touched
const touched = ref<Record<string, boolean>>({})

function markTouched(field: string) {
  touched.value[field] = true
}

function shouldShowError(field: string): boolean {
  return touched.value[field] && !!errors.value[field]
}

// Form submission
async function handleSubmit() {
  // Mark all fields as touched to show all errors
  Object.keys(form.value).forEach(key => {
    touched.value[key] = true
  })
  
  if (!isValid.value) {
    return
  }
  
  isSubmitting.value = true
  submitError.value = ''
  
  try {
    // Simulate API call
    await new Promise(resolve => setTimeout(resolve, 1500))
    
    // Simulate occasional error
    if (Math.random() < 0.2) {
      throw new Error('Email already registered')
    }
    
    submitSuccess.value = true
    console.log('Form submitted:', form.value)
  } catch (error) {
    submitError.value = error instanceof Error ? error.message : 'Submission failed'
  } finally {
    isSubmitting.value = false
  }
}

// Reset form
function resetForm() {
  form.value = {
    email: '',
    password: '',
    confirmPassword: '',
    name: '',
    agreeToTerms: false
  }
  touched.value = {}
  submitError.value = ''
  submitSuccess.value = false
}
</script>

<template>
  <div class="form-container">
    <h2>Create Account</h2>
    
    <!-- Success message -->
    <div v-if="submitSuccess" class="success-message">
      <p>Account created successfully!</p>
      <button @click="resetForm">Create Another</button>
    </div>
    
    <!-- Registration form -->
    <form v-else @submit.prevent="handleSubmit">
      <!-- Name field -->
      <div class="form-group">
        <label for="name">Name</label>
        <input
          id="name"
          v-model.trim="form.name"
          type="text"
          :class="{ error: shouldShowError('name') }"
          @blur="markTouched('name')"
        />
        <span v-if="shouldShowError('name')" class="error-text">
          {{ errors.name }}
        </span>
      </div>
      
      <!-- Email field -->
      <div class="form-group">
        <label for="email">Email</label>
        <input
          id="email"
          v-model.trim="form.email"
          type="email"
          :class="{ error: shouldShowError('email') }"
          @blur="markTouched('email')"
        />
        <span v-if="shouldShowError('email')" class="error-text">
          {{ errors.email }}
        </span>
      </div>
      
      <!-- Password field -->
      <div class="form-group">
        <label for="password">Password</label>
        <input
          id="password"
          v-model="form.password"
          type="password"
          :class="{ error: shouldShowError('password') }"
          @blur="markTouched('password')"
        />
        <span v-if="shouldShowError('password')" class="error-text">
          {{ errors.password }}
        </span>
      </div>
      
      <!-- Confirm password -->
      <div class="form-group">
        <label for="confirmPassword">Confirm Password</label>
        <input
          id="confirmPassword"
          v-model="form.confirmPassword"
          type="password"
          :class="{ error: shouldShowError('confirmPassword') }"
          @blur="markTouched('confirmPassword')"
        />
        <span v-if="shouldShowError('confirmPassword')" class="error-text">
          {{ errors.confirmPassword }}
        </span>
      </div>
      
      <!-- Terms checkbox -->
      <div class="form-group checkbox">
        <label>
          <input
            v-model="form.agreeToTerms"
            type="checkbox"
            @change="markTouched('agreeToTerms')"
          />
          I agree to the Terms of Service
        </label>
        <span v-if="shouldShowError('agreeToTerms')" class="error-text">
          {{ errors.agreeToTerms }}
        </span>
      </div>
      
      <!-- Error message -->
      <div v-if="submitError" class="error-message">
        {{ submitError }}
      </div>
      
      <!-- Submit button -->
      <button 
        type="submit" 
        :disabled="isSubmitting"
        class="submit-btn"
      >
        {{ isSubmitting ? 'Creating Account...' : 'Create Account' }}
      </button>
    </form>
  </div>
</template>

<style scoped>
.form-container {
  max-width: 400px;
  margin: 0 auto;
  padding: 2rem;
}

.form-group {
  margin-bottom: 1rem;
}

label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
}

input[type="text"],
input[type="email"],
input[type="password"] {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
}

input.error {
  border-color: #e53e3e;
}

.error-text {
  color: #e53e3e;
  font-size: 0.875rem;
  margin-top: 0.25rem;
  display: block;
}

.checkbox label {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.submit-btn {
  width: 100%;
  padding: 0.75rem;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 1rem;
  cursor: pointer;
}

.submit-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.error-message {
  background: #fed7d7;
  color: #c53030;
  padding: 0.75rem;
  border-radius: 4px;
  margin-bottom: 1rem;
}

.success-message {
  background: #c6f6d5;
  color: #276749;
  padding: 1rem;
  border-radius: 4px;
  text-align: center;
}
</style>
```

This form demonstrates:

- **Reactive form state** with `ref()`
- **Computed validation** that updates automatically
- **Touch tracking** to show errors only after interaction
- **Loading states** during submission
- **Error handling** with user feedback
- **Form reset** functionality
- **Accessibility** with proper labels and IDs

## Resources

- [Vue.js Form Handling](https://vuejs.org/guide/essentials/forms.html) — Official guide to building forms with Vue

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*