<template>
    <div class="bg" ondragstart="return false;">
        <div class="head">
            <div class="title">
                <a @click="setPanel('stickers')" :class="{ selected: panel === 'stickers' }">自定义表情</a>
                <a @click="setPanel('face')" :class="{ selected: panel === 'face' }">小黄脸</a>
                <a @click="setPanel('remote')" :class="{ selected: panel === 'remote' }" v-if="supportRemote">云端</a>
                <a @click="setPanel('emojis')" :class="{ selected: panel === 'emojis' }">Emojis</a>
            </div>
            <a @click="menu">
                <div class="opinion">
                    <i class="el-icon-more"></i>
                </div>
            </a>
        </div>
        <div v-show="panel === 'face'" class="panel face-panel">
            <div class="subheader" v-show="recentFace.length">最近使用</div>
            <div class="grid" v-show="recentFace.length">
                <div v-for="i in recentFace" :key="i">
                    <img
                        :src="getFacePreview(i)"
                        @click="pickFace(i)"
                        @click.right="pickLottie(i)"
                        :title="getFaceName(i)"
                    />
                </div>
            </div>
            <div class="subheader">
                超级表情<br />
                <small>右键作为超级表情发送</small>
            </div>
            <div class="grid">
                <div v-for="i in faceIdToLottie.keys()" :key="i">
                    <img
                        :src="getFacePreview(i)"
                        @click="pickFace(i)"
                        @click.right="pickLottie(i)"
                        :title="getFaceName(i)"
                    />
                </div>
            </div>
            <div class="subheader">全部表情</div>
            <div class="grid" v-show="face.length">
                <div v-for="i in face" :key="i">
                    <img
                        :src="getFacePreview(i)"
                        @click="pickFace(i)"
                        @click.right="pickLottie(i)"
                        :title="getFaceName(i)"
                    />
                </div>
            </div>
        </div>
        <div v-show="panel === 'remote'" class="panel">
            <div class="subheader" v-show="recentRemoteSticker.length">最近使用</div>
            <div class="grid" v-show="recentRemoteSticker.length">
                <div v-for="i in recentRemoteSticker" :key="i">
                    <img :src="i" @click="sendRemoteSticker(i)" @click.right="remoteStickerMenu(i, $event)" />
                </div>
            </div>
            <div class="subheader" v-show="recentRemoteSticker.length">全部表情</div>
            <div class="empty" v-show="!remote_pics.length">云端没有任何表情</div>
            <div class="grid" v-show="remote_pics.length">
                <div v-for="i in remote_pics" :key="i.id">
                    <img
                        :src="i.url"
                        @click="sendRemoteSticker(i.url)"
                        @click.right="remoteStickerMenu(i.url, $event)"
                    />
                </div>
            </div>
        </div>
        <div class="stickers_dir" v-if="panel === 'stickers'" @wheel="wheelHandler" ref="stickers_dir">
            <a @click="changeCurrentDir(RECENT_CATEGORY)" :class="{ selected: current_dir === RECENT_CATEGORY }">
                最近
            </a>
            <a @click="changeCurrentDir(DEFAULT_CATEGORY)" :class="{ selected: current_dir === DEFAULT_CATEGORY }">
                默认
            </a>
            <a
                v-for="i in subdirs"
                :key="i"
                @click="changeCurrentDir(i)"
                @click.right="dirMenu(i, $event)"
                :class="{ selected: current_dir === i }"
                >{{ i }}</a
            >
        </div>
        <div v-if="panel === 'stickers'" class="panel">
            <div class="empty" v-show="!pics.length">
                找不到表情
                <el-button v-show="current_dir !== RECENT_CATEGORY" @click="folder">Open stickers folder</el-button>
            </div>
            <div class="grid" v-show="pics.length">
                <div v-for="i in pics" :key="i">
                    <img
                        :src="getStickerPreview(i)"
                        :data-relative-path="i"
                        @click="sendLocalSticker(i)"
                        @click.right="localStickerMenu(i, $event)"
                        @error="errorHandler"
                        @mouseover="onmouseover"
                        @mouseout="onmouseout"
                    />
                </div>
            </div>
        </div>
        <div class="emoji-panel" v-show="panel === 'emojis'">
            <VEmojiPicker @select="$emit('selectEmoji', $event)" />
        </div>
    </div>
</template>

