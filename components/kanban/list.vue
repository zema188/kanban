<script setup>
import { v4 as uuidv4 } from 'uuid';

const props = defineProps({
    list: {
        type: Object,
        required: true
    }
})
const emit = defineEmits(['deleteList', 'updateTitle', 'postCard'])

let oldTitle = null
let readonly = ref(true)
let addCardIsActive = ref(false)
let areaRef = ref(null)
let areaText = ref('')

const focusInputTitle = () => {
    readonly.value = false
}

const blurInputTitle = () => {
    readonly.value = true
    if(oldTitle !== props.list.title && props.list.title) {

        let openRequest = indexedDB.open("kanban", 1);

        openRequest.onsuccess = function () {
            let db = openRequest.result;
            let transaction = db.transaction("lists", "readwrite");
            let lists = transaction.objectStore("lists");

            try {
                let getRequest = lists.get(props.list.id);

                getRequest.onsuccess = function () {
                    let existingList = getRequest.result;

                    if (existingList) {
                        existingList.title = props.list.title;

                        let updateRequest = lists.put(existingList);

                        updateRequest.onsuccess = function () {
                            console.log(`List with ID ${props.list.id} updated successfully.`);
                            oldTitle = props.list.title
                        }

                        updateRequest.onerror = function () {
                            console.log("Error updating list:", updateRequest.error);
                        }
                    } else {
                        console.log(`List with ID ${props.list.id} not found.`);
                    }
                }

                getRequest.onerror = function () {
                    console.log("Error getting list:", getRequest.error);
                }
            } catch (error) {
                console.error("Error updating list title in IndexedDB:", error);
            }
        }

        openRequest.onerror = function (event) {
            console.log("Error opening/creating the database", event.target.error);
        }
    } else {
        props.list.title = oldTitle
    }
}

const openAddCardForm = () => {
    addCardIsActive.value = true
}

const postCard = () => {
    if(areaText.value) {
        let openRequest = indexedDB.open("kanban", 1);

        openRequest.onsuccess = function() {
            let db = openRequest.result;
            let transaction = db.transaction("lists", "readwrite");
            let lists = transaction.objectStore("lists");

            try {
                let getRequest = lists.get(props.list.id);

                getRequest.onsuccess = function() {
                    let list = getRequest.result;

                    if (list) {
                        // Создаем новую карточку
                        let card = {
                            id: uuidv4(),
                            text: areaText.value,
                            created: new Date().toISOString()
                        };

                        // Добавляем карточку в список
                        list.cards.push(card);

                        // Обновляем список в IndexedDB
                        let updateRequest = lists.put(list);

                        updateRequest.onsuccess = function() {
                            // Обновляем данные в списке для отображения в UI
                            let insertedCard = { ...card}; // 
                            props.list.cards.push(card)

                            areaText.value = ''
                            console.log(`Card added to the list with ID ${props.list.id}.`);
                        };

                        updateRequest.onerror = function() {
                            console.log("Error updating list:", updateRequest.error);
                        };
                    } else {
                        console.log(`List with ID ${props.list.id} not found.`);
                    }
                };

                getRequest.onerror = function() {
                    console.log("Error getting list:", getRequest.error);
                };
            } catch (error) {
                console.error("Error posting card to IndexedDB:", error);
            }
        };

        openRequest.onerror = function(event) {
            console.log("Error opening/creating the database", event.target.error);
        };
    }
}

const deleteCard = (id) => {
    let openRequest = indexedDB.open("kanban", 1);

    openRequest.onsuccess = function() {
        let db = openRequest.result;
        let transaction = db.transaction("lists", "readwrite");
        let lists = transaction.objectStore("lists");

        try {
            let getRequest = lists.get(props.list.id);

            getRequest.onsuccess = function() {
                let list = getRequest.result;

                if (list) {
                    // Удаляем карточку из списка по id
                    list.cards = list.cards.filter(card => card.id !== id);

                    // Обновляем список в IndexedDB
                    let updateRequest = lists.put(list);

                    updateRequest.onsuccess = function() {
                        // Обновляем данные в списке для отображения в UI
                        props.list.cards = props.list.cards.filter(card => card.id !== id);
                        console.log(`Card with ID ${id} deleted from the list with ID ${props.list.id}.`);
                    };

                    updateRequest.onerror = function() {
                        console.log("Error updating list:", updateRequest.error);
                    };
                } else {
                    console.log(`List with ID ${props.list.id} not found.`);
                }
            };

            getRequest.onerror = function() {
                console.log("Error getting list:", getRequest.error);
            };
        } catch (error) {
            console.error("Error deleting card from IndexedDB:", error);
        }
    };

    openRequest.onerror = function(event) {
        console.log("Error opening/creating the database", event.target.error);
    };
}




