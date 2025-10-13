<template>
    <div class="mobile-desktop-container relative overflow-hidden">
        <!-- 页面容器 - 保留原页面切换逻辑 -->
        <div class="pages-container flex transition-transform duration-300 ease-out h-full"
            :style="{ transform: `translateX(-${currentPage * 100}%)` }" @touchstart="handleTouchStart"
            @touchmove="handleTouchMove" @touchend="handleTouchEnd" @mousedown="handleMouseDown"
            @mousemove="handleMouseMove" @mouseup="handleMouseUp" @mouseleave="handleMouseUp">
            <!-- 原有桌面页面内容（不变） -->
            <div v-for="(location, pageIndex) in locations" :key="location.id"
                class="desktop-page flex-shrink-0 w-full h-full relative" :style="{
                    backgroundImage: `url(${location.backgroundImage || defaultBackground})`,
                    backgroundSize: 'cover',
                    backgroundPosition: 'center'
                }">
                <div class="page-header bg-black/30 text-white p-2 backdrop-blur-sm">
                    <h2 class="text-lg font-medium">{{ location.name }}</h2>
                </div>
                <div class="page-content p-4 overflow-y-auto h-[calc(100%-40px)]">
                    <div class="icon-grid grid grid-cols-4 sm:grid-cols-5 md:grid-cols-6 lg:grid-cols-8 gap-4">
                        <div v-for="(item, itemIndex) in location.children" :key="item.id"
                            class="app-icon cursor-pointer transition-all duration-200 hover:scale-105 hover:shadow-lg"
                            :style="{
                                left: item.position?.left || null,
                                top: item.position?.top || null,
                                width: `${iconSize}px`
                            }" :data-id="item.id" :data-location-id="location.id" @click.stop="handleItemClick(item)">
                            <div
                                class="icon-wrapper w-full aspect-square rounded-lg bg-white/80 flex items-center justify-center shadow-md overflow-hidden">
                                <img v-if="getCachedImageUrl(item)" :src="getCachedImageUrl(item)" alt="Item icon"
                                    class="w-full h-full object-cover" @error="handleImageError(item)">
                                <div v-else class="w-full h-full flex items-center justify-center bg-gray-100">
                                    <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8 text-gray-400" fill="none"
                                        viewBox="0 0 24 24" stroke="currentColor">
                                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                            d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
                                    </svg>
                                </div>
                            </div>
                            <div class="icon-label w-full text-center text-xs truncate text-white mt-1 drop-shadow-md">
                                {{ item.name }}
                            </div>
                        </div>
                    </div>
                    <div v-if="location.children.length === 0" class="empty-state">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-12 w-12 mx-auto mb-2 opacity-70" fill="none"
                            viewBox="0 0 24 24" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2" />
                        </svg>
                        <p>当前位置暂无物品</p>
                    </div>
                </div>
            </div>
        </div>

        <!-- 原有页面指示器、编辑/添加按钮（不变） -->
        <div class="page-indicators absolute bottom-6 left-0 right-0 flex justify-center gap-2 z-10">
            <button v-for="(location, index) in locations" :key="location.id"
                class="indicator w-2.5 h-2.5 rounded-full transition-all duration-300" :class="{
                    'bg-white opacity-100 w-6': index === currentPage,
                    'bg-white/50': index !== currentPage
                }" @click="currentPage = index" :title="location.name"></button>
        </div>
        <button
            class="edit-bg-btn absolute top-12 right-4 p-2 rounded-full bg-black/30 text-white hover:bg-black/50 transition-colors z-10"
            @click="openBackgroundSelector">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                <path
                    d="M13.586 3.586a2 2 0 112.828 2.828l-.793.793-2.828-2.828.793-.793zM11.379 7.757L3 16.136V19h2.864l8.379-8.379-2.864-2.864z" />
            </svg>
        </button>
        <button
            class="add-item-btn absolute bottom-6 left-4 p-2 rounded-full bg-black/30 text-white hover:bg-black/50 transition-colors z-10"
            @click="handleAddItem">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd"
                    d="M10 18a8 8 0 100-16 8 8 0 000 16zm1-11a1 1 0 10-2 0v2H7a1 1 0 100 2h2v2a1 1 0 102 0v-2h2a1 1 0 100-2h-2V7z"
                    clip-rule="evenodd" />
            </svg>
        </button>

        <!-- ########## 核心修改：组件内分页弹窗（替代原全局弹窗） ########## -->
        <!-- 弹窗容器：absolute定位（组件内），点击外部可关闭 -->
        <div v-if="showItemPopup"
            class="folder-popup-backdrop absolute inset-0 bg-black/50 z-20 flex items-center justify-center p-4"
            @click="closeFolderPopup">
            <!-- 弹窗主体：占比80%，不触发外部关闭事件 -->
            <div class="folder-popup w-[80%] max-w-md bg-white rounded-xl overflow-hidden shadow-2xl" @click.stop>

                <!-- 1. 弹窗顶部栏（显示当前文件夹名称 + 关闭按钮） -->
                <div class="popup-header bg-white p-3 flex justify-between items-center border-b">
                    <div class="breadcrumbs text-sm font-medium text-gray-800 flex items-center gap-1">
                        <template v-for="(item, idx) in currentPopupItem?.path || []"
                            :key="`breadcrumb-${idx}-${item.id}`">
                            <!-- 可点击的路径项 -->
                            <span class="cursor-pointer hover:text-blue-600 transition-colors"
                                @click="handleBreadcrumbClick(item)">
                                {{ item.name }}
                            </span>
                            <!-- 分隔符（最后一项后不显示） -->
                            <span v-if="idx < (currentPopupItem?.path?.length || 0) - 1" class="text-gray-400">
                                >
                            </span>
                        </template>
                    </div>
                    <button @click="closeFolderPopup" class="text-gray-600 hover:text-gray-900 transition-colors">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24"
                            stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                d="M6 18L18 6M6 6l12 12" />
                        </svg>
                    </button>
                </div>

                <!-- 2. 弹窗分页指示器（模仿手机桌面） -->
                <div class="popup-page-indicators flex justify-center gap-3 mt-2 mb-3 z-20">
                    <button v-for="(page, idx) in 2" :key="`popup-indicator-${idx}`"
                        class="indicator px-4 py-1.5 text-sm rounded-full transition-all duration-300 font-medium"
                        :class="{
                            'bg-blue-600 text-white': idx === popupCurrentPage,
                            'bg-gray-100 text-gray-600 hover:bg-gray-200': idx !== popupCurrentPage
                        }" @click="popupCurrentPage = idx">
                        {{ idx === 0 ? '容器' : '简介' }}
                    </button>
                    <button
                        class="indicator px-4 py-1.5 text-sm rounded-full transition-all duration-300 font-medium bg-gray-100 text-gray-600 hover:bg-gray-200"
                        @click="handleDetailClick(currentPopupItem)">
                        详情↗
                    </button>
                </div>

                <!-- 3. 弹窗分页内容区（子项页 + 信息页） -->
                <div class="popup-pages-container flex-shrink-0 flex transition-transform duration-300 ease-out h-[60vh] "
                    :style="{ transform: `translateX(-${popupCurrentPage * 100}%)` }">

                    <!-- 页面1：文件夹子项（网格布局，模仿手机桌面） -->
                    <div class="popup-page popup-items-page flex-shrink-0 w-full h-full p-4 overflow-y-auto bg-gray-50">
                        <div class="grid grid-cols-4 sm:grid-cols-5 gap-4">
                            <!-- 子项图标（点击打开子文件夹/物品） -->
                            <div v-for="(child, idx) in currentPopupItem?.children || []" :key="`child-${idx}`"
                                class="app-icon cursor-pointer transition-all duration-200 hover:scale-105 hover:shadow-lg"
                                @click.stop="handleItemClick(child)">
                                <div
                                    class="icon-wrapper w-full aspect-square rounded-lg bg-white flex items-center justify-center shadow-md overflow-hidden">
                                    <img v-if="getCachedImageUrl(child)" :src="getCachedImageUrl(child)"
                                        alt="Child icon" class="w-full h-full object-cover"
                                        @error="handleImageError(child)">
                                    <div v-else class="w-full h-full flex items-center justify-center bg-gray-100">
                                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-gray-400"
                                            fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                                d="M3 7v10a2 2 0 002 2h14a2 2 0 002-2V7m-4 4h-10" />
                                        </svg>
                                    </div>
                                </div>
                                <div class="icon-label w-full text-center text-xs truncate text-gray-800 mt-1">
                                    {{ child.name }}
                                </div>
                            </div>
                        </div>
                        <!-- 子项空状态 -->
                        <div v-if="(currentPopupItem?.children || []).length === 0" class="empty-state mt-8">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-12 w-12 mx-auto mb-2 opacity-70"
                                fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                    d="M3 7v10a2 2 0 002 2h14a2 2 0 002-2V7m-4 4h-10" />
                            </svg>
                            <p>当前文件夹暂无子项</p>
                        </div>
                    </div>

                    <!-- 页面2：物品详情信息（列表布局） -->
                    <div class="popup-page popup-info-page flex-shrink-0 w-full h-full p-4 overflow-y-auto bg-white">
                        <div class="info-list space-y-4">
                            <div class="info-item p-3 border-b border-gray-200">
                                <h4 class="text-xs text-gray-500 mb-1">物品名称</h4>
                                <p class="text-gray-800 font-medium">{{ currentPopupItem?.info?.name || '未知' }}</p>
                            </div>
                            <div class="info-item p-3 border-b border-gray-200">
                                <h4 class="text-xs text-gray-500 mb-1">物品ID</h4>
                                <p class="text-gray-800 font-mono text-sm">{{ currentPopupItem?.info?.id || '未知' }}</p>
                            </div>
                            <div class="info-item p-3 border-b border-gray-200">
                                <h4 class="text-xs text-gray-500 mb-1">描述信息</h4>
                                <p class="text-gray-800">{{ currentPopupItem?.info?.description || '无描述信息' }}</p>
                            </div>
                            <div class="info-item p-3 border-b border-gray-200">
                                <h4 class="text-xs text-gray-500 mb-1">内部物品数量</h4>
                                <p class="text-gray-800">{{ (currentPopupItem?.children || []).length }} 个</p>
                            </div>
                            <div class="info-item p-3 border-b border-gray-200">
                                <h4 class="text-xs text-gray-500 mb-1">图标地址</h4>
                                <p class="text-gray-800 text-sm truncate">{{ currentPopupItem?.info?.imageUrl ||
                                    '使用默认图标' }}
                                </p>
                            </div>
                            <div class="info-item p-3 border-b border-gray-200">
                                <h4 class="text-xs text-gray-500 mb-1">创建时间</h4>
                                <p class="text-gray-800">{{ currentPopupItem?.info?.createTime || '未知' }}</p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue';
