<template>

  <div>

    <h1>{{ $tr('myClasses') }}</h1>
    <p>{{ $tr('pageDescription') }}</p>

    <div class="table-wrapper" v-if="!noClassesExist">
      <table class="main-table">
        <caption class="visuallyhidden">{{ $tr('tableCaption') }}</caption>
        <thead>
          <tr>
            <th scope="col" class="table-text">{{ $tr('className') }}</th>
            <th scope="col" class="table-data">{{ $tr('learners') }}</th>
            <th scope="col" class="table-data">{{ $tr('groups') }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="cl in classes">
            <th scope="row" class="table-text">
              <router-link :to="recentPageLink(cl.id)">
                {{ cl.name }}
              </router-link>
            </th>
            <td class="table-data">{{ cl.learner_count }}</td>
            <td class="table-data">{{ cl.admin_count }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <p v-else>{{ $tr('noClassesExist') }}</p>

  </div>

</template>


<script>

  const constants = require('../../constants');

  module.exports = {
    data: () => ({
      currentClassDelete: null,
    }),
    computed: {
      noClassesExist() {
        return this.classes ? this.classes.length === 0 : false;
      },
    },
    methods: {
      recentPageLink(id) {
        return {
          name: constants.PageNames.RECENT,
          params: { class_id: id },
        };
      },
    },
    vuex: {
      getters: {
        classes: state => state.pageState.classes,
      },
    },
    $trNameSpace: 'coachClassListPage',
    $trs: {
      myClasses: 'My classes',
      pageDescription: 'View learner progress and performance',
      className: 'Class name',
      tableCaption: 'List of classes',
      learners: 'Learners',
      groups: 'Groups',
      noClassesExist: 'No Classes Exist.',
    },
  };

</script>


<style lang="stylus" scoped>

  @require '~kolibri.styles.definitions'

  .main-table
    width: 100%
    border-spacing: 8px
    border-collapse: separate

  thead th
    color: $core-text-annotation
    font-size: smaller
    font-weight: normal

  .table-text
    text-align: left

  .table-data
    text-align: center

  .table-wrapper
    overflow-x: auto

</style>
