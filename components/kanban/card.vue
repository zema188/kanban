<script setup>
const props = defineProps({
    card: {
        type: Object,
        required: true
    },
    listID: {
        type: Number,
        required: true
    }
})
const emit = defineEmits('updateCard')

let oldText = null
let readonly = ref(true)

const focusText = () => {
    readonly.value = false
}

const blurText = () => {
    readonly.value = true
    if(oldText !== props.card.text && props.card.text) {
        let openRequest = indexedDB.open("kanban", 1);

        openRequest.onsuccess = function() {
            let db = openRequest.result;
            let transaction = db.transaction("lists", "readwrite");
            let lists = transaction.objectStore("lists");

            try {
                let getRequest = lists.get(props.listID);

                getRequest.onsuccess = function() {
                    let list = getRequest.result;

                    if (list) {
                        // Находим карточку в списке по id
                        let cardToUpdate = list.cards.find(card => card.id === props.card.id);

                        if (cardToUpdate) {
                            // Обновляем текст карточки
                            cardToUpdate.text = props.card.text;

                            // Обновляем список в IndexedDB
                            let updateRequest = lists.put(list);

                            updateRequest.onsuccess = function() {
                                oldText = props.card.text
                                console.log(`Card with ID ${props.card.id} updated in the list with ID ${props.listID}.`);
                            };

                            updateRequest.onerror = function() {
                                console.log("Error updating list:", updateRequest.error);
                            };
                        } else {
                            console.log(`Card with ID ${id} not found in the list with ID ${props.listID}.`);
                        }
                    } else {
                        console.log(`List with ID ${props.listID} not found.`);
                    }
                };

                getRequest.onerror = function() {
                    console.log("Error getting list:", getRequest.error);
                };
            } catch (error) {
                console.error("Error updating card in IndexedDB:", error);
            }
        };

        openRequest.onerror = function(event) {
            console.log("Error opening/creating the database", event.target.error);
        };
    } else {
        props.card.text = oldText
    }
}

onMounted(() => {
    oldText = props.card.text
})

</script>

<template>
    <div class="rounded-lg bg-white mb-4 p-6 relative"
        style="box-shadow: 0px 1px 1px #091e4240, 0px 0px 1px #091e424f"
    >
        <textarea class="w-full read-only:bg-white-100 resize-none"
            v-model="props.card.text"
            ref="textArea"
            @focus="focusText"
            @blur="blurText"
            :readonly="readonly"
        >
        </textarea>
        <div class="flex gap-x-2 absolute right-2 top-1">
            <!-- <button
            
            >
                <svg width="24" height="24" role="presentation" focusable="false" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" clip-rule="evenodd" d="M7.82034 14.4893L9.94134 16.6103L18.4303 8.12131L16.3093 6.00031H16.3073L7.82034 14.4893ZM17.7233 4.58531L19.8443 6.70731C20.6253 7.48831 20.6253 8.7543 19.8443 9.53531L10.0873 19.2933L5.13734 14.3433L14.8943 4.58531C15.2853 4.19531 15.7973 4.00031 16.3093 4.00031C16.8203 4.00031 17.3323 4.19531 17.7233 4.58531ZM5.20094 20.4097C4.49794 20.5537 3.87694 19.9327 4.02094 19.2297L4.80094 15.4207L9.00994 19.6297L5.20094 20.4097Z" fill="currentColor"></path></svg>
            </button> -->
            <button 
                @click="emit('deleteCard', props.card.id)"
            >
                <svg class="w-5" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <path d="M20.7457 3.32851C20.3552 2.93798 19.722 2.93798 19.3315 3.32851L12.0371 10.6229L4.74275 3.32851C4.35223 2.93798 3.71906 2.93798 3.32854 3.32851C2.93801 3.71903 2.93801 4.3522 3.32854 4.74272L10.6229 12.0371L3.32856 19.3314C2.93803 19.722 2.93803 20.3551 3.32856 20.7457C3.71908 21.1362 4.35225 21.1362 4.74277 20.7457L12.0371 13.4513L19.3315 20.7457C19.722 21.1362 20.3552 21.1362 20.7457 20.7457C21.1362 20.3551 21.1362 19.722 20.7457 19.3315L13.4513 12.0371L20.7457 4.74272C21.1362 4.3522 21.1362 3.71903 20.7457 3.32851Z" fill="#0F0F0F"></path> </g></svg>
            </button>
        </div>
    </div>
</template>


<style lang="scss" scoped>

</style>