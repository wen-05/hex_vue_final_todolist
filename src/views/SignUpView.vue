<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';

const registerField = ref({
    email: '',
    nickname: '',
    password: '',
    confirmPassword: ''
});

const url = import.meta.env.VITE_API_URL;
const router = useRouter();

const registerAccount = async () => {
    const check = (registerField.value.password !== registerField.value.confirmPassword)
    if (check) {
        alert("密碼與確認密碼不一致");
        registerField.value.password = '';
        registerField.value.confirmPassword = '';
        return;
    }

    try {
        const res = await fetch(`${url}/users/sign_up`, {
            method: 'POST',
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify(registerField.value)
        });

        if (res.ok) {
            alert("註冊成功");
            router.push('/login')
        } else {
            const data = await res.json();
            alert(data.message);
        }

    } catch (err) {
        console.log("error" + err);
    }
}
</script>

<template>
    <div id="signUpPage" class="bg-yellow">
        <div class="conatiner signUpPage vhContainer">
            <div class="side">
                <a href="#">
                    <img class="logoImg"
                        src="https://raw.githubusercontent.com/hexschool/2022-web-layout-training/main/todolist/logo.png"
                        alt="">
                </a>
                <img class="d-m-n"
                    src="https://raw.githubusercontent.com/hexschool/2022-web-layout-training/main/todolist/img.png"
                    alt="workImg">
            </div>
            <div>
                <form class="formControls">
                    <h2 class="formControls_txt">註冊帳號</h2>
                    <label class="formControls_label" for="email">Email</label>
                    <input v-model="registerField.email" class="formControls_input" type="text" id="email" name="email"
                        placeholder="請輸入 email" required>

                    <label class="formControls_label" for="name">您的暱稱</label>
                    <input v-model="registerField.nickname" class="formControls_input" type="text" name="name" id="name"
                        placeholder="請輸入您的暱稱" required>

                    <label class="formControls_label" for="pwd">密碼</label>
                    <input v-model="registerField.password" class="formControls_input" type="password" name="pwd"
                        id="pwd" placeholder="請輸入密碼" required>
                    <span>密碼長度至少六個字元</span>

                    <label class="formControls_label" for="pwd">再次輸入密碼</label>
                    <input v-model="registerField.confirmPassword" class="formControls_input" type="password"
                        name="confirm_pwd" id="confirm_pwd" placeholder="請再次輸入密碼" required>

                    <input @click="registerAccount" class="formControls_btnSubmit" type="button" value="註冊帳號">
                    <RouterLink to="/login" class="formControls_btnLink">登入</RouterLink>
                </form>
            </div>
        </div>

    </div>
</template>