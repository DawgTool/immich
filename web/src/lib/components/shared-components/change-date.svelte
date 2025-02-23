<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  import { DateTime } from 'luxon';
  import ConfirmDialogue from './confirm-dialogue.svelte';
  import Combobox from './combobox.svelte';

  export let initialDate: DateTime = DateTime.now();

  type ZoneOption = {
    /**
     * Timezone name
     *
     * e.g. Europe/Berlin
     */
    label: string;

    /**
     * Timezone offset
     *
     * e.g. UTC+01:00
     */
    value: string;
  };

  const timezones: ZoneOption[] = Intl.supportedValuesOf('timeZone').map((zone: string) => ({
    label: zone + ` (${DateTime.local({ zone }).toFormat('ZZ')})`,
    value: 'UTC' + DateTime.local({ zone }).toFormat('ZZ'),
  }));

  const initialOption = timezones.find((item) => item.value === 'UTC' + initialDate.toFormat('ZZ'));

  let selectedOption = initialOption && {
    label: initialOption?.label || '',
    value: initialOption?.value || '',
  };

  let selectedDate = initialDate.toFormat("yyyy-MM-dd'T'HH:mm");

  // Keep local time if not it's really confusing
  $: date = DateTime.fromISO(selectedDate).setZone(selectedOption?.value, { keepLocalTime: true });

  const dispatch = createEventDispatcher<{
    cancel: void;
    confirm: string;
  }>();

  const handleCancel = () => dispatch('cancel');

  const handleConfirm = () => {
    const value = date.toISO();
    if (value) {
      dispatch('confirm', value);
    }
  };

  const handleKeydown = (event: KeyboardEvent) => {
    if (['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight'].includes(event.key)) {
      event.stopPropagation();
    }
  };
</script>

<div role="presentation" on:keydown={handleKeydown}>
  <ConfirmDialogue
    confirmColor="primary"
    cancelColor="secondary"
    title="Edit date & time"
    prompt="Please select a new date:"
    disabled={!date.isValid}
    on:confirm={handleConfirm}
    on:cancel={handleCancel}
  >
    <div class="flex flex-col text-md px-4 text-center gap-2" slot="prompt">
      <div class="mt-2" />
      <div class="flex flex-col">
        <label for="datetime">Date and Time</label>
        <input
          class="immich-form-input text-sm my-4 w-full"
          id="datetime"
          type="datetime-local"
          bind:value={selectedDate}
        />
      </div>
      <div class="flex flex-col w-full mt-2">
        <label for="timezone">Timezone</label>
        <Combobox bind:selectedOption id="timezone" options={timezones} placeholder="Search timezone..." />
      </div>
    </div>
  </ConfirmDialogue>
</div>
