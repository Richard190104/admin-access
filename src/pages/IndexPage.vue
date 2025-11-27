<template>
  <q-page class="q-pa-md page-bg">
    <div class="q-pa-md container">
      <div class="row q-col-gutter-md">
        <div class="col-12 col-md-5 col-lg-4">
          <q-card class="rounded-borders shadow-2">
            <q-card-section>
              <div class="text-h6">Vytvoriť termíny</div>
              <div class="text-caption text-grey">
                Vyberte deň a čas, alebo len deň nechajte systém vygenerovať dostupné časy.
              </div>
            </q-card-section>

            <q-card-section>
              <q-input v-model="form.date" type="date" label="Dátum" filled />
              <q-input
                v-model="form.time"
                type="time"
                label="Čas (voliteľné)"
                filled
                class="q-mt-sm"
              />

              <q-btn
                label="Vygenerovať časy"
                color="primary"
                :disable="!form.date"
                @click="submitForm"
                class="q-mt-md"
              />
            </q-card-section>

            <q-separator />

            <q-card-section v-if="displayTimes">
              <div class="text-subtitle2 q-mb-sm">
                Dostupné časy pre {{ formatDate(form.date, false) }}
              </div>

              <q-list padding>
                <q-item v-for="time in times" :key="time" class="rounded-borders q-my-xs date-item">
                  <q-item-section avatar>
                    <q-icon name="schedule" />
                  </q-item-section>

                  <q-item-section>
                    <div class="date-main">{{ formatDate(time) }}</div>
                  </q-item-section>

                  <q-item-section side>
                    <q-checkbox
                      v-model="selectedTimes"
                      :val="time"
                      color="primary"
                      dense
                      @click.stop
                    />
                  </q-item-section>
                </q-item>
              </q-list>

              <q-btn
                label="Potvrdiť výber dátumov"
                color="primary"
                class="full-width q-mt-sm"
                @click="submitNewDates"
              />
            </q-card-section>
          </q-card>
        </div>

        <div class="col-12 col-md-7 col-lg-6">
          <q-card class="rounded-borders shadow-2">
            <q-card-section>
              <div class="text-h6">Vypísané termíny</div>
              <div class="text-caption text-grey">Prehľad všetkých vypísaných termínov</div>
            </q-card-section>

            <q-separator />

            <q-card-section>
              <q-list bordered separator>
                <q-expansion-item
                  v-for="items in allDatesGroupByDay"
                  :key="items.date"
                  v-model="openedExpansion"
                  :label="items.date"
                  icon="event"
                  group="datesAccordion"
                  switch-toggle
                  expand-separator
                  @click="selectDate(items.date)"
                  dense
                >
                  <q-list padding>
                    <q-item
                      v-for="item in availableDates"
                      :key="String(item.id)"
                      clickable
                      active-class="item-active"
                      class="rounded-borders q-my-xs date-item"
                    >
                      <q-item-section avatar>
                        <q-icon name="schedule" />
                      </q-item-section>

                      <q-item-section>
                        <div class="date-main">{{ item.date }}</div>
                      </q-item-section>
                      <q-item-section side>
                        <q-btn
                          icon="delete"
                          color="negative"
                          dense
                          flat
                          @click.stop="removeDate(item.id ?? 0)"
                        />
                      </q-item-section>
                    </q-item>
                  </q-list>
                </q-expansion-item>
              </q-list>
            </q-card-section>
          </q-card>
        </div>
      </div>
    </div>

    <div class="q-pa-md container">
      <div class="row">
        <div class="col-12">
          <q-card class="rounded-borders shadow-2 q-mt-md">
            <q-card-section>
              <div class="text-h6">Používatelia</div>
              <div class="text-caption text-grey">
                Vytvorte nového používateľa alebo spravujte existujúcich.
              </div>
            </q-card-section>

            <q-separator />

            <q-card-section>
              <div class="row items-center q-col-gutter-sm">
                <div class="col-12 col-md-4">
                  <q-input v-model="newUser.userID" label="Id" filled />
                </div>
                <div class="col-12 col-md-4">
                  <q-input v-model="newUser.email" label="Email" filled />
                </div>
                <div class="col-12 col-md-4">
                  <q-btn label="Vytvoriť používateľa" color="primary" @click="createUser" />
                </div>
              </div>

              <q-separator class="q-mt-md q-mb-md" />

              <q-list bordered separator>
                <q-item v-for="user in users" :key="user.userID" class="rounded-borders q-my-xs">
                  <q-item-section avatar>
                    <q-avatar>
                      <q-icon name="person" />
                    </q-avatar>
                  </q-item-section>

                  <q-item-section>
                    <div class="text-weight-medium text-grey">
                      ID: <b class="text-black">{{ user.userID }}</b>
                    </div>
                    <div class="text-caption text-grey">email: {{ user.email }}</div>
                  </q-item-section>
                  <q-item-section>
                    <div class="text-caption text-grey">
                      Termíny:
                      <div v-if="user.dates && user.dates.length > 0">
                        <div v-for="date in user.dates" :key="String(date)" class="text-black">
                          <div v-if="date">{{ formatDate(String(date) ?? '') }}</div>
                        </div>
                      </div>
                      <div v-else class="text-black">Žiadne termíny</div>
                    </div>
                  </q-item-section>

                  <q-item-section side>
                    <q-btn
                      icon="delete"
                      color="negative"
                      dense
                      flat
                      @click.stop="removeUser(user.userID)"
                    />
                  </q-item-section>
                </q-item>
                <q-item v-if="users.length === 0" class="no-users">
                  <q-item-section>
                    <div class="text-caption text-grey">Žiadni používatelia</div>
                  </q-item-section>
                </q-item>
              </q-list>
            </q-card-section>
          </q-card>
        </div>
      </div>
    </div>

    <q-dialog v-model="displayRemoveInfo" persistent>
      <q-card style="width: 380px; max-width: 90vw; border-radius: 14px">
        <q-card-section class="row items-center q-gutter-sm">
          <q-icon name="warning" color="red" size="md" />
          <div class="text-h6">Potvrdiť odstránenie</div>
        </q-card-section>

        <q-card-section class="text-body1"> Naozaj chcete odstrániť tento termín? </q-card-section>

        <q-card-actions align="right" class="q-gutter-sm q-pa-sm">
          <q-btn flat label="Zrušiť" color="grey-8" @click="displayRemoveInfo = false" />

          <q-btn unelevated color="red" label="Odstrániť" @click="removeConfirm()" />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <q-dialog v-model="displayRemoveUserInfo" persistent>
      <q-card style="width: 380px; max-width: 90vw; border-radius: 14px">
        <q-card-section class="row items-center q-gutter-sm">
          <q-icon name="warning" color="red" size="md" />
          <div class="text-h6">Potvrdiť odstránenie používateľa</div>
        </q-card-section>

        <q-card-section class="text-body1">
          Naozaj chcete odstrániť používateľa
          <span class="text-weight-medium"> {{ clickedUserForRemove.name }} </span>?
        </q-card-section>

        <q-card-actions align="right" class="q-gutter-sm q-pa-sm">
          <q-btn flat label="Zrušiť" color="grey-8" @click="displayRemoveUserInfo = false" />

          <q-btn unelevated color="red" label="Odstrániť" @click="removeUserConfirm()" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import type { dateSelection } from 'src/components/models';
