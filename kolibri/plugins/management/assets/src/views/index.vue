<template>

  <core-base :topLevelPageName="topLevelPageName" :appBarTitle="$tr('managementTitle')">

    <div v-if="isAdminOrSuperuser" slot="content">
      <div class="manage-content">
        <top-nav/>
      </div>
      <component class="manage-content page" :is="currentPage"/>
    </div>

    <div v-else slot="content" class="login-message">
      <h1>{{ $tr('logInPrompt') }}</h1>
      <p>{{ $tr('logInCommand') }}</p>
    </div>

  </core-base>

</template>


<script>

  const store = require('../state/store');
  const PageNames = require('../constants').PageNames;
  const isAdminOrSuperuser = require('kolibri.coreVue.vuex.getters').isAdminOrSuperuser;
  const TopLevelPageNames = require('kolibri.coreVue.vuex.constants').TopLevelPageNames;

  module.exports = {
    $trNameSpace: 'managementRoot',
    $trs: {
      managementTitle: 'Management',
      logInPrompt: 'Did you forget to log in?',
      logInCommand: 'You must be logged in as an Admin to view this page.',
    },
    components: {
      'top-nav': require('./top-nav'),
      'manage-class-page': require('./manage-class-page'),
      'class-edit-page': require('./class-edit-page'),
      'class-enroll-page': require('./class-enroll-page'),
      'user-page': require('./user-page'),
      'data-page': require('./data-page'),
      'manage-content-page': require('./manage-content-page'),
      'scratchpad-page': require('./scratchpad-page'),
      'core-base': require('kolibri.coreVue.components.coreBase'),
    },
    computed: {
      topLevelPageName: () => TopLevelPageNames.MANAGE,
      currentPage() {
        if (this.pageName === PageNames.CLASS_MGMT_PAGE) {
          return 'manage-class-page';
        }
        if (this.pageName === PageNames.CLASS_EDIT_MGMT_PAGE) {
          return 'class-edit-page';
        }
        if (this.pageName === PageNames.CLASS_ENROLL_MGMT_PAGE) {
          return 'class-enroll-page';
        }
        if (this.pageName === PageNames.USER_MGMT_PAGE) {
          return 'user-page';
        }
        if (this.pageName === PageNames.DATA_EXPORT_PAGE) {
          return 'data-page';
        }
        if (this.pageName === PageNames.CONTENT_MGMT_PAGE) {
          return 'manage-content-page';
        }
        if (this.pageName === PageNames.SCRATCHPAD) {
          return 'scratchpad-page';
        }
        return null;
      },
    },
    vuex: {
      getters: {
        pageName: state => state.pageName,
        isAdminOrSuperuser,
      },
    },
    store, // make this and all child components aware of the store
  };

</script>


<style lang="stylus" scoped>

  @require '~kolibri.styles.definitions'

  .manage-content
    width: 100%
    @media screen and (max-width: $medium-breakpoint)
        width: 90%
        margin-left: auto
        margin-right: auto

  .page
    padding: 1em 2em
    padding-bottom: 3em
    background-color: $core-bg-light
    margin-top: 1em
    border-radius: $radius

  .login-message
    text-align: center
    margin-top: 200px

</style>
