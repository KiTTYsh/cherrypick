<!--
SPDX-FileCopyrightText: syuilo and misskey-project
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<div v-if="files.files.length > 0" :class="$style.root">
	<div v-if="files.files[0].isSensitive && !showingFiles.includes(files.files[0].id)" :key="files.id + files.files[0].id" :class="$style.img" @click="showingFiles.push(files.files[0].id)">
		<!-- TODO: 画像以外のファイルに対応 -->
		<ImgWithBlurhash :class="$style.sensitiveImg" :hash="files.files[0].blurhash" :src="thumbnail(files.files[0])" :title="files.files[0].name" :forceBlurhash="true"/>
		<div :class="$style.sensitive">
			<div>
				<div><i class="ti ti-eye-exclamation"></i> {{ i18n.ts.sensitive }}</div>
				<div>{{ i18n.ts.clickToShow }}</div>
			</div>
		</div>
	</div>
	<MkA v-else :class="[$style.img, { [$style.multipleImg]: files.files.length > 1 }]" :to="notePage(files)">
		<!-- TODO: 画像以外のファイルに対応 -->
		<ImgWithBlurhash
			:hash="files.files[0].blurhash"
			:src="thumbnail(files.files[0])"
			:title="files.files[0].name"
			@mouseover="defaultStore.state.showingAnimatedImages === 'interaction' ? playAnimation = true : ''"
			@mouseout="defaultStore.state.showingAnimatedImages === 'interaction' ? playAnimation = false : ''"
			@touchstart="defaultStore.state.showingAnimatedImages === 'interaction' ? playAnimation = true : ''"
			@touchend="defaultStore.state.showingAnimatedImages === 'interaction' ? playAnimation = false : ''"
		/>
	</MkA>
	<div v-if="files.files.length > 1" :class="$style.multiple">
		<i class="ti ti-box-multiple"></i>
	</div>
</div>
</template>

<script lang="ts" setup>
import { onMounted, onUnmounted, ref } from 'vue';
import * as Misskey from 'cherrypick-js';
import { getStaticImageUrl } from '@/scripts/media-proxy.js';
import { notePage } from '@/filters/note.js';
import ImgWithBlurhash from '@/components/MkImgWithBlurhash.vue';
import MkA from '@/components/global/MkA.vue';
import { defaultStore } from '@/store.js';
import { i18n } from '@/i18n.js';

const props = defineProps<{
	user: Misskey.entities.UserDetailed;
	files: Misskey.entities.DriveFile[];
}>();

const showingFiles = ref<string[]>([]);

const playAnimation = ref(true);
if (defaultStore.state.showingAnimatedImages === 'interaction') playAnimation.value = false;
let playAnimationTimer = setTimeout(() => playAnimation.value = false, 5000);

function thumbnail(image: Misskey.entities.DriveFile): string | null {
	return (defaultStore.state.disableShowingAnimatedImages || defaultStore.state.dataSaver.media) || (['interaction', 'inactive'].includes(<string>defaultStore.state.showingAnimatedImages) && !playAnimation.value)
		? getStaticImageUrl(image.url)
		: image.thumbnailUrl;
}

function resetTimer() {
	playAnimation.value = true;
	clearTimeout(playAnimationTimer);
	playAnimationTimer = setTimeout(() => playAnimation.value = false, 5000);
}

onMounted(() => {
	if (defaultStore.state.showingAnimatedImages === 'inactive') {
		window.addEventListener('mousemove', resetTimer);
		window.addEventListener('touchstart', resetTimer);
		window.addEventListener('touchend', resetTimer);
	}
});

onUnmounted(() => {
	if (defaultStore.state.showingAnimatedImages === 'inactive') {
		window.removeEventListener('mousemove', resetTimer);
		window.removeEventListener('touchstart', resetTimer);
		window.removeEventListener('touchend', resetTimer);
	}
});
</script>

<style lang="scss" module>
.root {
	position: relative;
}

.img {
	display: flex;
	position: relative;
	height: 256px;
	border-radius: 3px;
	overflow: clip;
}

.empty {
	margin: 0;
	padding: 16px;
	text-align: center;
}

.sensitiveImg {
	position: absolute;
	top: 0;
	left: 0;
	width: 100%;
	height: 100%;
	filter: brightness(0.7);
}

.sensitive {
	position: absolute;
	top: 0;
	left: 0;
	width: 100%;
	height: 100%;
	display: grid;
  place-items: center;
	font-size: 0.8em;
	color: #fff;
	cursor: pointer;
}

.multipleImg {
	filter: brightness(0.9);
}

.multiple {
	position: absolute;
	top: 10px;
	right: 10px;
	font-size: 1.45em;
	color: #fff;
	opacity: .9;
}

@container (max-width: 785px) {
	.img {
		height: 192px;
	}
}

@container (max-width: 660px) {
	.img {
		height: 160px;
	}
}

@container (max-width: 530px) {
	.img {
		height: 145px;
	}

	.multiple {
		top: 7px;
		right: 7px;
		font-size: 1.3em;
	}
}
</style>
