<template>

  <div>
    <!--USER BREADCRUMBS-->
    <breadcrumbs :list="userBreadcrumbs"/>

    <!--TABS-->
    <all-recent-tabs
      :recentViewLink="recentViewLink"
      :allViewLink="allViewLink"
      :isRecentView="isRecentView"/>


    <div class="tabcontents">
      <div class="top-section">
        <!--CONTENT BREADCRUMBS-->
        <breadcrumbs
          v-if="!isRecentView && contentBreadcrumbs.length > 1"
          :list="contentBreadcrumbs"
        />

        <!--HEADER SECTION-->
        <report-header
          :contentKind="pageState.content_scope_summary.kind"
          :contentTitle="pageState.content_scope_summary.title"
          :userFullName="pageState.user_scope_summary.full_name"/>

        <!--SUMMARY SECTION-->
        <summary-section
          :kind="pageState.content_scope_summary.kind"
          :exerciseCount="exerciseCount"
          :exerciseProgress="exerciseProgress"
          :contentCount="contentCount"
          :contentProgress="contentProgress"
          :lastActive="pageState.content_scope_summary.last_active"
          :singleUser="isSingleUser"
          :userCount="userCount"
          :completionCount="completionCount"
          :isRecentView="isRecentView"/>
      </div>

      <!-- TABLE SECTION -->
      <div v-if="!isSingleUser || !isSingleItem" class="table-section">

      <!--VIEW-BY SWITCH-->
        <view-by-switch
          v-if="!isRecentView"
          :isContent="isViewByContent"
          :vlink="viewByLink"
          :disabled="isSingleUser || isSingleItem"/>

        <!--TABLE SECTION-->
        <table class="data-table">
          <thead>
            <tr>
              <th is="header-cell"
                :text="$tr('name')"
                :column="ReportConstants.TableColumns.NAME"
                class="name-col coach-filter table-name"
              ></th>
              <th is="header-cell"
                :text="$tr('avg-exercise-progress')"
                :column="ReportConstants.TableColumns.EXERCISE"
                class="progress-col coach-filter"
              ></th>
              <th is="header-cell"
                :text="$tr('avg-content-progress')"
                :column="ReportConstants.TableColumns.CONTENT"
                class="progress-col coach-filter"
              ></th>
              <th is="header-cell"
                v-if="!isRecentView"
                :text="$tr('last-activity')"
                :column="ReportConstants.TableColumns.DATE"
                class="date-col coach-filter"
              ></th>
            </tr>
          </thead>
          <tbody is="transition-group" name="item">
            <tr v-for="row in dataTable" :key="row.id">
              <th scope="row" class="name-col">
                <item-cell
                  :kind="row.kind"
                  :title="row.title"
                  :id="row.id"
                  :parent="row.parent"
                  :exerciseCount="row.exerciseCount"
                  :contentCount="row.contentCount"
                />
              </th>
              <td class="progress-col">
                <progress-cell :num="row.exerciseProgress" :isExercise="true"/>
              </td>
              <td class="progress-col">
                <progress-cell :num="row.contentProgress" :isExercise="false"/>
              </td>
              <td class="date-col" v-if="!isRecentView">
                <date-cell :date="row.lastActive"/>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

    </div>
  </div>

</template>


