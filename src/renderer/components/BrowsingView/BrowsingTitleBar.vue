<template>
  <div
    class="browsing-title-bar"
  >
    <div
      @mouseup="handleMinimize"
      class="control-button minimize-icon no-drag button-hover"
    >
      <Icon
        type="browsingminimize"
      />
    </div>
    <div
      @mouseup="handleMiddleButton"
      class="control-button fullscreen-icon button-hover"
    >
      <Icon
        ref="back"
        :type="isFullScreen ? 'browsingExitFull'
          : isMaximized ? 'browsingrestore' : 'browsingfullscreen'"
      />
    </div>
    <div
      @mouseup="handleClose"
      class="control-button close-icon button-hover"
    >
      <Icon
        ref="forward"
        type="browsingclose"
      />
    </div>
  </div>
</template>
<script lang="ts">
import { mapGetters } from 'vuex';
import Icon from '@/components/BaseIconContainer.vue';

export default {
  components: {
    Icon,
  },
  props: {
    showSidebar: {
      type: Boolean,
      required: true,
    },
  },
  computed: {
    ...mapGetters(['isMaximized', 'isFullScreen']),
  },
  methods: {
    handleMinimize() {
      this.$electron.ipcRenderer.send('callMainWindowMethod', 'minimize');
    },
    handleMiddleButton() {
      const currentWindow = this.$electron.remote.getCurrentWindow();
      if (this.isFullScreen) {
        this.$electron.ipcRenderer.send('callMainWindowMethod', 'setFullScreen', [false]);
      } else if (this.isMaximized) {
        this.$electron.ipcRenderer.send('callMainWindowMethod', 'unmaximize');
        currentWindow.getBrowserViews()[0].setBounds({
          x: this.showSidebar ? 76 : 0,
          y: 40,
          width: this.showSidebar ? currentWindow.getSize()[0] - 76
            : currentWindow.getSize()[0],
          height: currentWindow.getSize()[1] - 40,
        });
      } else {
        this.$electron.ipcRenderer.send('callMainWindowMethod', 'maximize');
        const bounds = currentWindow.getBounds();
        currentWindow.getBrowserViews()[0].setBounds({
          x: this.showSidebar ? 76 : 0,
          y: 40,
          width: this.showSidebar ? bounds.width + (bounds.x * 2) - 76
            : bounds.width + (bounds.x * 2),
          height: bounds.height - 40,
        });
      }
    },
    handleClose() {
      this.$electron.ipcRenderer.send('callMainWindowMethod', 'close');
    },
  },
};
</script>
<style lang="scss" scoped>
.browsing-title-bar {
  height: 100%;
  display: flex;
  align-items: center;
  z-index: 6;
  width: 114px;
  border-left: 1px solid #F2F1F4;
  .control-button {
    width: 30px;
    height: 30px;
    border-radius: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    transition: background-color 100ms ease-in;
  }
  .button-hover:hover {
    background-color: #ECEEF0;
  }
  .minimize-icon {
    margin-left: 8px;
  }
  .fullscreen-icon {
    margin-left: 4px;
    margin-right: 4px;
  }
  .close-icon {
    margin-right: 8px;
  }
}
</style>
