<template>
  <q-page class="relative-position">
    <q-scroll-area class="absolute full-height full-width">
      <div class="q-py-lg q-px-md row items-end q-col-gutter-md">
        <div class="col">
          <q-input
            bottom-slots
            v-model="newQweet"
            placeholder="What's happening?"
            maxlength="280"
            class="new-qweet"
            counter
            autogrow
          >
            <template v-slot:before>
              <q-avatar size="xl">
                <img src="/ninja.png" />
              </q-avatar>
            </template>
          </q-input>
        </div>
        <div class="col col-shrink">
          <q-btn
            color="primary"
            label="Qweet"
            :disable="!newQweet"
            class="q-mb-lg"
            @click="addNewQweet"
            unelevated
            rounded
            no-caps
          />
        </div>
      </div>
      <q-separator size="10px" color="grey-2" class="divider" />

      <q-list separator>
        <transition-group
          appear
          enter-active-class="animated fadeIn"
          leave-active-class="animated fadeOut"
        >
          <q-item class="q-py-md qweet" v-for="qweet in qweets" :key="qweet.id">
            <!-- <q-item-section avatar top>
              <q-avatar size="xl">
                <img src="/ninja.png" />
              </q-avatar>
            </q-item-section> -->

            <q-item-section>
              <q-item-label class="text-subtitle1">
                <strong>Stephen Lai </strong>
                <span class="text-grey-7">
                  @stephen__lai
                  <br class="lt-sm" />&bull;
                  {{ formatDistanceToNow(qweet.createdAt.toDate()) }}
                </span>
              </q-item-label>
              <q-item-label class="qweet-content text-body1">
                {{ qweet.content }}
              </q-item-label>
              <div class="qweet-icons row justify-between q-mt-sm">
                <q-btn
                  color="grey"
                  icon="far fa-comment"
                  size="sm"
                  flat
                  round
                />
                <q-btn
                  color="grey"
                  icon="fas fa-retweet"
                  size="sm"
                  flat
                  round
                />
                <q-btn
                  :color="qweet.liked ? 'red-5' : 'grey'"
                  @click="toggleLiked(qweet)"
                  :icon="qweet.liked ? 'fas fa-heart' : 'far fa-heart'"
                  size="sm"
                  flat
                  round
                />
                <q-btn
                  color="grey"
                  icon="fas fa-trash"
                  size="sm"
                  @click="deleteQweet(qweet.id)"
                  flat
                  round
                />
              </div>
            </q-item-section>
          </q-item>
        </transition-group>
      </q-list>
    </q-scroll-area>
  </q-page>
</template>

<script>
import { ref, onMounted } from "vue";
import { fireDB } from "../database/firebase";
import { formatDistanceToNow } from "date-fns";

export default {
  setup() {
    const newQweet = ref("");
    const qweets = ref([]);

    const addNewQweet = () => {
      const temp = {
        content: newQweet.value,
        createdAt: new Date(),
        liked: false,
      };
      fireDB.collection("qweets").add(temp);
      newQweet.value = "";
    };

    const deleteQweet = (id) => {
      fireDB.collection("qweets").doc(id).delete();
    };

    const toggleLiked = (qweet) => {
      fireDB.collection("qweets").doc(qweet.id).update({
        liked: !qweet.liked,
      });
    };

    onMounted(() => {
      fireDB
        .collection("qweets")
        .orderBy("createdAt", "desc")
        .onSnapshot((snap) => {
          /* 1st way to retrieve dataq from firestore */
          // qweets.value = snap.docs.map((doc) => {
          //   return { ...doc.data(), id: doc.id };
          // });

          /* 2nd way to retrieve dataq from firestore */
          let results = [];
          snap.docs.forEach((doc) => {
            doc.data().createdAt && results.push({ ...doc.data(), id: doc.id });
          });
          qweets.value = results;
        });
    });

    return {
      newQweet,
      qweets,
      formatDistanceToNow,
      addNewQweet,
      deleteQweet,
      toggleLiked,
    };
  },
};
</script>

<style lang="scss" scoped>
.new-qweet {
  textarea {
    font-size: 19px;
    line-height: 1.4 !important;
  }
}
.divider {
  border-top: 1px solid;
  border-bottom: 1px solid;
  border-color: $grey-4;
}
.qweet-content textarea {
  white-space: pre-line;
}
.qweet-icons {
  margin-left: -5px;
}
.qweet:not(:first-child) {
  border-top: 1px solid rgba(0, 0, 0, 0.12);
}
.active {
  background: red;
}
</style>