<script>

  const ReportConstants = require('../../reportConstants');
  const getters = require('../../state/getters');
  const coreGetters = require('kolibri.coreVue.vuex.getters');
  const genLink = require('./genLink');
  const logging = require('kolibri.lib.logging');

  module.exports = {
    $trNameSpace: 'reportPage',
    $trs: {
      'name': 'Name',
      'avg-exercise-progress': 'Avg. exercise progress',
      'avg-content-progress': 'Avg. resource progress',
      'last-activity': 'Last activity',
      'all-learners': 'All learners ({0, number, integer})',
    },
    components: {
      'breadcrumbs': require('./breadcrumbs'),
      'all-recent-tabs': require('./all-recent-tabs'),
      'report-header': require('./report-header'),
      'summary-section': require('./summary-section'),
      'view-by-switch': require('./view-by-switch'),
      'header-cell': require('./header-cell'),
      'user-cell': require('./data-cells/user-cell'),
      'date-cell': require('./data-cells/date-cell'),
      'progress-cell': require('./data-cells/progress-cell'),
      'item-cell': require('./data-cells/item-cell'),
    },
    computed: {
      ReportConstants() {
        return ReportConstants; // allow constants to be accessed inside templates
      },
      isViewByContent() {
        return this.pageState.view_by_content_or_learners === ReportConstants.ViewBy.CONTENT;
      },
      isRecentView() {
        return this.pageState.all_or_recent === ReportConstants.AllOrRecent.RECENT;
      },
      isSingleUser() {
        return this.pageState.user_scope === ReportConstants.UserScopes.USER;
      },
      isSingleItem() {
        return this.pageState.content_scope === ReportConstants.ContentScopes.CONTENT;
      },
      userBreadcrumbs() {
        if (this.pageState.user_scope === ReportConstants.UserScopes.FACILITY) {
          return [{ title: this.$tr('all-learners', [this.userCount]) }];
        } else if (this.pageState.user_scope === ReportConstants.UserScopes.USER) {
          const FACILITY_ID = '1'; // TODO - facility ID should not be hard-coded.
          return [
            {
              title: 'All Learners',
              vlink: genLink(this.pageState, {
                view_by_content_or_learners:
                  this.isRecentView ? ReportConstants.ViewBy.CONTENT : ReportConstants.ViewBy.LEARNERS,
                user_scope: ReportConstants.UserScopes.FACILITY,
                user_scope_id: FACILITY_ID,
              }),
            },
            { title: this.pageState.user_scope_summary.full_name },
          ];
        }
        logging.error(`Unhandled user_scope: '${this.pageState.user_scope}'`);
        return [];
      },
      contentBreadcrumbs() {
        const list = this.pageState.content_scope_summary.ancestors.map((item, index) => ({
          title: item.title,
          vlink: genLink(this.pageState, {
            view_by_content_or_learners: ReportConstants.ViewBy.CONTENT,
            content_scope: index ? ReportConstants.ContentScopes.TOPIC : ReportConstants.ContentScopes.ROOT,
            content_scope_id: item.pk,
          }),
        }));
        list.push({ title: this.pageState.content_scope_summary.title });
        return list;
      },
      recentViewLink() {
        return genLink(this.pageState, {
          all_or_recent: ReportConstants.AllOrRecent.RECENT,
          content_scope: ReportConstants.ContentScopes.ROOT, // recent view only applies to root
          content_scope_id: this.currentChannel.root_id,
          view_by_content_or_learners: ReportConstants.ViewBy.CONTENT,
        });
      },
      allViewLink() {
        return genLink(this.pageState, { all_or_recent: ReportConstants.AllOrRecent.ALL });
      },
      viewByLink() {
        // target of the link is the opposite of the current view
        const view = this.isViewByContent ? ReportConstants.ViewBy.LEARNERS : ReportConstants.ViewBy.CONTENT;
        return genLink(this.pageState, { view_by_content_or_learners: view });
      },
    },
    vuex: {
      getters: {
        pageState: state => state.pageState,
        exerciseCount: getters.exerciseCount,
        exerciseProgress: getters.exerciseProgress,
        contentCount: getters.contentCount,
        contentProgress: getters.contentProgress,
        dataTable: getters.dataTable,
        userCount: getters.userCount,
        completionCount: getters.completionCount,
        currentChannel: coreGetters.getCurrentChannelObject,
      },
    },
  };

</script>


<style lang="stylus" scoped>

  @require '~kolibri.styles.definitions'
  @require './reports.styl'

  .data-table
    width: 100%
    font-size: smaller
    border-spacing: 0

    td, th
      padding: $col-padding
      text-align: left

    .table-name
      text-align: left

    .name-col
      text-align: left

    .progress-col
      text-align: center
      width: $progress-col-width

    .date-col
      text-align: left
      width: $date-col-width

  .item-move
    transition: transform 0.5s cubic-bezier(0.55, 0, 0.1, 1)

  .top-section,
  .table-section
    background-color: $core-bg-light
    margin-top: 1em
    margin-bottom: 1em
    padding: 1em

</style>
