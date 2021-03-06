<template>
  <fragment>
    <v-form>
      <v-container>
        <v-row
          align="center"
          justify="center"
          class="mt-4"
        >
          <v-col
            sm="8"
          >
            <div
              v-if="!isOwner"
              data-test="notTheOwner"
              style="text-align:center"
            >
              <h3
                class="pl-6"
              >
                You're not the owner of this namespace.
              </h3>
              <br>
            </div>

            <div
              class="mt-6 pl-4 pr-4"
            >
              <v-row
                class="mb-4"
              >
                <v-col
                  md="auto"
                >
                  <v-card
                    tile
                    :elevation="0"
                  >
                    Tenant ID:
                  </v-card>
                </v-col>
                <v-col
                  md="auto"
                  class="ml-auto"
                >
                  <v-card
                    class="auto"
                    tile
                    :elevation="0"
                  >
                    <v-chip>
                      <span
                        data-test="tenant"
                      >
                        {{ tenant }}
                      </span>
                      <v-icon
                        v-clipboard="tenant"
                        v-clipboard:success="() => {
                          this.$store.dispatch('snackbar/showSnackbarCopy', this.$copy.tenantId);
                        }"
                        right
                      >
                        mdi-content-copy
                      </v-icon>
                    </v-chip>
                  </v-card>
                </v-col>
              </v-row>
              <v-divider />
              <v-divider />
            </div>

            <div
              v-if="isOwner"
              class="mt-6 mb-6 pl-4 pr-4"
              data-test="editOperation"
            >
              <h3>
                Edit namespace
              </h3>

              <ValidationObserver
                ref="data"
                v-slot="{ passes }"
              >
                <div
                  class="mt-4 mb-2"
                >
                  <ValidationProvider
                    v-slot="{ errors }"
                    ref="providerName"
                    vid="name"
                    name="Priority"
                    rules="required|rfc1123"
                  >
                    <v-row>
                      <v-col
                        class="ml-3"
                      >
                        <v-text-field
                          v-model="name"
                          label="Name"
                          :error-messages="errors"
                          required
                          data-test="name-text"
                        />
                      </v-col>
                      <v-col
                        md="auto"
                        class="ml-auto"
                      >
                        <v-btn
                          outlined
                          @click="passes(editNamespace)"
                        >
                          Rename Namespace
                        </v-btn>
                      </v-col>
                    </v-row>
                  </ValidationProvider>
                </div>
              </ValidationObserver>
              <v-divider />
              <v-divider />
            </div>

            <div
              v-if="isHostedOwner"
              class="mt-6 mb-6 pl-4 pr-4"
              data-test="userOperation"
            >
              <v-row>
                <v-col>
                  <h3>
                    Members
                  </h3>
                </v-col>

                <v-spacer />

                <v-col
                  md="auto"
                  class="ml-auto"
                >
                  <NamespaceNewMember :ns-tenant="tenant" />
                </v-col>
              </v-row>

              <div>
                <v-list>
                  <v-list-item
                    v-for="item in namespace.members"
                    :key="item.id"
                    class="mt-5 mb-2"
                  >
                    <v-row>
                      <v-col
                        md="auto"
                        class="ml-auto"
                      >
                        <v-icon>
                          mdi-account
                        </v-icon>
                      </v-col>

                      <v-col>
                        <v-list-item-title
                          :data-test="item.name"
                        >
                          {{ item.name }}
                        </v-list-item-title>
                      </v-col>

                      <v-spacer />

                      <v-col
                        md="auto"
                        class="ml-auto"
                      >
                        <v-btn
                          data-test="remove-member"
                          v-if="item.id!==owner"
                          outlined
                          @click="remove(item.name)"
                        >
                          <v-tooltip
                            bottom
                          >
                            <template #activator="{ on }">
                              <v-icon
                                v-on="on"
                              >
                                delete
                              </v-icon>
                            </template>
                            <span>
                              Remove user
                            </span>
                          </v-tooltip>
                        </v-btn>
                        <p
                          data-test="owner"
                          class="mr-3"
                          v-else
                        >
                          Owner
                        </p>
                      </v-col>
                    </v-row>
                  </v-list-item>
                </v-list>
              </div>
              <v-divider />
              <v-divider />
            </div>

            <div
              v-if="isOwner"
              class="mt-6 mb-6 pl-4 pr-4"
              data-test="deleteOperation"
            >
              <h3
                class="mb-5"
              >
                Danger Zone
              </h3>

              <v-row
                class="mt-4 mb-2"
              >
                <v-col
                  class="ml-3"
                >
                  Delete this namespace
                </v-col>
                <v-col
                  md="auto"
                  class="ml-auto mb-4"
                >
                  <NamespaceDelete :ns-tenant="tenant" />
                </v-col>
              </v-row>
              <v-divider />
              <v-divider />
            </div>

            <div
              v-if="isHostedOwner"
              data-test="securityOperation"
              class="mt-6 mb-6 pl-4 pr-4"
            >
              <SettingSecurity />
            </div>
          </v-col>
        </v-row>
      </v-container>
    </v-form>
  </fragment>
</template>

<script>

import {
  ValidationObserver,
  ValidationProvider,
} from 'vee-validate';

import SettingSecurity from '@/components/setting/SettingSecurity';
import NamespaceNewMember from '@/components/app_bar/namespace/NamespaceNewMember';
import NamespaceDelete from '@/components/app_bar/namespace/NamespaceDelete';

export default {
  name: 'SettingNamespace',

  components: {
    ValidationProvider,
    ValidationObserver,
    NamespaceNewMember,
    NamespaceDelete,
    SettingSecurity,
  },

  data() {
    return {
      name: '',
    };
  },

  computed: {
    isOwner() {
      return this.owner === this.$store.getters['auth/id'];
    },

    owner() {
      return this.$store.getters['namespaces/get'].owner;
    },

    namespace() {
      return this.$store.getters['namespaces/get'];
    },

    tenant() {
      return localStorage.getItem('tenant');
    },

    isHostedOwner() {
      return this.$env.isHosted && this.isOwner;
    },
  },

  async created() {
    await this.getNamespace();
    this.name = this.namespace.name;
  },

  methods: {
    async editNamespace() {
      try {
        await this.$store.dispatch('namespaces/put', { id: this.tenant, name: this.name });
        await this.$store.dispatch('namespaces/get', this.tenant);
        this.$store.dispatch('snackbar/showSnackbarSuccessAction', this.$success.namespaceEdit);
      } catch {
        this.$store.dispatch('snackbar/showSnackbarErrorAction', this.$errors.namespaceEdit);
      }
    },

    async getNamespace() {
      try {
        await this.$store.dispatch('namespaces/get', this.tenant);
      } catch (e) {
        if (e.response.status === 403) {
          this.$store.dispatch('snackbar/showSnackbarErrorAssociation');
        } else {
          this.$store.dispatch('snackbar/showSnackbarErrorAction', this.$errors.namespaceLoad);
        }
      }
    },

    async remove(username) {
      try {
        await this.$store.dispatch('namespaces/removeUser', {
          username,
          tenant_id: this.tenant,
        });
        this.dialog = false;
        this.username = '';
        this.$store.dispatch('snackbar/showSnackbarSuccessAction', this.$success.namespaceRemoveUser);
      } catch {
        this.$store.dispatch('snackbar/showSnackbarErrorAction', this.$errors.namespaceRemoveUser);
      }
    },
  },
};

</script>
