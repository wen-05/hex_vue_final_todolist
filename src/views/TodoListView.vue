<script setup>
import { computed, onMounted, ref } from 'vue';
import { useRouter } from 'vue-router';

const url = import.meta.env.VITE_API_URL;
const router = useRouter();

// 使用者驗證與登出
const token = document.cookie.replace(
    /(?:(?:^|.*;\s*)authTokenCookie\s*\=\s*([^;]*).*$)|^.*$/,
    "$1",
);

const nickname = document.cookie.replace(
    /(?:(?:^|.*;\s*)authNickname\s*\=\s*([^;]*).*$)|^.*$/,
    "$1",
);

// 驗證登入
onMounted(async () => {
    try {
        const res = await fetch(`${url}/users/checkout`, {
            headers: {
                Authorization: token
            }
        });

        if (res.ok) {
            alert(`哈囉 ${nickname}! 開始使用你的代辦清單吧~ `)
            getTodo();
        } else {
            alert('已登出或無權使用')
            router.push('/login')
        }
    } catch (err) {
        console.log(err.message);
    }
})

// 登出
const signOut = async () => {
    try {
        const res = await fetch(`${url}/users/sign_out`, {
            method: 'POST',
            headers: {
                Authorization: token
            }
        })

        if (res.ok) {
            alert('登出成功')
            router.push('/login')
        } else {
            alert('登出失敗')
        }
    } catch (err) {
        console.log(err.message);
    }
}


// 代辦事項相關操作
const todos = ref([]);
const newItem = ref('');

const getTodo = async () => {
    try {
        const res = await fetch(`${url}/todos`, {
            headers: {
                Authorization: token
            }
        })

        const result = await res.json();
        todos.value = result.data;

    } catch (err) {
        console.log(err.message);
    }
}

const addTodo = async () => {
    if (newItem.value === '') {
        alert('欄位不可空白');
        return;
    }

    try {
        const res = await fetch(`${url}/todos`, {
            method: 'POST',
            headers: {
                "Content-Type": "application/json",
                Authorization: token
            },
            body: JSON.stringify({
                content: newItem.value
            })
        })

        if (res.ok) {
            getTodo();
            newItem.value = '';
            alert('新增成功')
            statusTodos.value = 'all';
        } else {
            alert('新增失敗')
        }
    } catch (err) {
        console.log(err.message);
    }
}

const editTodo = async (id) => {
    const newItemContent = prompt('請輸入修改內容文字:')

    if (newItemContent === null) {
        alert('取消編輯');
        return;
    }

    if (newItemContent === '') {
        alert('內容不可空白');
        return;
    }

    try {
        const res = await fetch(`${url}/todos/${id}`, {
            method: 'PUT',
            headers: {
                "Content-Type": "application/json",
                Authorization: token
            },
            body: JSON.stringify({
                content: newItemContent
            })
        })

        if (res.ok) {
            getTodo();
            alert('更新成功');
        } else {
            alert('更新失敗');
        }
    } catch (err) {
        console.log(err.message);
    }
}

const delTodo = async (id) => {
    try {
        const res = await fetch(`${url}/todos/${id}`, {
            method: 'DELETE',
            headers: {
                Authorization: token
            }
        })

        const result = await res.json();
        if (res.ok) {
            alert(result.message)
            getTodo();
        } else {
            alert(result.message)
        }
    } catch (err) {
        console.log(err.message);
    }
}

const toggleTodo = async (id) => {
    try {
        const res = await fetch(`${url}/todos/${id}/toggle`, {
            method: 'PATCH',
            headers: {
                Authorization: token
            }
        })

        const result = await res.json();
        if (res.ok) {
            alert(result.message)
            getTodo();
        } else {
            alert(result.message)
        }
    } catch (err) {
        console.log(err.message);
    }
}

const unFinishItem = computed(() => {
    return todos.value.filter(todo => todo.status === false).length;
});

const statusTodos = ref('all');

const filteredTodos = computed(() => {
    if (statusTodos.value === 'finish') {
        return todos.value.filter(todo => todo.status === true);
    } else if (statusTodos.value === 'unfinish') {
        return todos.value.filter(todo => todo.status === false);
    } else {
        return todos.value;
    }
});
</script>

<template>
    <div id="todoListPage" class="bg-half">
        <nav>
            <h1><a href="#">ONLINE TODO LIST</a></h1>
            <ul>
                <li>
                    <RouterLink to="/todo"><span>{{ nickname }}的代辦</span></RouterLink>
                </li>
                <li>
                    <a href="#" @click.prevent="signOut">登出</a>
                </li>
            </ul>
        </nav>
        <div class="conatiner todoListPage vhContainer">
            <div class="todoList_Content">
                <div class="inputBox">
                    <input v-model="newItem" type="text" placeholder="請輸入待辦事項">
                    <a @click.prevent="addTodo" href="#">
                        <i class="fa fa-plus"></i>
                    </a>
                </div>

                <div v-if="todos.length === 0">目前尚無待辦事項</div>

                <div v-else class="todoList_list">
                    <ul class="todoList_tab">
                        <li>
                            <a href="#" :class="{ active: statusTodos === 'all' }"
                                @click.prevent="statusTodos = 'all'">全部</a>
                        </li>
                        <li>
                            <a href="#" :class="{ active: statusTodos === 'unfinish' }"
                                @click.prevent="statusTodos = 'unfinish'">待完成</a>
                        </li>
                        <li>
                            <a href="#" :class="{ active: statusTodos === 'finish' }"
                                @click.prevent="statusTodos = 'finish'">已完成</a>
                        </li>
                    </ul>
                    <div class="todoList_items">
                        <ul class="todoList_item">
                            <li v-for="todo in filteredTodos" :key="todo.id">
                                <label class="todoList_label">
                                    <input class="todoList_input" type="checkbox" @click="toggleTodo(todo.id)"
                                        :checked="todo.status">
                                    <span>{{ todo.content }}</span>
                                </label>
                                <a @click.prevent="editTodo(todo.id)" href="#">
                                    <i class="fa-solid fa-pen-to-square"></i>
                                </a>
                                <a @click.prevent="delTodo(todo.id)" href="#">
                                    <i class="fa fa-times"></i>
                                </a>
                            </li>
                        </ul>
                        <div class="todoList_statistics">
                            <p v-if="statusTodos === 'all'"> {{ unFinishItem }}個待完成項目</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>