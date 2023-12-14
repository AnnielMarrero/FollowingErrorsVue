<template>
  <div class="q-pa-md">
    <div class="row q-mb-md q-gutter-xl">
      <q-select
        class="col"
        option-value="id"
        option-label="name"
        emit-value
        map-options
        v-model="userSelected"
        :options="users"
        label="Users"
        @update:model-value="getBugs"
      />
      <q-select
        class="col"
        option-value="id"
        option-label="name"
        emit-value
        map-options
        v-model="projectSelected"
        :options="projects"
        label="Projects"
        @update:model-value="getBugs"
      />
      <q-input class="col" v-model="startDate" label="Start Date">
        <template v-slot:prepend>
          <q-icon name="event" class="cursor-pointer">
            <q-popup-proxy
              cover
              transition-show="scale"
              transition-hide="scale"
            >
              <q-date v-model="startDate" :mask="dateTimeFormat">
                <div class="row items-center justify-end">
                  <q-btn
                    no-caps
                    v-close-popup
                    label="Confirm"
                    color="primary"
                    flat
                    @click="getBugs"
                  />
                </div>
              </q-date>
            </q-popup-proxy>
          </q-icon>
        </template>

        <template v-slot:append>
          <q-icon name="access_time" class="cursor-pointer">
            <q-popup-proxy
              cover
              transition-show="scale"
              transition-hide="scale"
            >
              <q-time v-model="startDate" :mask="dateTimeFormat" format24h>
                <div class="row items-center justify-end">
                  <q-btn
                    no-caps
                    v-close-popup
                    label="Confirm"
                    color="primary"
                    flat
                    @click="getBugs"
                  />
                </div>
              </q-time>
            </q-popup-proxy>
          </q-icon>
        </template>
      </q-input>

      <q-input
        class="col"
        v-model="endDate"
        label="End Date"
        @update:model-value="getBugs"
      >
        <template v-slot:prepend>
          <q-icon name="event" class="cursor-pointer">
            <q-popup-proxy
              cover
              transition-show="scale"
              transition-hide="scale"
            >
              <q-date v-model="endDate" :mask="dateTimeFormat">
                <div class="row items-center justify-end">
                  <q-btn
                    no-caps
                    v-close-popup
                    label="Confirm"
                    color="primary"
                    flat
                    @click="getBugs"
                  />
                </div>
              </q-date>
            </q-popup-proxy>
          </q-icon>
        </template>

        <template v-slot:append>
          <q-icon name="access_time" class="cursor-pointer">
            <q-popup-proxy
              cover
              transition-show="scale"
              transition-hide="scale"
            >
              <q-time v-model="endDate" :mask="dateTimeFormat" format24h>
                <div class="row items-center justify-end">
                  <q-btn
                    no-caps
                    v-close-popup
                    label="Confirm"
                    color="primary"
                    flat
                    @click="getBugs"
                  />
                </div>
              </q-time>
            </q-popup-proxy>
          </q-icon>
        </template>
      </q-input>
    </div>

    <q-table
      title="Bugs"
      :rows="bugs"
      :columns="columns"
      row-key="id"
      :rows-per-page-options="[0]"
      :loading="loading"
    >
      <template v-slot:top>
        <q-btn
          color="primary"
          :disable="loading"
          label="Add bug"
          @click="showAddDialog = true"
        />
      </template>
    </q-table>

    <q-dialog v-model="showAddDialog" persistent>
      <q-card style="width: 600px">
        <q-card-section class="text-center">
          <span>Add new bug</span>
        </q-card-section>
        <q-form class="q-ma-md" @submit="addBug">
          <div class="row q-gutter-md">
            <div class="col">
              <q-select
                class="col"
                option-value="id"
                option-label="name"
                emit-value
                map-options
                v-model="newBug.userId"
                :options="users"
                label="Users*"
                :rules="[(val) => !!val || 'Required field.']"
              />
            </div>
            <div class="col">
              <q-select
                class="col"
                option-value="id"
                option-label="name"
                emit-value
                map-options
                v-model="newBug.projectId"
                :options="projects"
                label="Projects*"
                :rules="[(val) => !!val || 'Required field.']"
              />
            </div>
          </div>
          <div class="row">
            <q-input
              class="fit"
              v-model="newBug.description"
              filled
              type="textarea"
              :rules="[(val) => !!val && val.length <= 100 || 'Required field with maximun length 100 characteres.']"
              label="Description*"
            />
          </div>

          <q-card-actions align="center">
            <q-btn no-caps label="Cancel" color="negative" v-close-popup />
            <q-btn no-caps label="Add" color="positive" type="submit" />
          </q-card-actions>
        </q-form>
      </q-card>
    </q-dialog>
  </div>
