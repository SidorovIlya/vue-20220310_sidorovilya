<template>
  <form class="meetup-form" @submit.prevent="handleSubmit">
    <div class="meetup-form__content">
      <fieldset class="meetup-form__section">
        <ui-form-group label="Название">
          <ui-input name="title" v-model="innerMeetup.title" />
        </ui-form-group>
        <ui-form-group label="Дата">
          <ui-input-date type="date" name="date" v-model="innerMeetup.date" />
        </ui-form-group>
        <ui-form-group label="Место">
          <ui-input name="place" v-model="innerMeetup.place" />
        </ui-form-group>
        <ui-form-group label="Описание">
          <ui-input multiline rows="3" name="description" v-model="innerMeetup.description" />
        </ui-form-group>
        <ui-form-group label="Изображение">
          <!--
               Предлагается сохранять файл для будущей загрузки во временное поле imageToUpload
               и передавать в объекте со всеми данными формы
          -->
          <ui-image-uploader
            name="image"
            :preview="innerMeetup.image"
            @select="innerMeetup.imageToUpload = $event"
            @remove="innerMeetup.imageToUpload = null"
          />
        </ui-form-group>
      </fieldset>

      <h3 class="meetup-form__agenda-title">Программа</h3>
      
      <meetup-agenda-item-form
         :key="agendaItem.id"
         :agenda-item="agendaItem"
         class="meetup-form__agenda-item"
         v-for="(agendaItem, index) in innerMeetup.agenda"
         @remove="deleteItem(index)"
         @update:agendaItem="changeItem(index, $event)"
       />       

      <div class="meetup-form__append">
        <button class="meetup-form__append-button" type="button" data-test="addAgendaItem" @click="addItem">
          + Добавить этап программы
        </button>
      </div>
    </div>

    <div class="meetup-form__aside">
      <div class="meetup-form__aside-stick">
        <!-- data-test атрибуты используются для поиска нужного элемента в тестах, не удаляйте их -->
        <ui-button variant="secondary" block class="meetup-form__aside-button" data-test="cancel" @click="$emit('cancel')">Отмена</ui-button>
        <ui-button variant="primary" block class="meetup-form__aside-button" data-test="submit" type="submit">
          {{ submitText }}
        </ui-button>
      </div>
    </div> 
  </form>
</template>

<script>
import { cloneDeep } from 'lodash-es';

import MeetupAgendaItemForm from './MeetupAgendaItemForm';
import UiButton from './UiButton';
import UiFormGroup from './UiFormGroup';
import UiImageUploader from './UiImageUploader';
import UiInput from './UiInput';
import UiInputDate from './UiInputDate';

import { createAgendaItem } from '../meetupService';

export default {
  name: 'MeetupForm',

  components: {
    MeetupAgendaItemForm,
    UiButton,
    UiFormGroup,
    UiImageUploader,
    UiInput,
    UiInputDate,
  },

  props: {
    meetup: {
      type: Object,
      required: true,
    },

    submitText: {
      type: String,
      default: '',
    },
  },

  data() {
    return {
      innerMeetup: cloneDeep(this.meetup)
    }
  },

  emits: ['submit', 'cancel'],

  methods: {
    handleSubmit() {
      this.$emit('submit', cloneDeep(this.innerMeetup));
    },
    addItem() {
      this.innerMeetup.agenda.push( createAgendaItem() );
      if (this.innerMeetup.agenda.length > 1) {
        this.innerMeetup.agenda[this.innerMeetup.agenda.length - 1].startsAt = this.innerMeetup.agenda[this.innerMeetup.agenda.length - 2].endsAt;
      }
    },
    deleteItem(index) {
      this.innerMeetup.agenda.splice(index, 1);
    },
    changeItem(index, event) {
      this.innerMeetup.agenda[index] = event;
    }
  }
};
</script>

<style scoped>
.meetup-form__section {
  border: none;
}

.meetup-form__agenda-title {
  font-weight: 700;
  font-size: 28px;
  line-height: 150%;
  color: var(--body-color);
  margin: 0 0 24px 0;
}

.meetup-form__aside {
  margin: 48px 0;
}

.meetup-form__aside-button {
  margin: 0 0 12px 0;
}

.meetup-form__agenda-item + .meetup-form__agenda-item {
  margin-top: 24px;
}

.meetup-form__append {
  margin-top: 24px;
}

.meetup-form__append-button {
  box-shadow: none;
  border: none;
  background-color: transparent;
  padding: 0;
  outline: none;
  color: var(--blue);
  cursor: pointer;
  font-size: 20px;
  line-height: 28px;
}

.meetup-form__append-button:hover {
  text-decoration: underline;
}

@media all and (min-width: 992px) {
  .meetup-form {
    display: flex;
    flex-direction: row;
  }

  .meetup-form__content {
    flex: 1 0 calc(100% - 320px);
  }

  .meetup-form__aside {
    flex: 1 0 320px;
    max-width: 320px;
    width: 100%;
    padding-left: 137px;
    margin: 0;
  }

  .meetup-form__aside-stick {
    position: sticky;
    top: 32px;
  }
}
</style>