onMounted(() => {
    oldTitle = props.list.title
})
</script>

<template>
    <div class="rounded-lg bg-gray-100 w-72 p-2 h-fit">
        <div class="flex justify-between items-center mb-5 p-2">
            <input class="read-only:bg-gray-100"
                v-model="props.list.title"
                @focus="focusInputTitle"
                @blur="blurInputTitle"
                :readonly="readonly"
            />
            <button
                @click="emit('deleteList', props.list.id)"
            >
                <svg class="w-5" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <path d="M20.7457 3.32851C20.3552 2.93798 19.722 2.93798 19.3315 3.32851L12.0371 10.6229L4.74275 3.32851C4.35223 2.93798 3.71906 2.93798 3.32854 3.32851C2.93801 3.71903 2.93801 4.3522 3.32854 4.74272L10.6229 12.0371L3.32856 19.3314C2.93803 19.722 2.93803 20.3551 3.32856 20.7457C3.71908 21.1362 4.35225 21.1362 4.74277 20.7457L12.0371 13.4513L19.3315 20.7457C19.722 21.1362 20.3552 21.1362 20.7457 20.7457C21.1362 20.3551 21.1362 19.722 20.7457 19.3315L13.4513 12.0371L20.7457 4.74272C21.1362 4.3522 21.1362 3.71903 20.7457 3.32851Z" fill="#0F0F0F"></path> </g></svg>
            </button>
        </div>
        <kanban-card
            v-for="(card, index) of props.list.cards" :key="index" :card="card"
            :listID="props.list.id"
            @delete-card="(id) => deleteCard(id)"
            @update-card="(id, text) => updateCard(id, text)"
        />
        <button class="flex items-center gap-x-2 bg-gray-100 rounded-lg p-2 hover:bg-sky-700 hover:text-white"
            v-if="!addCardIsActive"
            @click="openAddCardForm()"
        >
            <svg class="w-5 stroke-current" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <path d="M4 12H20M12 4V20" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path> </g></svg>    
            <span class="pb-1">
                Добавьте карточку
            </span>
        </button>
        <div
            v-else
        >
            <textarea class="rounded-lg p-2 mb-2 w-full" ref="areaRef" v-model="areaText"></textarea>
            <div class="flex items-center"
            >
                <button class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 me-2 dark:bg-blue-600 dark:hover:bg-blue-700 focus:outline-none dark:focus:ring-blue-800"
                    @click="postCard()"
                >
                    Добавить карточку
                </button>
                <button
                    @click="addCardIsActive = false"
                >
                    <svg width="24" height="24" role="presentation" focusable="false" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="M10.5858 12L5.29289 6.70711C4.90237 6.31658 4.90237 5.68342 5.29289 5.29289C5.68342 4.90237 6.31658 4.90237 6.70711 5.29289L12 10.5858L17.2929 5.29289C17.6834 4.90237 18.3166 4.90237 18.7071 5.29289C19.0976 5.68342 19.0976 6.31658 18.7071 6.70711L13.4142 12L18.7071 17.2929C19.0976 17.6834 19.0976 18.3166 18.7071 18.7071C18.3166 19.0976 17.6834 19.0976 17.2929 18.7071L12 13.4142L6.70711 18.7071C6.31658 19.0976 5.68342 19.0976 5.29289 18.7071C4.90237 18.3166 4.90237 17.6834 5.29289 17.2929L10.5858 12Z" fill="currentColor"></path></svg>
                </button>
            </div>
        </div>
    </div>
</template>


<style lang="scss" scoped>

</style>