<template>
    <el-form ref="ruleFormRef" style="max-width: 600px" :model="ruleForm" status-icon :rules="rules" label-width="auto"
        class="demo-ruleForm">
        <el-form-item label="Login" prop="login">
            <el-input v-model="ruleForm.login" type="login" autocomplete="on" />
        </el-form-item>
        <el-form-item label="Password" prop="pass">
            <el-input v-model="ruleForm.pass" type="password" autocomplete="off" />
        </el-form-item>
        <el-form-item label="Confirm" prop="checkPass" v-if="!isLogin">
            <el-input v-model="ruleForm.checkPass" type="password" autocomplete="off" />
        </el-form-item>

        <el-form-item>
            <el-button type="primary" @click="submitForm(ruleFormRef)">Submit</el-button>
            <el-button @click="resetForm(ruleFormRef)">Reset</el-button>
        </el-form-item>
    </el-form>
</template>

<script lang="ts" setup>
import { onMounted, reactive, ref } from 'vue'
import type { FormInstance, FormRules } from 'element-plus'

onMounted(() => {
    console.log(1);

})

const props = defineProps({
    isLogin: {
        required: true,
        default: true,
        type: Boolean
    }
})

const ruleForm = reactive({
    login: '',
    pass: '',
    checkPass: '',
})

const ruleFormRef = ref<FormInstance>()

const register = async () => {
    try {
        const response = await (fetch('/signup', {
            method: 'POST',
            body: JSON.stringify({
                email: ruleForm.login,
                password: ruleForm.pass
            })
        }))
            .then(async res => await res.json())
        console.log(response, 'response');
        if (response.code === 201) {
            location.assign('/')
        }
    } catch (error) {
        console.warn(error);
    }
}

const auth = async () => {
    try {
        const response = await (fetch('/signin', {
            method: 'POST',
            body: JSON.stringify({
                email: ruleForm.login,
                password: ruleForm.pass
            })
        }))
            .then(async res => await res.json())

        console.log(response, 'response');

        if (response.code === 200) {
            location.assign('/')
        } else {
            alert(response.message)
        }
    } catch (error) {
        console.warn(error);
    }
}

const validatePass = (rule: any, value: any, callback: any) => {
    if (value === '') {
        callback(new Error('Please input the password'))
    } else {
        if (ruleForm.checkPass !== '') {
            if (!ruleFormRef.value) return
            ruleFormRef.value.validateField('checkPass', () => null)
        }
        callback()
    }
}
const validatePass2 = (rule: any, value: any, callback: any) => {
    if (props.isLogin) {
        callback()
        return
    }

    if (value === '') {
        callback(new Error('Please input the password again'))
    } else if (value !== ruleForm.pass) {
        callback(new Error("Two inputs don't match!"))
    } else {
        callback()
    }
}

const validateLogin = (rule: any, value: any, callback: any) => {
    if (value === '') {
        callback(new Error('Please input the login again'))
    } else {
        callback()
    }
}

const rules = reactive<FormRules<typeof ruleForm>>({
    login: [{ validator: validateLogin, trigger: 'blur' }],
    pass: [{ validator: validatePass, trigger: 'blur' }],
    checkPass: [{ validator: validatePass2, trigger: 'blur' }],
})

const submitForm = (formEl: FormInstance | undefined) => {
    if (!formEl) return
    formEl.validate((valid) => {
        if (valid) {
            if (!props.isLogin)
                register()
            else
                auth()
        } else {
            console.log('error submit!')
            return false
        }
    })
}

const resetForm = (formEl: FormInstance | undefined) => {
    if (!formEl) return
    formEl.resetFields()
}
</script>