<template>
    <div class="chat">
        <div class="messager">
            <div class="messager-header">
                <div class="contact-info">
                    <div @click="showContactInfo = !showContactInfo" class="contact-name">{{ contactInfo.firstName }} {{ contactInfo.lastName }}</div>
                    <!-- <div class="contact-status-online"></div> -->
                    <!-- <div class="contact-status-offline"></div> -->
                </div>
            </div>
            <Messages :messages="messages"/>
            <input type="text" class="write-message-input" @keydown.enter="handleSubmit" v-model="message" placeholder="Start typing your message..." />
        </div>

        <div v-if="showContactInfo" class="user-info">
            <img :src="contactInfo.avatar" />
            <div class="contact-name">{{ contactInfo.firstName }} {{ contactInfo.lastName }}</div>
            <div class="contact-geo">{{ contactInfo.location }}</div>
            <div class="additional-contacts">
                <div v-if="contactInfo.email">
                    <i class="bi bi-envelope"></i>
                    <div> {{ contactInfo.email }} </div>
                </div>
                <div v-if="contactInfo.phone">
                    <i class="bi bi-telephone-fill"></i>
                    <div>{{ contactInfo.phone }}</div>
                </div>
                <div v-if="contactInfo.website">
                    <i class="bi bi-globe"></i>
                    <div>{{ contactInfo.website }}</div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import Messages from "@/components/Messages"
import { socket } from '@/socket'
import { getGroup } from "@/reselects/getGroup"

export default {
    components: {
        Messages
    },
    data() {
        return {
            showContactInfo: true,
            messages: [],
            message: "",
            contactInfo: {
                firstName: "",
                lastName: "",
                avatar: "",
                website: "",
                phone: "",
                location: ""
            }
        }
    },
    beforeRouteEnter (to, from, next) {
        getGroup(to.params.id, (group) => {
            next(wm => wm.setGroup(group))
        })
    },
    beforeRouteUpdate (to, from, next) {
        getGroup(to.params.id, (group) => {
            this.setGroup(group)
            next()
        })
    },
    methods: {
        handleSubmit() {
            if (this.message.length >= 1) {
                socket.emit("message", {
                    text: this.message,
                    authorId: this.$store.state.userId,
                    groupId: this.$route.params.id
                })
                this.message = ""
            }
        },
        setGroup(group) {
            const groupId = group.id
            const contact = group.users.find(user => user.userId !== this.$store.state.userId)
            this.messages = group.messages
            this.contactInfo = {...contact}
            socket.on(`message in group: ${groupId}`, (data) => { 
                this.$store.commit("newMessage", data)
            })
        }
    }
}
</script>

<style lang="scss" scoped>
    .chat {
        position: relative;
        width: 80vw;
        display: flex;
        justify-content: space-between;
        .messager {
            width: 100%;
        }
        .user-info {
            width: 45%;
            background: #fff;
            display: flex;
            text-align: center;
            flex-direction: column;
            img {
                border-radius: 150px;
                width: 250px;
                margin: 30px 130px;
                height: 250px;
            }
            .contact-geo {
                font-size: 14px;
                color: #abb2b9;
            }
        }
    }
    .write-message-input {
        margin-top: 5vh;
        width: 100%;
        border: none;
        padding: 20px 40px;
        background: #f0f2f5;
        color: #9c9c9d;
        &::placeholder {
            color: #9c9c9d;
        }
    }
    .messager-header {
        padding-top: 50px;
        padding-left: 50px;
        padding-bottom: 10px;
    }
    .contact-info {
        display: flex;
        flex-wrap: wrap;
        align-items: center;
        .contact-status-online, .contact-status-offline {
            margin-left: 10px;
        }
    }
    .contact-name {
        font-size: 23px;
        font-weight: 400;
        cursor: pointer;
    }
    .contact-status {
        color:  #808b96;
        font-size: 14px;
    }

    .additional-contacts {
        margin-top: 30px;
        div {
            display: flex;
            align-items: center;
            padding: 15px;
            justify-content: flex-start;
        }
        div:nth-child(odd) {
            border-bottom: 0.25px solid #abb2b9;
            border-top: 0.25px solid #abb2b9;
        }
    }
</style>