<script setup>
import { ref } from "vue"

const listsData = ref([])

let addListFormIsActive = ref(false)
let inputAddList = ref(null)
let nameList = ref('')

const initData = async () => {
    try {
        let openRequest = indexedDB.open("kanban", 1)

        openRequest.onupgradeneeded = function() {
            let db = openRequest.result
            if (!db.objectStoreNames.contains('lists')) {
                let objectStore = db.createObjectStore('lists', { keyPath: 'id', autoIncrement: true })
            }
        }

        openRequest.onerror = function() {
            console.error("Error", openRequest.error)
        }

        const db = await new Promise((resolve, reject) => {
            openRequest.onsuccess = function() {
                resolve(openRequest.result)
            }
            openRequest.onblocked = function() {
                reject(new Error("Blocked by another open connection"))
            }
        })

        const transaction = db.transaction("lists", "readwrite")
        const lists = transaction.objectStore("lists")

        const objects = await new Promise((resolve, reject) => {
            const request = lists.getAll()
            request.onsuccess = function(event) {
                resolve(event.target.result)
            }
            request.onerror = function() {
                reject(new Error("Error fetching objects from object store"))
            }
        })
        listsData.value = [...objects]
        db.onversionchange = function() {
            db.close()
            alert("База данных устарела, пожалуйста, перезагрузите страницу.")
        }
    } catch (error) {
        console.error("An error occurred:", error)
    }
}

const openAddListForm = () => {
    addListFormIsActive.value = true
    // inputAddList.value.focus()
}

const postList = () => {
    let openRequest = indexedDB.open("kanban", 1)

    openRequest.onsuccess = function() {
        let db = openRequest.result
        let transaction = db.transaction("lists", "readwrite")

        let lists = transaction.objectStore("lists")

        let list = {
            title: nameList.value,
            cards: [],
            created: new Date().toISOString()
        }

        try {

            let request = lists.put(list)

            request.onsuccess = function() {
                let insertedList = { ...list, id: request.result }; // Добавляем id из результата запроса

                listsData.value.push(insertedList);
                addListFormIsActive.value = false
                nameList.value = ''
            }

            request.onerror = function() {
                console.log("Ошибка", request.error)
            }
        } catch (error) {
            console.error("Error adding data to IndexedDB:", error)
        }
    }

    openRequest.onerror = function(event) {
        console.log("Ошибка при открытии/создании базы данных", event.target.error)
    }
}

const deleteList = (id) => {
    let openRequest = indexedDB.open("kanban", 1);

    openRequest.onsuccess = function() {
        let db = openRequest.result;
        let transaction = db.transaction("lists", "readwrite");
        let lists = transaction.objectStore("lists");

        try {
            let request = lists.delete(id);

            request.onsuccess = function() {
                // Handle success, e.g., update UI or other logic
                console.log(`List with ID ${id} deleted successfully.`);
                
                // Assuming listsData.value is an array containing the lists
                listsData.value = listsData.value.filter(list => list.id !== id);
            }

            request.onerror = function() {
                console.log("Error deleting list:", request.error);
            }
        } catch (error) {
            console.error("Error deleting list from IndexedDB:", error);
        }
    }

    openRequest.onerror = function(event) {
        console.log("Error opening/creating the database", event.target.error);
    }
}

onMounted(() => {
    initData()
})

</script>

<template>
    <div class="flex gap-x-10 p-5 items-start">
        <div class="flex gap-x-10">
            <kanban-list
                v-for="(list, index) of listsData" :key="index" :list="list"
                @delete-list="(id) => deleteList(id)"
            />
        </div>
        <button class="flex items-center gap-x-2 bg-gray-100 rounded-lg p-2 whitespace-nowrap"
            v-if="!addListFormIsActive"
            @click="openAddListForm()"
        >
            <svg class="w-5" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <path d="M4 12H20M12 4V20" stroke="#000000" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path> </g></svg>    
            <span class="pb-1">
                Добавьте колонку
            </span>
        </button>
        <div class="bg-gray-100 rounded-lg p-2"
            v-else
        >
            <input class="rounded-lg p-2 mb-2"
                ref="inputAddList"
                v-model="nameList"
            >
            <div class="flex items-center">
                <button class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 me-2 dark:bg-blue-600 dark:hover:bg-blue-700 focus:outline-none dark:focus:ring-blue-800"
                    @click="postList()"
                >
                    Добавить список
                </button>
                <button
                    @click="addListFormIsActive = false"
                >
                    <svg width="24" height="24" role="presentation" focusable="false" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="M10.5858 12L5.29289 6.70711C4.90237 6.31658 4.90237 5.68342 5.29289 5.29289C5.68342 4.90237 6.31658 4.90237 6.70711 5.29289L12 10.5858L17.2929 5.29289C17.6834 4.90237 18.3166 4.90237 18.7071 5.29289C19.0976 5.68342 19.0976 6.31658 18.7071 6.70711L13.4142 12L18.7071 17.2929C19.0976 17.6834 19.0976 18.3166 18.7071 18.7071C18.3166 19.0976 17.6834 19.0976 17.2929 18.7071L12 13.4142L6.70711 18.7071C6.31658 19.0976 5.68342 19.0976 5.29289 18.7071C4.90237 18.3166 4.90237 17.6834 5.29289 17.2929L10.5858 12Z" fill="currentColor"></path></svg>
                </button>
            </div>
        </div>
    </div>
</template>


<style lang="scss" scoped>

</style>