const api = useUserApi();

// 原有Props和Emits（新增popup-close事件，可选）
const props = defineProps({
    items: { type: Array, default: () => [] },
    tree: { type: Array, default: () => [] },
    currentLocation: { type: String, default: 'root' }
});
const cacheItems = {};
const scanItems = (children) => {
    if (Array.isArray(children)) {
        children.forEach(e => {
            if (e.id) cacheItems[e.id] = e;
            if (0 < e?.children?.length) scanItems(e.children)
        })
    }
}
scanItems(props.tree);

const emit = defineEmits(['item-click', 'add-item', 'update-tree', 'item-detail', 'popup-close']);

// ########## 原有核心状态（不变） ##########
const locations = ref([]);
const currentPage = ref(0);
const touchStartX = ref(0);
const touchMoveX = ref(0);
const isTouching = ref(false);
const startX = ref(0);
const currentTranslate = ref(0);
const iconSize = 80;
const defaultBackground = 'https://picsum.photos/id/456/1080/1920';
const defaultIcons = [
    'https://picsum.photos/seed/item1/200/200',
    'https://picsum.photos/seed/item2/200/200',
    'https://picsum.photos/seed/item3/200/200',
    'https://picsum.photos/seed/item4/200/200',
];

// ########## 弹窗核心状态（修改为单一弹窗） ##########
const showItemPopup = ref(false); // 弹窗显示开关
const currentPopupItem = ref(null); // 当前弹窗物品
const popupCurrentPage = ref(0); // 弹窗当前分页（0=子项页，1=信息页）

