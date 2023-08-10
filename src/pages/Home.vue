<template>
  <q-page class="relative-position">
    <q-scroll-area class="absolute full-height full-width">
      <div class="q-py-lg q-px-md row items-end q-col-gutter-md">
        <div class="col">
          <q-input
              bottom-slots
              v-model="newTweetContent"
              class="new-tweet"
              placeholder="What's happening?"
              counter
              maxlength="280"
              autogrow
          >
            <template v-slot:before>
              <q-avatar size="xl">
                <img src="https://cdn.quasar.dev/img/avatar5.jpg">
              </q-avatar>
            </template>
          </q-input>
        </div>

        <div class="col col-shrink">
          <q-btn
              @click="addNewTweet"
              class="q-md-lg"
              :disable="!newTweetContent"
              unelevated
              no-caps
              rounded
              color="primary"
              label="Tweet"/>
        </div>
      </div>
      <q-separator
          class="divider"
          size="10px"
          color="grey-2"
      ></q-separator>


      <q-list separator>
        <transition-group
            appear
            enter-active-class="animated fadeIn slow"
            leave-active-class="animated fadeOut"
        >
          <q-item
              v-for="tweet in tweets"
              :key="tweet.id"
              class="q-py-md tweet"
          >
            <q-item-section avatar top>
              <q-avatar size="xl">
                <img src="https://cdn.quasar.dev/img/avatar5.jpg">
              </q-avatar>
            </q-item-section>

            <q-item-section>
              <q-item-label class="text-subtitle1">
                <strong>Kai Kåsa</strong>
                <span class="text-grey-7">
                            @kai.kåsa
                            <br class="lt-md">&bull; {{ relativeDate(tweet.date) }}
                        </span>
              </q-item-label>
              <q-item-label class="tweet-content text-body1">
                {{ tweet.content }}
              </q-item-label>

              <div class="tweet-icons row justify-between q-mt-sm">
                <q-btn
                    flat
                    round
                    color="grey"
                    icon="far fa-comment"
                    size="sm"
                />
                <q-btn
                    flat
                    round
                    color="grey"
                    icon="fas fa-retweet"
                    size="sm"
                />
                <q-btn
                    flat
                    round
                    :color="tweet.liked ? 'red' : 'grey'"
                    :icon="tweet.liked ? 'fas fa-heart' : 'far fa-heart'"
                    size="sm"
                    @click="likeTweet(tweet)"
                />
                <q-btn
                    @click="deleteTweet(tweet)"
                    flat
                    round
                    color="grey"
                    icon="fas fa-trash"
                    size="sm"
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
import {defineComponent} from 'vue'
import {formatDistance} from 'date-fns'
import db from 'src/boot/firebase'

export default defineComponent({
  name: 'Home',
  data() {
    return {
      newTweetContent: '',
      tweets: []
    }
  },
  mounted() {
    db.collection("tweets").orderBy("date").onSnapshot((snapshot) => {
      snapshot.docChanges().forEach((change) => {
        let tweetsChange = change.doc.data()
        tweetsChange.id = change.doc.id
        if (change.type === "added") {
          this.tweets.unshift(tweetsChange)

        }
        if (change.type === "modified") {
          let index = this.tweets.findIndex(tweet => tweet.id === tweetsChange.id)
          Object.assign(this.tweets[index], tweetsChange)
        }
        if (change.type === "removed") {
          let index = this.tweets.findIndex(tweet => tweet.id === tweetsChange.id)
          this.tweets.splice(index, 1)
        }
      });
    });

  },
  methods: {
    addNewTweet() {
      let newTweet = {
        content: this.newTweetContent,
        date: Date.now(),
        liked: false
      }

      db.collection("tweets").add(newTweet)
      this.newTweetContent = ''
    },
    deleteTweet(tweet) {
      db.collection("tweets").doc(tweet.id).delete()
    },
    likeTweet(tweet) {
      db.collection("tweets").doc(tweet.id).update({
        liked: !tweet.liked
      })
    },
    relativeDate(value) {
      return formatDistance(value, new Date())
    }
  }
})
</script>

<style lang="sass">
.new-tweet
  textarea
    font-size: 19px
    line-height: 1.4 !important

.divider
  border-top: 1px solid
  border-bottom: 1px solid
  border-color: $grey-4

.tweet-content
  white-space: pre-line

.tweet-icons
  margin-left: -5px

.tweet:not(:first-child)
  border-top: 1px solid rgba(0, 0, 0, 0.12)
</style>
