<template>
  <span class="d-inline-block v-align-middle line-height-0">
    <UiAvatar :imgsrc="url" :address="spaceAddress" :size="size" />
  </span>
</template>

<script>
import { formatBytes32String } from '@ethersproject/strings';

export default {
  props: ['space', 'size', 'symbolIndex'],
  data() {
    return {
      error: false
    };
  },
  computed: {
    url() {
      return this.space.avatar
    },
    spaceAddress() {
      if (this.spaceId) return formatBytes32String(this.spaceId.slice(0, 24));
      return '';
    },
    spaceId() {
      return this.space.id ?? this.space.key;
    }
  }
};
</script>
