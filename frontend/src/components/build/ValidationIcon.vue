<template>
    <span>
        <button @click="toggle" class="btn btn-link text-primary" :class="{'bg-light':show_details}">
            <i v-if="status == 'ok'" class="far fa-check-circle"></i>
            <i v-if="status == 'sync'" class="fas fa-sync"></i>
            <i v-if="status == 'warn'" class="fas fa-exclamation-triangle text-warning"></i>
        </button>

        <div v-if="show_details"  v-click-outside="hide_details" class="validation-warns">
            <div class="card">
                <div class="card-body">
                    <div style="max-height: 300px; overflow-y: scroll;">
                        <table class="table table-sm">
                            <tr v-for="warn in warnings">
                                <th>
                                    <router-link :to="warn_router_link(warn)" href="#">
                                        {{ warn.id }}
                                    </router-link>
                                </th>
                                <td class="text-danger">{{ warn.message }}</td>
                            </tr>
                            <tr v-if="warnings.length == 0"><td>No problems detected.</td></tr>
                        </table>
                        
                    </div>

                </div>
            </div>
        </div>
    </span>
</template>

<script>

import { store } from '../../store'
import { validate } from '../../validation'
import _ from 'lodash'

export default {
    name: 'validation-icon',
    data() {
        return {
            status: 'ok',
            show_details: false,
            warnings: [],
            project: store.project,
        }
    },
    created() {
        this.run_validate(this)
    },
    watch: {
        project: {
            deep: true,
            handler() {
                this.status = 'sync'
                this.run_validate(this)
            }
        }
    },
    methods: {
        toggle() {
            this.show_details = !this.show_details
        },
        hide_details() {
            this.show_details = false
        },
        warn_router_link(warn) {
            let items = warn.id.split('.')
            return {name: 'model-field', params: {app: items[0], model:items[1], field:items[2]}}
        },
        run_validate: _.debounce((component)=>{
            component.warnings = validate(component.project)
            component.status = (component.warnings.length == 0) ? 'ok' : 'warn'
        }, 500)
    }
}

</script>

<style scoped>
    .validation-warns {
        position: fixed;
        top: 4rem;
        right: 1rem;
        z-index: 1;
        min-width: 300px;
    }
</style>