</template>

<script setup lang="ts">
import { date } from 'quasar';
import { Notify } from 'quasar';
import { api } from 'src/boot/axios';
import { onMounted, ref } from 'vue';

const columns = [
  { name: 'userName', align: 'center', label: 'User Name', field: 'userName' },
  { name: 'project', label: 'Project Name', field: 'project', align: 'center' },
  { name: 'description', label: 'Description', field: 'description' },
  {
    name: 'creationDate',
    label: 'Creation Date',
    field: 'creationDate',
    format: (val) => `${date.formatDate(val, dateTimeFormat)}`,
  },
];

interface BugDto {
  id: number;
  description: string;
  userName: string;
  project: string;
  projectId: number;
  userId: number;
  creationDate: string;
}

interface ComboBox {
  id: number;
  name: string;
}

const userSelected = ref<number | null>(null);
const projectSelected = ref<number | null>(null);
const startDate = ref(null);
const endDate = ref(null);
const dateTimeFormat = 'YYYY-MM-DD HH:mm';
const showAddDialog = ref(false);

const loading = ref(false);
const users = ref<ComboBox[]>();
const projects = ref<ComboBox[]>();

const bugs = ref<BugDto[]>();

const newBug = ref<BugDto>({} as BugDto);
const getProjects = async () => {
  try {
    const resp = await api.get('projects');
    projects.value = resp.data;
  } catch (err: unknown) {
    //console.log("can't load projects", err);
  }
};
const getUsers = async () => {
  try {
    const resp = await api.get('users');
    users.value = resp.data;
  } catch (err: unknown) {
    //console.log("can't load users", err);
  }
};
const getBugs = async () => {
  try {
    loading.value = true;
    const filters = [];
    if (userSelected.value) filters.push(`userId=${userSelected.value}`);
    if (projectSelected.value)
      filters.push(`projectId=${projectSelected.value}`);
    if (startDate.value) {
      filters.push(`startDate=${startDate.value}`);
    }
    if (endDate.value) {
      filters.push(`endDate=${endDate.value}`);
    }
    if (filters.length === 0) {
      showAlert('You must provide at least one filter for list the bugs');
      return;
    }
    const resp = await api.get(`bugs?${filters.join('&')}`);
    bugs.value = resp.data.bugs;
  } catch (err: unknown) {
    //console.log("any bugs found", err);
    bugs.value = [];
  } finally {
    loading.value = false;
  }
};

const addBug = async () => {
  try {
    await api.post('bugs', newBug.value);
    showAlert('New bugs added with success', true);
    showAddDialog.value = false;
    await getBugs();
  } catch (err: unknown) {
    showAlert('Some errors ocurred when trying to add the bug');
  } finally {
    //clear form
    newBug.value = {} as BugDto;
  }
};

const showAlert = (msg: string, success = false) => {
  //console.log('showAlert', msg);
  Notify.create({
    color: success ? 'positive' : 'negative',
    position: 'top',
    message: msg,
    icon: success ? 'check' : 'report_problem',
  });
};

onMounted(async () => {
  await getUsers();
  await getProjects();
  if (users.value) {
    const first: ComboBox = users.value[0];
    userSelected.value = first.id;
  }
  await getBugs();
});
</script>