<script>
import { VEmojiPicker } from 'v-emoji-picker'
import { shell } from 'electron'
import ipc from '../utils/ipc'
import fs from 'fs'
import path from 'path'
import md5 from 'md5'
import getStaticPath from '../../utils/getStaticPath'
import { faceIdToLottie } from '@icalingua/types/LottieFaceType'

const faceMap = require('oicq-icalingua-plus-plus/lib/message/face').map

const DEFAULT_CATEGORY = Symbol('DEFAULT')
const RECENT_CATEGORY = Symbol('RECENT')

const RECENTS = {
    max: {
        recentLocalSticker: 32,
        recentRemoteSticker: 8,
        recentFace: 18,
    },
    recents: {},
    get(name) {
        if (!(name in this.recents)) {
            this.recents[name] = []
            try {
                const recents = JSON.parse(localStorage[name])
                if (recents instanceof Array) {
                    this.recents[name] = recents
                }
            } catch (e) {}
        }
        return this.recents[name]
    },
    push(name, img) {
        let recents = this.get(name)
        const index = recents.indexOf(img)
        if (index !== -1) {
            recents = [img, ...recents.slice(0, index), ...recents.slice(index + 1)]
        } else {
            recents = [img, ...recents.slice(0, this.max[name] - 1)]
        }
        localStorage[name] = JSON.stringify(recents)
        this.recents[name] = recents
    },
}