// ########## 原有初始化逻辑（不变） ##########
const initializeLocations = () => {
    const locs = props.tree.map(location => ({
        ...location,
        backgroundImage: location.backgroundImage || defaultBackground,
        children: location.children.map(item => ({
            ...item,
            children: item.children || [], // 确保每个物品都是容器
            createTime: item.createTime || '未知' // 补全默认字段
        }))
    }));
    locations.value = locs;
};
watch(() => props.tree, initializeLocations, { deep: true });
onMounted(() => {
    initializeLocations();
    const handleResize = () => locations.value = [...locations.value];
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
});

// ########## 原有图片缓存逻辑（不变） ##########
const imageCache = ref({});
const loadingUrls = new Set();
const getItemImage = (item) => {
    if (!item.imageUrl) item.imageUrl = defaultIcons[Math.floor(Math.random() * defaultIcons.length)];
    return item.imageUrl;
};
const urlToBase64 = (url) => new Promise((resolve, reject) => {
    const img = new Image();
    img.crossOrigin = 'anonymous';
    img.src = url;
    img.onload = () => {
        const canvas = document.createElement('canvas');
        [canvas.width, canvas.height] = [img.width, img.height];
        canvas.getContext('2d').drawImage(img, 0, 0);
        resolve(canvas.toDataURL('image/jpeg', 0.8));
    };
    img.onerror = (err) => {
        console.error('图片转Base64失败:', url, err);
        reject(err);
    };
});
const loadAndCacheImage = async (url) => {
    if (imageCache.value[url]) return imageCache.value[url];
    if (loadingUrls.has(url)) {
        while (loadingUrls.has(url)) await new Promise(resolve => setTimeout(resolve, 50));
        return imageCache.value[url];
    }
    loadingUrls.add(url);
    try {
        const base64 = await urlToBase64(url);
        imageCache.value[url] = base64;
        return base64;
    } catch (err) {
        const placeholder = 'data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PHBhdGggZmlsbD0iIzAwMCIgZD0iTTExIDhoMnY2aC0ydi02em0xIDExaC0ydi05aDJ2OXptLTggMkg1YTIgMiAwIDAgMC0yIDJ2MTRhMiAyIDAgMCAwIDIgMmgxNmEyIDIgMCAwIDAgMi0yVjVhMiAyIDAgMCAwLTItMkg5em0wIDE0VjdsOCA0LTggNHoiLz48L3N2Zz4=';
        imageCache.value[url] = placeholder;
        return placeholder;
    } finally {
        loadingUrls.delete(url);
    }
};
const getCachedImageUrl = (item) => {
    const url = getItemImage(item);
    if (!imageCache.value[url] && !loadingUrls.has(url)) loadAndCacheImage(url);
    return imageCache.value[url];
};
const handleImageError = (item) => {
    const url = getItemImage(item);
    imageCache.value[url] = 'data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PHBhdGggZmlsbD0iIzAwMCIgZD0iTTExIDhoMnY2aC0ydi02em0xIDExaC0ydi05aDJ2OXptLTggMkg1YTIgMiAwIDAgMC0yIDJ2MTRhMiAyIDAgMCAwIDIgMmgxNmEyIDIgMCAwIDAgMi0yVjVhMiAyIDAgMCAwLTItMkg5em0wIDE0VjdsOC4wMSA0LTguMDEgNHoiLz48L3N2Zz4=';
};

