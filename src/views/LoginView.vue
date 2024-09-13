<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';

const loginField = ref({
    email: '',
    password: ''
})

let isDisabled = ref(false);

const url = import.meta.env.VITE_API_URL;
const router = useRouter();
const dashboard = async () => {
    const {email, password} = loginField.value;
    if (!email || !password){
        alert("所有欄位為必填，請確實填寫!");
        return;
    }

    isDisabled.value = true;

    try {
        const res = await fetch(`${url}/users/sign_in`, {
            method: 'POST',
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify(loginField.value)
        });
        
        if (res.ok) {
            const data = await res.json();
            const { exp, token, nickname } = data;
            document.cookie = `authTokenCookie=${token}; expires=${exp}; path=/ `;
            document.cookie = `authNickname=${nickname}; expires=${exp}; path=/ `;
            
            alert("登入成功");
            router.push('/todo')
        } else {
            switch (res.status) {
                case 400:
                    alert('登入失敗：欄位驗證失敗');
                    break;
                case 401:
                    alert('登入失敗：帳號密碼驗證錯誤');
                    break;
                case 404:
                    alert('登入失敗：用戶不存在');
                    break;
            }
        }

    } catch (err) {
        console.log("error" + err);
    } finally {
        isDisabled.value = false;
    }
}
</script>

<template>
    <div id="loginPage" class="bg-yellow">
        <div class="conatiner loginPage vhContainer ">
            <div class="side">
                <a href="#"><img class="logoImg"
                        src="https://raw.githubusercontent.com/hexschool/2022-web-layout-training/main/todolist/logo.png"
                        alt=""></a>
                <img class="d-m-n"
                    src="https://raw.githubusercontent.com/hexschool/2022-web-layout-training/main/todolist/img.png"
                    alt="workImg">
            </div>
            <div>
                <form class="formControls">
                    <h2 class="formControls_txt">最實用的線上代辦事項服務</h2>

                    <label class="formControls_label" for="email">Email</label>
                    <input v-model="loginField.email" class="formControls_input" type="text" id="email" name="email"
                        placeholder="請輸入 email" required>
                    <span>此欄位不可留空</span>

                    <label class="formControls_label" for="pwd">密碼</label>
                    <input v-model="loginField.password" class="formControls_input" type="password" name="pwd" id="pwd"
                        placeholder="請輸入密碼" required>
                    <span>密碼長度至少六個字元</span>

                    <input @click="dashboard" class="formControls_btnSubmit" type="button" value="登入" :disabled="isDisabled">
                    <RouterLink to="/signup" class="formControls_btnLink">註冊帳號</RouterLink>
                </form>
            </div>
        </div>
    </div>
    <RouterView />
</template>