export default {
    name: 'Stickers',
    components: { VEmojiPicker },
    watch: {
        panel() {
            this.recentFace = RECENTS.get('recentFace')
            this.recentRemoteSticker = RECENTS.get('recentRemoteSticker')
        },
        open() {
            this.recentFace = RECENTS.get('recentFace')
            this.recentRemoteSticker = RECENTS.get('recentRemoteSticker')
        },
    },
    props: {
        open: { type: Boolean, required: false, default: false },
    },
    data() {
        return {
            remote_pics: [],
            face: [],
            pics: [],
            panel: '',
            subdirs: [],
            current_dir: DEFAULT_CATEGORY,
            supportRemote: false,
            recentFace: [],
            recentRemoteSticker: [],
            descSortStickersByTime: true,
        }
    },
    async created() {
        this.DEFAULT_CATEGORY = DEFAULT_CATEGORY
        this.RECENT_CATEGORY = RECENT_CATEGORY
        this.face_dir = path.join(getStaticPath(), 'face/')
        this.faceIdToLottie = faceIdToLottie
        this.watchedPath = {}
        this.generatingPath = new Set()
        this.panel = await ipc.getLastUsedStickerType()
        this.descSortStickersByTime = (await ipc.getSettings()).descSortStickersByTime

        // Remote Stickers
        if (!(await ipc.getDisabledFeatures()).includes('RemoteStickers')) {
            this.supportRemote = true
            setTimeout(async () => (this.remote_pics = await ipc.getRoamingStamp(true)), 10 * 1000)
            setInterval(async () => (this.remote_pics = await ipc.getRoamingStamp(true)), 1000 * 60 * 60)
        }

        // Face
        if (!fs.existsSync(this.face_dir)) {
            this.$message.error('No face folder found!')
            await fs.promises.mkdir(this.face_dir)
        }
        fs.readdir(this.face_dir, (_err, files) => {
            this.face = files
        })

        // Stickers
        const store_dir = await ipc.getStorePath()
        this.default_dir = path.join(store_dir, 'stickers/')
        this.preview_dir = path.join(store_dir, 'stickers_preview/')
        if (!fs.existsSync(this.default_dir)) {
            await fs.promises.mkdir(this.default_dir)
        }
        if (!fs.existsSync(this.preview_dir)) {
            await fs.promises.mkdir(this.preview_dir)
        }
        const updateDefaultDir = async () => {
            if (this.current_dir != DEFAULT_CATEGORY) return
            /** @type {[string, fs.Stats][]} */
            let fileAndStats
            try {
                fileAndStats = await Promise.all(
                    (await fs.promises.readdir(this.default_dir))
                        .filter((i) => !i.startsWith('.'))
                        .map(async (i) => [i, await fs.promises.stat(this.default_dir + i)]),
                )
            } catch (err) {
                console.error('Failed to update sticker dir', DEFAULT_CATEGORY, err)
                return
            }
            this.subdirs = fileAndStats
                .filter(([_, stat]) => stat.isDirectory())
                .map(([i, _]) => i)
                .sort()
            if (!this.descSortStickersByTime) {
                this.pics = fileAndStats.filter(([_, stat]) => stat.isFile()).map(([i, _]) => i)
            } else {
                // 后添加的表情排在前面，类似于QQ
                this.pics = fileAndStats
                    .filter(([_, stat]) => stat.isFile())
                    .sort(([_a, statA], [_b, statB]) => statB.mtime - statA.mtime)
                    .map(([i, _]) => i)
            }
        }
        updateDefaultDir()
        fs.watch(this.default_dir, updateDefaultDir)
    },
    methods: {
        getStickerPreview(relPath) {
            return 'file://' + this.preview_dir + md5(relPath)
        },
        getFacePreview(i) {
            let faceId = String(i)
            if (faceId.length < 3) {
                faceId = '0'.repeat(3 - faceId.length) + faceId
            }
            return 'file://' + this.face_dir + faceId
        },
        getFaceName(i) {
            return String(faceMap[parseInt(i)] || '').replace(/\//, '')
        },
        errorHandler(e) {
            // generate preview
            const relPath = e.target.dataset.relativePath
            const previewPath = this.getStickerPreview(relPath).replace(/^file:\/\//, '')
            if (fs.existsSync(previewPath) || this.generatingPath.has(relPath)) {
                return
            }
            console.log('Generating preview for', relPath)
            this.generatingPath.add(relPath)
            const img = document.createElement('img')
            img.src = 'file://' + path.join(this.default_dir, relPath)
            img.onload = async () => {
                const canvas = document.createElement('canvas')
                canvas.width = img.width
                canvas.height = img.height
                const ctx = canvas.getContext('2d')
                ctx.drawImage(img, 0, 0, img.width, img.height)
                const dataURL = canvas.toDataURL('image/webp', 0.8)
                const base64Data = dataURL.replace(/^data:image\/webp;base64,/, '')
                try {
                    await fs.promises.writeFile(previewPath, base64Data, 'base64')
                } catch (err) {
                    console.error('Failed to generate preview for', relPath, 'at', previewPath)
                    console.error(err)
                    return
                } finally {
                    this.generatingPath.delete(relPath)
                }
                console.log('Preview generated for', relPath)
                e.target.src = e.target.src
            }
        },
        changeCurrentDir(dir) {
            console.log('Stickers directory changed:', dir)
            this.current_dir = dir
            if (dir == RECENT_CATEGORY) {
                this.pics = RECENTS.get('recentLocalSticker')
                return
            }
            const subDir = dir == DEFAULT_CATEGORY ? '' : dir + '/'
            const fullDir = this.default_dir + subDir
            const updateDir = async () => {
                if (this.current_dir != dir) return
                /** @type {[string, fs.Stats][]} */
                let fileAndStats
                try {
                    fileAndStats = await Promise.all(
                        (await fs.promises.readdir(fullDir))
                            .filter((i) => !i.startsWith('.'))
                            .map(async (i) => [i, await fs.promises.stat(fullDir + i)]),
                    )
                } catch (err) {
                    console.error('Failed to update sticker dir', dir, err)
                    return
                }
                if (!this.descSortStickersByTime) {
                    this.pics = fileAndStats.filter(([_, stat]) => stat.isFile()).map(([i, _]) => subDir + i)
                } else {
                    this.pics = fileAndStats
                        .filter(([_, stat]) => stat.isFile())
                        .sort(([_a, statA], [_b, statB]) => statB.mtime - statA.mtime)
                        .map(([i, _]) => subDir + i)
                }
            }
            updateDir()
            if (dir == DEFAULT_CATEGORY || dir in this.watchedPath) return
            this.watchedPath[dir] = fs.watch(fullDir, updateDir)
        },
        sendLocalSticker(img) {
            this.$emit('send', this.default_dir + img)
            RECENTS.push('recentLocalSticker', img)
        },
        sendRemoteSticker(img) {
            this.$emit('send', img)
            RECENTS.push('recentRemoteSticker', img)
        },
        pickFace(face) {
            this.$emit('selectFace', face)
            RECENTS.push('recentFace', parseInt(face))
        },
        pickLottie(face) {
            const faceId = parseInt(face)
            const qlottie = faceIdToLottie.get(faceId)
            if (!qlottie) {
                this.$message.error(
                    `${String(faceMap[faceId] || 'Face').replace(/\//, '')}(${faceId}) 没有对应的 Lottie 超级表情`,
                )
                return
            }
            this.$emit('sendLottie', { qlottie: qlottie.lottieId, id: face })
            RECENTS.push('recentFace', faceId)
        },
        folder() {
            shell.openPath(
                this.current_dir == DEFAULT_CATEGORY ? this.default_dir : path.join(this.default_dir, this.current_dir),
            )
        },
        menu: ipc.popupStickerMenu,
        localStickerMenu(relPath, e) {
            ipc.popupStickerItemMenu(
                this.default_dir + relPath,
                this.pics.map((i) => this.default_dir + i),
                e,
            )
        },
        remoteStickerMenu(url, e) {
            ipc.popupStickerItemMenu(
                url,
                this.remote_pics.map((i) => i.url),
                e,
            )
        },
        dirMenu: ipc.popupStickerDirMenu,
        setPanel(type) {
            this.panel = type
            ipc.setLastUsedStickerType(type)
        },
        wheelHandler(e) {
            e.preventDefault()
            this.$refs.stickers_dir.scrollTo({
                left: this.$refs.stickers_dir.scrollLeft + e.deltaY,
                behavior: 'smooth',
            })
        },
        onmouseover(e) {
            e.target.src = 'file://' + path.join(this.default_dir, e.target.dataset.relativePath)
        },
        onmouseout(e) {
            e.target.src = this.getStickerPreview(e.target.dataset.relativePath)
        },
    },
}
</script>

<style scoped lang="scss">
.stickers_dir {
    width: 100%;
    white-space: nowrap;
    overflow-x: auto;
    overflow-y: hidden;
    border-bottom: var(--chat-border-style);
    background-color: var(--panel-header-bg);
    display: flex;
    line-height: 30px;
    height: 30px;
    min-height: 30px;

    a {
        margin-right: 8px;
        color: var(--panel-color-sticker-type);

        &:hover {
            color: var(--panel-color-sticker-type-hover);
        }

        &.selected {
            color: var(--panel-color-sticker-type-selected);
        }
    }
}

.head {
    height: 64px;
    min-height: 64px;
    border-bottom: var(--chat-border-style);
    display: flex;
    align-items: center;
    font-size: 17px;
    padding: 0 16px;
    background-color: var(--panel-header-bg);
}

.title {
    width: 100%;

    a {
        margin-right: 8px;
        color: var(--panel-color-sticker-type);

        &:hover {
            color: var(--panel-color-sticker-type-hover);
        }

        &.selected {
            color: var(--panel-color-sticker-type-selected);
        }
    }
}

.opinion {
    margin-left: 5px;
    font-size: 16px;
    cursor: pointer;
    transition: all 0.2s;
    color: var(--chat-color);

    &:hover {
        transform: scale(1.1);
        opacity: 0.7;
    }
}

.panel {
    overflow: auto;
    height: 100vh;
}

.subheader {
    margin: 0.5em;
    color: var(--chat-color-placeholder);
}

.empty {
    padding: 1em;
    text-align: center;
    color: var(--chat-color-placeholder);

    button {
        margin-top: 1em;
    }
}

.grid {
    display: grid;
    overflow: hidden;
    grid-template-columns: 1fr 1fr 1fr 1fr;

    img {
        object-fit: contain;
        width: 100%;
        height: 100%;
        box-sizing: border-box;
        position: absolute;
        border: 1px solid transparent;
        transition: border-color 0.5s;

        &:hover {
            border-color: #999;
        }
    }

    div {
        width: 100%;
        height: 0;
        padding-bottom: 100%;
        position: relative;
        background-color: var(--panel-background);
    }
}

.face-panel {
    padding: 0 0.5em;

    .subheader {
        margin-left: 0;
        margin-right: 0;
    }

    .grid {
        margin: 0.5em 0;
        grid-template-columns: repeat(9, 1fr);
    }
}

.bg {
    background-color: var(--panel-background);
    height: -webkit-fill-available;
    display: flex;
    flex-direction: column;
}

@media screen and (min-width: 1200px) {
    .bg {
        border-left: var(--chat-border-style);
    }
}

// 修复 emoji 面板溢出
@mixin emoji-flex {
    display: flex;
    flex-direction: column;
    overflow: hidden;
}

.emoji-panel {
    @include emoji-flex;
    height: 100vh;
}

::v-deep #Emojis {
    @include emoji-flex;
    display: flex !important;
}

::v-deep #Categories,
::v-deep #InputSearch {
    flex-shrink: 0;
}

::v-deep .container-emoji {
    height: auto !important;
}

.emoji-picker {
    --ep-color-bg: auto !important;
    --ep-color-border: auto !important;
    --ep-color-sbg: #fff !important;
    --ep-color-active: #409eff !important;
    width: 100% !important;
    border: none !important;
}
</style>