const dayTimes = {
  1: ['09:00', '9:30', '10:00', '10:30'],
  2: ['14:00', '14:30', '15:00', '15:30'],
};
export default defineComponent({
  name: 'IndexPage',

  data() {
    return {
      form: {
        date: '',
        time: '',
      },
      allDates: [] as dateSelection[],
      submitted: [] as Array<{ date: string; time: string }>,
      allDatesTemp: [] as dateSelection[],
      selectedDate: '' as string,
      times: [] as string[],
      displayTimes: false,
      selectedTimes: [] as string[],
      displayRemoveInfo: false,
      clickedIDforRemove: null as number | null,
      openedExpansion: false,
      displayRemoveUserInfo: false,
      clickedUserForRemove: { userID: null as number | null, name: '' as string },
      users: [] as Array<{ userID: number; name: string; email?: string; dates?: Date[] }>,
      newUser: { userID: '', email: null } as { userID: string; email: string | null },
    };
  },

  methods: {
    generateDayTimes(date: string) {
      if (!date) {
        return [];
      }
      if (this.form.time) {
        this.selectedTimes = [`${date} ${this.form.time}`];
        return [`${date} ${this.form.time}`];
      }
      const times: string[] = [];
      Object.values(dayTimes).forEach((timeArray) => {
        timeArray.forEach((time) => {
          times.push(`${date} ${time}`);
        });
      });
      this.selectedTimes = times;
      return times;
    },
    getDates() {
      fetch('https://backendday.vercel.app/api/getDates', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          userid: 123,
        }),
      })
        .then((response) => response.json())
        .then((data) => {
          if (Array.isArray(data.dates)) {
            data.dates.sort(
              (a: { date: string }, b: { date: string }) =>
                new Date(a.date).getTime() - new Date(b.date).getTime(),
            );
          }

          this.allDates = data.dates.map((d: dateSelection) => ({
            id: d.id,
            date: this.formatDate(d.date),
            raw: d.date,
            block: d.block,
          })) as dateSelection[];
          this.allDatesTemp = this.allDates;
        })
        .catch((error) => {
          console.error('Error fetching dates:', error);
        });
    },
    removeDate(dateId: number) {
      this.clickedIDforRemove = dateId;
      this.displayRemoveInfo = true;
    },
    async removeConfirm() {
      await fetch('https://backendday.vercel.app/api/removeDate', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          id: this.clickedIDforRemove,
        }),
      }).then((response) => {
        if (!response.ok) {
          throw new Error('Network response was not ok');
        }
        this.getDates();
        this.displayRemoveInfo = false;
        this.clickedIDforRemove = null;
        this.selectedDate = '';
        this.openedExpansion = false;
      });
    },
    submitNewDates() {
      const timesAsDateSelection = this.selectedTimes.map((dt) => ({
        date: this.formatDate(dt),
        raw: dt,
        block: (() => {
          const timePart = String(dt).split(' ')[1] ?? '';
          const hour = parseInt(timePart.split(':')[0] ?? '0', 10);
          return hour >= 9 && hour < 12 ? 1 : 2;
        })(),
      }));
      fetch('https://backendday.vercel.app/api/submitDates', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          userid: 123,
          dates: timesAsDateSelection,
        }),
      })
        .then((response) => {
          if (!response.ok) {
            throw new Error('Network response was not ok');
          }
          return response.json();
        })
        .then(() => {
          this.getDates();
          this.displayTimes = false;
          this.selectedTimes = [];
        })
        .catch((error) => {
          console.error('Error submitting dates:', error);
        });
    },

    formatDate(rawDate: string, withTime: boolean = true) {
      const d = new Date(rawDate);
      if (!withTime) {
        return d.toLocaleDateString('sk-SK', {
          day: '2-digit',
          month: 'long',
          year: 'numeric',
        });
      }
      return d.toLocaleDateString('sk-SK', {
        day: '2-digit',
        month: 'long',
        year: 'numeric',
        hour: '2-digit',
        minute: '2-digit',
      });
    },

    submitForm() {
      this.displayTimes = true;
      this.times = this.generateDayTimes(this.form.date);
    },

    getUsers() {
      fetch('https://backendday.vercel.app/api/getAllUsers', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ userid: 123 }),
      })
        .then((r) => r.json())
        .then((data) => {
          if (Array.isArray(data.users)) {
            this.users = data.users.map(
              (u: { userID: number; name: string; email: string; date1: Date; date2: Date }) => ({
                userID: u.userID,
                name: u.name,
                email: u.email,
                dates: [u.date1, u.date2],
              }),
            );
          } else {
            this.users = [];
          }
        })
        .catch((err) => {
          console.error('Error fetching users', err);
          this.users = [];
        });
    },

    createUser() {
      if (!this.newUser.userID) return;
      fetch('https://backendday.vercel.app/api/createUser', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ user: this.newUser }),
      })
        .then((r) => {
          if (!r.ok) throw new Error('Network response was not ok');
          return r.json();
        })
        .then(() => {
          this.newUser.userID = '';
          this.newUser.email = null;
          this.getUsers();
        })
        .catch((err) => console.error('Error creating user', err));
    },

    removeUser(userId: number) {
      // open confirmation dialog and store selected user
      if (!userId) return;
      const user = this.users.find((u) => u.userID === userId);
      this.clickedUserForRemove = { userID: userId, name: user?.name ?? '' };
      this.displayRemoveUserInfo = true;
    },

    removeUserConfirm() {
      const id = this.clickedUserForRemove.userID;
      if (!id) return;
      fetch('https://backendday.vercel.app/api/removeUser', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ userID: id }),
      })
        .then((r) => {
          if (!r.ok) throw new Error('Network response was not ok');
          this.getUsers();
          this.displayRemoveUserInfo = false;
          this.clickedUserForRemove = { userID: null, name: '' };
        })
        .catch((err) => console.error('Error removing user', err));
    },

    selectDate(date: string) {
      if (this.selectedDate === date) {
        this.selectedDate = '';
        return;
      }

      this.selectedDate = date;
    },
  },

  computed: {
    allDatesGroupByDay(): Array<{ date: string }> {
      const uniqueDates = new Set<string>();
      this.allDates.forEach((d) => {
        const formatted = this.formatDate(String(d.raw), false);
        uniqueDates.add(formatted);
      });
      if (this.selectedDate) {
        return Array.from(uniqueDates)
          .map((date) => ({ date }))
          .filter((d) => d.date === this.selectedDate);
      }
      return Array.from(uniqueDates).map((date) => ({ date }));
    },

    availableDates(): Array<{ id: number | null; date: string }> {
      return this.allDates
        .filter((d) => this.formatDate(String(d.raw), false) === this.selectedDate)
        .map((d) => ({
          id: d.id,
          date: this.formatDate(String(d.raw), true),
        }));
    },
  },

  mounted() {
    this.getDates();
    this.getUsers();
  },
});
</script>
