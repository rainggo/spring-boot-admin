<!--
  - Copyright 2014-2018 the original author or authors.
  -
  - Licensed under the Apache License, Version 2.0 (the "License");
  - you may not use this file except in compliance with the License.
  - You may obtain a copy of the License at
  -
  -     http://www.apache.org/licenses/LICENSE-2.0
  -
  - Unless required by applicable law or agreed to in writing, software
  - distributed under the License is distributed on an "AS IS" BASIS,
  - WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  - See the License for the specific language governing permissions and
  - limitations under the License.
  -->

<template>
  <section class="section">
    <div class="container">
      <div v-if="error" class="message is-warning">
        <div class="message-body">
          <strong>
            <font-awesome-icon class="has-text-warning" icon="exclamation-triangle"/>
            Server connection failed.
          </strong>
          <p v-text="error.message"/>
        </div>
      </div>
      <div class="level applications-stats">
        <div class="level-item has-text-centered">
          <div>
            <p class="heading">Applications</p>
            <p class="title" v-text="applicationsCount">1</p>
          </div>
        </div>
        <div class="level-item has-text-centered">
          <div>
            <p class="heading">Instances</p>
            <p class="title" v-text="instancesCount">1</p>
          </div>
        </div>
        <div class="level-item has-text-centered">
          <div v-if="downCount === 0">
            <p class="heading">Status</p>
            <p class="title has-text-success">all up</p>
          </div>
          <div v-else>
            <p class="heading">instances down</p>
            <p class="title has-text-danger" v-text="downCount"/>
          </div>
        </div>
      </div>
      <div class="application-group" v-for="group in statusGroups" :key="group.status">
        <p class="heading" v-text="group.status"/>
        <applications-list :applications="group.applications" :selected="selected"/>
      </div>
      <div v-if="statusGroups.length === 0">
        <p class="is-muted">No applications registered.</p>
      </div>
    </div>
  </section>
</template>

<script>
  import * as _ from 'lodash';
  import applicationsList from './applications-list';
  import label from './label';

  export default {
    props: {
      applications: {
        type: Array,
        default: () => [],
      },
      error: {
        type: null,
        default: null
      },
      selected: {
        type: String,
        default: null
      }
    },
    components: {
      applicationsList,
    },
    computed: {
      statusGroups() {
        const byStatus = _.groupBy(this.applications, application => application.status);
        const list = _.transform(byStatus, (result, value, key) => {
          result.push({status: key, applications: _.sortBy(value, [application => application.name])})
        }, []);
        return _.sortBy(list, [item => item.status]);
      },
      applicationsCount() {
        return this.applications.length;
      },
      instancesCount() {
        return this.applications.reduce((current, next) => current + next.instances.length, 0);
      },
      downCount() {
        return this.applications.reduce((current, next) => {
          return current + (next.instances.filter(instance => instance.statusInfo.status !== 'UP').length);
        }, 0);
      }
    },
    install({viewRegistry}) {
      viewRegistry.addView({
        path: '/applications/:selected?',
        props: true,
        name: 'applications',
        label,
        order: 0,
        component: this
      });
      viewRegistry.addRedirect('/', 'applications');
    }
  };
</script>


<style lang="scss">
  @import "~@/assets/css/utilities";

  .application-group {
    margin: $gap 0;
  }

</style>
