<template>

  <exercise-detail-view :backPageLink="backPageLink">
    <template slot="text"> {{ backtoText(contentName) }} </template>
    <template slot="body">
      <div class="page-status-container">
        <page-status
          :contentName="contentName"
          :userName="userName"
          :progress="progress"
          :assessment="assessment"
          :date="date"/>
      </div>
      <div class="outer-container">
        <div class="answer-history-container column">
          <answer-history
            :questionHistory="questionHistory"
          />
        </div>
        <div class="exercise-container column">
          <question-attempt
            :questionNumber="pageState.selectedAttemptLogIndex + 1"/>
          <content-renderer
            class="content-renderer"
            :id="content.id"
            :kind="content.kind"
            :files="content.files"
            :contentId="content.content_id"
            :channelId="channelId"
            :available="content.available"
            :extraFields="content.extra_fields"/>
        </div>
      </div>
    </template>
  </exercise-detail-view>

</template>


<script>

  const constants = require('../../constants');

  module.exports = {
    $trNameSpace: 'coachExerciseRenderPage',
    $trs: {
      backto: 'Back to { text }',
    },
    components: {
      'exercise-detail-view': require('./../exercise-detail-view'),
      'content-renderer': require('kolibri.coreVue.components.contentRenderer'),
      'page-status': require('./page-status'),
      'answer-history': require('./answer-history'),
      'question-attempt': require('./question-attempt'),
    },
    computed: {
      backPageLink() {
        return { name: constants.PageNames.CLASS_LIST };
      },
      content() {
        return {
          id: '84658d43b99f5824bc1aa5e3eb6b3578',
          kind: 'exercise',
          files: [{
            extension: 'perseus',
            download_url: '/downloadcontent/898fa0875f5cdf1721a32eb7540d0ec8.perseus/Divide_fractions_and_whole_numbers_word_problems_Exercise.perseus',
            available: true,
            checksum: '898fa0875f5cdf1721a32eb7540d0ec8',
            file_size: 182937,
            id: '47e59275f0f64c89aba65a537aeb38c2',
            lang: null,
            preset: 'Exercise',
            priority: null,
            storage_url: '/zipcontent/898fa0875f5cdf1721a32eb7540d0ec8.perseus/',
            supplementary: false,
            thumbnail: false,
          }],
          content_id: '357f3d15348c4e3d8ac5d459ad8b924d',
          available: true,
          extraFields: null,
        };
      },
      channelId() {
        return '78eed5c0b59b30c0a40c94c17c849af6';
      },
    },
    methods: {
      backtoText(text) {
        return this.$tr('backto', { text });
      },
    },
    vuex: {
      getters: {
        pageState: state => state.pageState,
        // fake date for page-status
        contentName: () => 'Adding Fractions',
        userName: () => 'James Howard',
        progress: () => 1,
        assessment: () => '4 of 5',
        date: () => '18 Nov 2016',
        // fake date for answer-history
        questionHistory: () => [1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1],
        // fake date for question-attempts
      },
    },
  };

</script>


<style lang="stylus" scoped>

  .column
    float: left

  .page-status-container
    padding-top: 20px
    padding-left: 10px
    padding-right: 10px

  .outer-container
    display: table-cell
    height: 100%
    width: 1%
    padding: 10px

  .answer-history-container
    width: 30%
    height: 100%
    overflow-y: auto

  .exercise-container
    width: 70%

</style>