// ########## 原有页面切换逻辑（不变） ##########
const handleTouchStart = (e) => { if (!showItemPopup.value) { isTouching.value = true; touchStartX.value = e.touches[0].clientX; startX.value = touchStartX.value; } };
const handleTouchMove = (e) => { if (!showItemPopup.value && isTouching.value) touchMoveX.value = e.touches[0].clientX; };
const handleTouchEnd = (e) => {
    if (!showItemPopup.value && isTouching.value) {
        const diff = touchMoveX.value - touchStartX.value;
        if (diff > 50 && currentPage.value > 0) currentPage.value--;
        else if (diff < -50 && currentPage.value < locations.value.length - 1) currentPage.value++;
        isTouching.value = false;
    }
};
const handleMouseDown = (e) => { if (!showItemPopup.value) { startX.value = e.clientX; currentTranslate.value = -currentPage.value * 100; isTouching.value = true; } };
const handleMouseMove = (e) => { if (!showItemPopup.value && isTouching.value) currentTranslate.value = -currentPage.value * 100 + (e.clientX - startX.value) / window.innerWidth * 100; };
const handleMouseUp = (e) => {
    if (!showItemPopup.value && isTouching.value) {
        const currentPageValue = Math.round(-currentTranslate.value / 100);
        if (currentPageValue >= 0 && currentPageValue < locations.value.length) currentPage.value = currentPageValue;
        isTouching.value = false;
    }
};

