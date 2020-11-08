<template>
    <div class="avatar" ref="avatar">
        <div style="position: absolute; width: inherit; height: inherit; pointer-events: none; background: rgba(21,21,21,0.48); backdrop-filter: blur(1px)"></div>
        <div class="avatar__cutter" draggable="false" @dragstart.prevent="" ref="avatarCutter">
            <div class="avatar__cutter__inner" @mousedown="move" @dragstart.prevent="" draggable="false" :style="{background: `url(${miniSrc})`}"></div>
            <div class="avatar__cutter__resize-controller" @mousedown="resize" draggable="false" ref="resizeController"></div>
        </div>

        <img class="avatar__img" :src="src" draggable="false" ref="avatarImg"/>
    </div>
</template>

<script>
    export default {
        name: 'Avatar',
        data() {
          return {
              miniSrc: ""
          }
        },
        props: {
            src: {
                type: [String, Object]
            }
        },
        watch: {
            src() {
                this.processImage(this.src);
            }
        },
        methods: {
            move(downEvent) {
                const mousemove = (moveEvent) => {
                    const {x, y} = this.limitMove(downEvent, moveEvent);

                    downEvent.target.parentElement.style.top = y || moveEvent.clientY - (downEvent.target.parentElement.clientHeight / 2) + "px";
                    downEvent.target.parentElement.style.left = x || moveEvent.clientX - (downEvent.target.parentElement.clientWidth / 2) + "px";
                    this.cutImage();
                }
                const mouseUp = () => {
                    document.body.removeEventListener("mousemove", mousemove);
                    document.body.removeEventListener("mouseup", mouseUp);
                }

                document.body.addEventListener("dragstart", () => false);
                document.body.addEventListener("mousemove", mousemove);
                document.body.addEventListener("mouseup", mouseUp);

            },
            resize(downEvent) {
                if (!downEvent.target.parentElement.style.top) {
                    downEvent.target.parentElement.style.top = downEvent.target.parentElement.offsetTop + "px";
                    downEvent.target.parentElement.style.left = downEvent.target.parentElement.offsetLeft + "px";
                }


                const mousemove = (moveEvent) => {
                    const sizeStep = moveEvent.clientX - (downEvent.target.parentElement.offsetLeft) + "px";
                    downEvent.target.parentElement.style.width = sizeStep;
                    downEvent.target.parentElement.style.height = sizeStep;

                    const avatarSize = {
                        left: this.$refs.avatarImg.offsetLeft,
                        right: this.$refs.avatarImg.offsetLeft + this.$refs.avatarImg.offsetWidth,
                        top: this.$refs.avatarImg.offsetTop,
                        bottom: this.$refs.avatarImg.offsetTop + this.$refs.avatarImg.clientHeight,
                        height: this.$refs.avatarImg.clientHeight
                    }

                    const cutterSize = {
                        left: downEvent.target.parentElement.offsetLeft,
                        right: downEvent.target.parentElement.offsetLeft + downEvent.target.parentElement.offsetWidth,
                        top: downEvent.target.parentElement.offsetTop,
                        bottom: downEvent.target.parentElement.offsetTop + downEvent.target.parentElement.clientWidth,
                        widthAndHeight: downEvent.target.parentElement.clientWidth
                    }

                    if (cutterSize.right > avatarSize.right && cutterSize.bottom < avatarSize.bottom) {
                        downEvent.target.parentElement.style.width = avatarSize.right - cutterSize.left + "px";
                        downEvent.target.parentElement.style.height = avatarSize.right - cutterSize.left + "px";
                        return;
                    }

                    if (cutterSize.bottom > avatarSize.bottom) {
                        downEvent.target.parentElement.style.height = avatarSize.bottom - cutterSize.top - 1 + "px";
                        downEvent.target.parentElement.style.width = avatarSize.bottom - cutterSize.top - 1 + "px";
                        return;
                    }
                    if (cutterSize.widthAndHeight <= 50) {
                        downEvent.target.parentElement.style.width = "51px";
                        downEvent.target.parentElement.style.height = "51px";
                    }

                    this.cutImage();
                }
                const mouseUp = () => {
                    document.body.removeEventListener("mousemove", mousemove);
                    document.body.removeEventListener("mouseup", mouseUp);
                }

                document.body.addEventListener("mousemove", mousemove);
                document.body.addEventListener("dragstart", () => false);
                document.body.addEventListener("mouseup", mouseUp);
            },
            limitMove(downEvent, moveEvent) {
                let limitCoordinates = {
                    x: false,
                    y: false
                }

                const avatarSize = {
                    left: this.$refs.avatarImg.offsetLeft,
                    right: this.$refs.avatarImg.offsetLeft + this.$refs.avatarImg.offsetWidth,
                    top: this.$refs.avatarImg.offsetTop,
                    bottom: this.$refs.avatarImg.offsetTop + this.$refs.avatarImg.clientHeight
                }

                const cutterSize = {
                    left: downEvent.target.parentElement.offsetLeft,
                    right: downEvent.target.parentElement.offsetLeft + downEvent.target.parentElement.offsetWidth,
                    top: downEvent.target.parentElement.offsetTop,
                    bottom: downEvent.target.parentElement.offsetTop + downEvent.target.clientWidth,
                    widthAndHeight: downEvent.target.clientWidth
                }

                if (cutterSize.left <= avatarSize.left || moveEvent.clientX - cutterSize.widthAndHeight / 2 <= avatarSize.left) {
                    limitCoordinates.x = (avatarSize.left + 1) + "px";
                }
                if (cutterSize.right >= avatarSize.right || moveEvent.clientX + cutterSize.widthAndHeight / 2 >= avatarSize.right) {
                    limitCoordinates.x = (avatarSize.right - downEvent.target.clientWidth - this.$refs.resizeController.clientWidth / 2) + "px";
                }
                if (cutterSize.top <= avatarSize.top || moveEvent.clientY - cutterSize.widthAndHeight / 2 <= avatarSize.top) {
                    limitCoordinates.y = (avatarSize.top + 1) + "px";
                }
                if (cutterSize.bottom >= avatarSize.bottom || moveEvent.clientY + cutterSize.widthAndHeight / 2 >= avatarSize.bottom) {
                    limitCoordinates.y = (avatarSize.bottom - downEvent.target.clientHeight - 1) + "px";
                }

                return limitCoordinates;
            },
            processImage(src) {
                const newImage = new Image();
                newImage.onload = () => {
                    const canvas = document.createElement('canvas');
                    canvas.width = this.$refs.avatarImg.clientWidth;
                    canvas.height = this.$refs.avatarImg.clientHeight;
                    canvas.getContext('2d').drawImage(newImage, 0, 0, newImage.naturalWidth, newImage.naturalHeight, 0, 0, canvas.width, canvas.height);
                    this.$refs.avatarImg.src = canvas.toDataURL();
                    setTimeout(() => {
                        this.cutImage();
                    }, 0);
                }
                newImage.src = src;
            },
            cutImage() {

                const metaCutterImage = {
                    x: this.$refs.avatarCutter.offsetLeft - this.$refs.avatarImg.offsetLeft,
                    y: this.$refs.avatarCutter.offsetTop - this.$refs.avatarImg.offsetTop,
                    widthAndHeight: this.$refs.avatarCutter.clientWidth,
                    source: this.$refs.avatarImg
                }

                const cutterImage = document.createElement('canvas');
                cutterImage.width = metaCutterImage.widthAndHeight;
                cutterImage.height = metaCutterImage.widthAndHeight;

                cutterImage.getContext('2d').drawImage(
                    metaCutterImage.source,
                    metaCutterImage.x,
                    metaCutterImage.y,
                    metaCutterImage.widthAndHeight,
                    metaCutterImage.widthAndHeight,
                    0,
                    0,
                    cutterImage.width,
                    cutterImage.height);
                this.miniSrc = cutterImage.toDataURL();
                this.$emit('miniAvatar', cutterImage.toDataURL());
            }
        }
    }
</script>

<style lang="scss">
    .avatar {
        display: flex;
        justify-content: center;
        align-items: center;
        height: fit-content;
        z-index: 2;

        &__img {
            user-select: none;
            width: inherit;
            object-fit: contain;
        }

        &__cutter {
            backdrop-filter: blur(0);
            position: absolute;
            width: 80px;
            height: 80px;
            display: flex;
            flex-direction: column;
            justify-content: flex-end;
            overflow: visible;

            &__inner {
                cursor: grab;
                width: inherit;
                height: inherit;
                border-radius: 3px;
                border: 1px solid rgba(255, 255, 255, 0.4);
                background-size: contain;
            }

            &__resize-controller {
                cursor: nw-resize;
                position: absolute;
                bottom: -4px;
                right: -4px;
                width: 10px;
                height: 10px;
                background: rgba(198, 198, 198, 0.35);
                align-self: flex-end;
                border-radius: 1px;

                &:hover {
                    background: rgba(206, 206, 206, 0.76);
                }
            }
        }
    }

</style>