<template>
  <table class="mail-table">
    <tbody>
    <tr v-for="email in unarchievedEmails"
        :key="email.id"
        :class="['clickable', email.read ? 'read' : '']"
        @click="openEmail(email)">
      <td>
        <input type="checkbox" />
      </td>
      <td>{{email.from}}</td>
      <td>
        <p><strong>{{email.subject}}</strong> - {{email.body}}</p>
      </td>
      <td class="date">{{format(new Date(email.sentAt), 'MMM do yyyy')}}</td>
      <td><button @click="archiveEmail(email)">Archive</button></td>
    </tr>
    </tbody>
  </table>
  <ModalView v-if="openedEmail" @closeModal="closeModal">
    <MailView :email="openedEmail" @changeEmail="changeEmail" />
  </ModalView>
</template>

<script>
import {ref} from 'vue';
import {format} from "date-fns";
import axios from 'axios';
import MailView from '@/components/MailView.vue';
import ModalView from '@/components/ModalView.vue';

export default {
  name: "MailTable",
  // data() cannot be async
  // to perform data fetching before component renders, use async setup() + <Suspense>
  async setup(){
    let { data: emails } = await axios.get('http://localhost:3000/emails/')
    return {
      format,
      emails: ref(emails),
      openedEmail: ref(null),
    }
  },
  components: {
    MailView,
    ModalView,
  },
  computed: {
    sortedEmails() {
      return this.emails.sort((e1, e2) => {
        return e1.sentAt < e2.sentAt ? 1 : -1;
      })
    },
    unarchievedEmails() {
      return this.sortedEmails.filter(e => !e.archived);
    },
  },
  methods: {
    openEmail(email) {
      this.openedEmail = email;

      if(email) {
        email.read = true;
        this.updateEmail(email);
      }
    },
    archiveEmail(email) {
      email.archived = true;
      this.updateEmail(email);
    },
    updateEmail(email) {
      axios.put(`http://localhost:3000/emails/${email.id}`, email);
    },
    changeEmail({toggleRead, toggleArchive, save, closeModal, changeIndex}) {
      let email = this.openedEmail;
      if(toggleRead) { email.read = !email.read }
      if(toggleArchive) { email.archived = !email.archived }
      if(save) { this.updateEmail(email) }
      if(closeModal) { this.openedEmail = null }

      if(changeIndex) {
        let emails = this.unarchievedEmails
        let currentIndex = this.unarchievedEmails.indexOf(email)
        let newEmail = emails[currentIndex + changeIndex]
        this.openEmail(newEmail)
      }
    },
    closeModal() {
      this.openedEmail = null;
    }
  }
}
</script>

<style scoped>

</style>