// ########## 原有编辑背景/添加物品逻辑（不变） ##########
const openBackgroundSelector = () => {
    const currentLoc = locations.value[currentPage.value];
    if (!currentLoc) return;
    const newBg = prompt('请输入背景图片URL:', currentLoc.backgroundImage || defaultBackground);
    if (newBg) {
        const updatedLocs = [...locations.value];
        updatedLocs[currentPage.value].backgroundImage = newBg;
        locations.value = updatedLocs;
        emit('update-tree', updatedLocs);
    }
};
const handleAddItem = () => {
    const currentLoc = locations.value[currentPage.value];
    if (currentLoc) emit('add-item', currentLoc.id);
};

// ########## 弹窗核心交互逻辑（修改为单一弹窗） ##########
// 1. 点击图标打开文件夹
const handleItemClick = (item) => {
    console.log(item)
    if (!item?.path) {
        item.path = [{
            "type": "item",
            "id": item.id,
            "name": item.name
        }]
        item.path.nextUpdateTime = Date.now() - 10;
    }
    if (!item?.path?.nextUpdateTime || item?.path?.nextUpdateTime < Date.now()) {
        // 更新路径
        api.items.fullpath(item.id).then(resp => {
            if (resp.error) {
                toast.error(t("items.toast.failed_load_item"));
                return;
            }
            item.path = resp.data;
            // 60秒内不再更新路径
            item.path.nextUpdateTime = Date.now() + (60 * 1000);
        }).catch(err => {
            return;
        });
    }
    // 获取基本信息
    if (!item?.info?.nextUpdateTime || item?.info?.nextUpdateTime < Date.now()) {
        // 更新物品信息
        api.items.get(item.id).then(resp => {
            if (resp.error) {
                toast.error(t("items.toast.failed_load_item"));
                return;
            }
            item.info = resp.data;
            // 120秒内不再更新物品信息
            item.info.nextUpdateTime = Date.now() + (2 * 60 * 1000);
        }).catch(err => {
            return;
        });
    }
    // 获取子节点信息
    if (!item?.children?.nextUpdateTime || item?.children?.nextUpdateTime < Date.now()) {
        // 更新物品信息
        api.items.getAll({
            parentIds: [item.id],
        }).then(resp => {
            if (resp.error) {
                toast.error(t("items.toast.failed_load_items"));
                return;
            }
            // 清理多余的属性
            resp.data.items.forEach(e => {
                for (const k in e) {
                    if (!['id', 'name', 'type', 'children'].includes(k)) delete e[k];
                }
                if (!e.type) e.type = 'item';
                if (!e.children) e.children = [];
            })
            item.children = resp.data.items;
            // 120秒内不再更新物品信息
            item.children.nextUpdateTime = Date.now() + (2 * 60 * 1000);
        }).catch(err => {
            return;
        });
    }
    // 添加缓存
    if (cacheItems[item.id] != item) cacheItems[item.id] = item;
    // 直接设置当前弹窗物品
    currentPopupItem.value = item;
    showItemPopup.value = true;
    // 默认显示子项页
    popupCurrentPage.value = 0 < item?.children?.length ? 0 : 1;
    emit('item-click', item);
};

const handleBreadcrumbClick = (item) => {
    // 先尝试从当前的缓存里面读取
    if (cacheItems[item.id]) item = cacheItems[item.id];
    handleItemClick(item);
}

// 3. 关闭文件夹弹窗
const closeFolderPopup = () => {
    showItemPopup.value = false;
    currentPopupItem.value = null; // 清空当前弹窗物品
    emit('popup-close');
};

// 4. 详情按钮触发事件（外部实现逻辑）
const handleDetailClick = (item) => {
    if (item) emit('item-detail', item);
    if (item.id) location.pathname = '/item/' + item.id;
};

</script>

<style scoped>
/* 原有样式（不变） */
.mobile-desktop-container {
    width: 100%;
    height: calc(100vh - 4rem);
    background-color: #000;
}

.pages-container {
    height: 100%;
    will-change: transform;
}

.desktop-page {
    height: 100%;
}

.page-content {
    padding-bottom: 2rem;
}

.app-icon {
    position: relative;
    user-select: none;
}

.page-content::-webkit-scrollbar,
.popup-items-page::-webkit-scrollbar,
.popup-info-page::-webkit-scrollbar {
    width: 6px;
    height: 6px;
}

.page-content::-webkit-scrollbar-track,
.popup-items-page::-webkit-scrollbar-track,
.popup-info-page::-webkit-scrollbar-track {
    background: rgba(0, 0, 0, 0.1);
    border-radius: 10px;
}

.page-content::-webkit-scrollbar-thumb,
.popup-items-page::-webkit-scrollbar-thumb,
.popup-info-page::-webkit-scrollbar-thumb {
    background: rgba(255, 255, 255, 0.3);
    border-radius: 10px;
}

.page-content::-webkit-scrollbar-thumb:hover,
.popup-items-page::-webkit-scrollbar-thumb:hover,
.popup-info-page::-webkit-scrollbar-thumb:hover {
    background: rgba(255, 255, 255, 0.5);
}

.empty-state {
    color: white;
    text-align: center;
    padding: 2rem;
    opacity: 0.7;
}

/* ########## 弹窗相关样式 ########## */
/* 弹窗背景层 */
.folder-popup-backdrop {
    backdrop-filter: blur(2px);
}

/* 弹窗主体：80%宽度，白色背景 */
.folder-popup {
    border-radius: 16px;
    max-height: 80vh;
    position: relative;
}

/* 弹窗顶部栏 */
.popup-header {
    height: 56px;
    box-sizing: border-box;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

/* 弹窗分页容器 */
.popup-pages-container {
    position: relative;
    will-change: transform;
}

/* 弹窗单个页面 */
.popup-page {
    height: 100%;
    box-sizing: border-box;
}

/* 子项页样式 */
.popup-items-page {
    background-color: #f9f9f9;
}

/* 信息页样式 */
.popup-info-page {
    background-color: white;
}

/* 信息列表样式 */
.info-list {
    margin-top: 16px;
}

.info-item {
    background-color: transparent;
}

.info-item h4 {
    text-transform: uppercase;
    letter-spacing: 0.5px;
}

/* 弹窗分页指示器 */
.popup-page-indicators {
    bottom: 16px !important;
    z-index: 21 !important;
}
</style>