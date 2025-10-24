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
                    <h1 class="text-3xl font-medium">{{ location.name }}</h1>
                    <div class="flex overflow-x-auto space-x-3 py-2 scrollbar-hide">
                        <div v-for="(item, itemIndex) in moveStagedItems" :key="item.id"
                            class="app-icon cursor-pointer transition-all duration-200 hover:scale-105 hover:shadow-lg"
                            :style="{
                                left: item.position?.left || null,
                                top: item.position?.top || null,
                                width: `${iconSize}px`
                            }" :data-id="item.id" :data-location-id="location.id" @click.stop="handleItemClick(item)">
                            <!-- <div class="icon-wrapper w-80px h-80px aspect-square rounded-lg bg-white/80 flex items-center justify-center shadow-md overflow-hidden"> -->

                            <!-- 右上角控制按钮 -->
                            <button v-if="moveItemsMode"
                                class="absolute -top-2 -right-2 w-6 h-6 rounded-full text-white text-sm font-bold flex items-center justify-center shadow-lg transition-all duration-200 z-20"
                                :class="isItemStaged(item) ? 'bg-red-500 hover:bg-red-600' : 'bg-green-500 hover:bg-green-600'"
                                @click.stop="toggleStagedItem(item)">
                                {{ isItemStaged(item) ? '-' : '+' }}
                            </button>

                            <div class="image-container">
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
                </div>

                <!-- moveItemsMode -->
                <div class="page-content p-4 overflow-y-auto h-[calc(100%-40px)]">
                    <div class="icon-grid grid grid-cols-4 sm:grid-cols-5 md:grid-cols-6 lg:grid-cols-8 gap-4">

                        <div v-if="moveItemsMode"
                            class="app-icon cursor-pointer transition-all duration-200 hover:scale-105 hover:shadow-lg"
                            :style="{
                                left: addItem.position?.left || null,
                                top: addItem.position?.top || null,
                                width: `${iconSize}px`
                            }" :data-id="addItem.id" :data-location-id="location.id"
                            @click.stop="moveStagedItemsApi(void 0, location)">

                            <div class="image-container">
                                <img v-if="getCachedImageUrl(addItem)" :src="getCachedImageUrl(addItem)" alt="Item icon"
                                    class="w-full h-full object-cover" @error="handleImageError(addItem)">
                                <div v-else class="w-full h-full flex items-center justify-center bg-gray-100">
                                    <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8 text-gray-400" fill="none"
                                        viewBox="0 0 24 24" stroke="currentColor">
                                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                            d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
                                    </svg>
                                </div>
                            </div>

                            <div class="icon-label w-full text-center text-xs truncate text-white mt-1 drop-shadow-md">
                                {{ addItem.name }}
                            </div>
                        </div>

                        <div v-for="(item, itemIndex) in location.children" :key="item.id"
                            class="app-icon cursor-pointer transition-all duration-200 hover:scale-105 hover:shadow-lg"
                            :style="{
                                left: item.position?.left || null,
                                top: item.position?.top || null,
                                width: `${iconSize}px`
                            }" :data-id="item.id" :data-location-id="location.id" @click.stop="handleItemClick(item)">
                            <!-- <div class="icon-wrapper w-80px h-80px aspect-square rounded-lg bg-white/80 flex items-center justify-center shadow-md overflow-hidden"> -->

                            <!-- 右上角控制按钮 -->
                            <button v-if="moveItemsMode"
                                class="absolute -top-2 -right-2 w-6 h-6 rounded-full text-white text-sm font-bold flex items-center justify-center shadow-lg transition-all duration-200 z-20"
                                :class="isItemStaged(item) ? 'bg-red-500 hover:bg-red-600' : 'bg-green-500 hover:bg-green-600'"
                                @click.stop="toggleStagedItem(item)">
                                {{ isItemStaged(item) ? '-' : '+' }}
                            </button>

                            <div class="image-container">
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
            class="edit-bg-btn absolute top-2 right-4 p-2 rounded-full bg-black/30 text-white hover:bg-black/50 transition-colors z-10"
            @click="moveItems">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                <path
                    d="M13.586 3.586a2 2 0 112.828 2.828l-.793.793-2.828-2.828.793-.793zM11.379 7.757L3 16.136V19h2.864l8.379-8.379-2.864-2.864z" />
            </svg>
        </button>
        <button
            class="edit-bg-btn absolute top-2 right-16 p-2 rounded-full bg-black/30 text-white hover:bg-black/50 transition-colors z-10"
            @click="editBackgroundUrl">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd"
                    d="M4 3a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V5a2 2 0 00-2-2H4zm12 12H4l4-8 3 6 2-4 3 6z"
                    clip-rule="evenodd" />
            </svg>
        </button>
        <!--         
        <button
            class="add-item-btn absolute bottom-6 left-4 p-2 rounded-full bg-black/30 text-white hover:bg-black/50 transition-colors z-10"
            @click="handleAddItem">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd"
                    d="M10 18a8 8 0 100-16 8 8 0 000 16zm1-11a1 1 0 10-2 0v2H7a1 1 0 100 2h2v2a1 1 0 102 0v-2h2a1 1 0 100-2h-2V7z"
                    clip-rule="evenodd" />
            </svg>
        </button>
        -->

        <!-- ########## 核心修改：组件内分页弹窗（替代原全局弹窗） ########## -->
        <!-- 弹窗容器：absolute定位（组件内），点击外部可关闭 -->
        <div v-if="showItemPopup"
            class="folder-popup-backdrop absolute inset-0 bg-black/50 z-20 flex items-center justify-center p-4"
            @click="closeFolderPopup">
            <!-- 弹窗主体：占比80%，不触发外部关闭事件 -->
            <div class="folder-popup max-w-md bg-white rounded-xl overflow-hidden shadow-2xl" @click.stop>
                <!-- <div class="folder-popup w-[80%] max-w-md bg-white rounded-xl overflow-hidden shadow-2xl" @click.stop> -->

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
                    <button
                        v-if="typeof currentPopupItem?.path?.length == 'number' && -1 < currentPopupItem?.path?.length - 2"
                        class="indicator px-4 py-1.5 text-sm rounded-full transition-all duration-300 font-medium bg-gray-100 text-gray-600 hover:bg-gray-200"
                        @click="handleLestItemClick(currentPopupItem)">
                        ↑上级
                    </button>
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

                            <!-- 添加项 -->
                            <div class="app-icon cursor-pointer transition-all duration-200 hover:scale-105 hover:shadow-lg"
                                v-if="moveItemsMode" @click.stop="moveStagedItemsApi(currentPopupItem)">
                                <div
                                    class="icon-wrapper w-80px h-80px aspect-square rounded-lg bg-white flex items-center justify-center shadow-md overflow-hidden">
                                    <img v-if="getCachedImageUrl(addItem)" :src="getCachedImageUrl(addItem)"
                                        alt="Child icon" class="w-full h-full object-cover object-center"
                                        @error="handleImageError(addItem)">
                                    <div v-else class="w-full h-full flex items-center justify-center bg-gray-100">
                                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-gray-400"
                                            fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                                d="M3 7v10a2 2 0 002 2h14a2 2 0 002-2V7m-4 4h-10" />
                                        </svg>
                                    </div>
                                </div>
                                <div class="icon-label w-full text-center text-xs truncate text-gray-800 mt-1">
                                    {{ addItem.name }}
                                </div>
                            </div>

                            <!-- 子项图标（点击打开子文件夹/物品） -->
                            <div v-for="(child, idx) in currentPopupItem?.children || []" :key="`child-${idx}`"
                                class="app-icon cursor-pointer transition-all duration-200 hover:scale-105 hover:shadow-lg"
                                @click.stop="handleItemClick(child)">

                                <!-- 右上角控制按钮 -->
                                <button v-if="moveItemsMode"
                                    class="absolute -top-2 -right-2 w-6 h-6 rounded-full text-white text-sm font-bold flex items-center justify-center shadow-lg transition-all duration-200 z-20"
                                    :class="isItemStaged(child) ? 'bg-red-500 hover:bg-red-600' : 'bg-green-500 hover:bg-green-600'"
                                    @click.stop="toggleStagedItem(child)">
                                    {{ isItemStaged(child) ? '-' : '+' }}
                                </button>

                                <div
                                    class="icon-wrapper w-80px h-80px aspect-square rounded-lg bg-white flex items-center justify-center shadow-md overflow-hidden">
                                    <img v-if="getCachedImageUrl(child)" :src="getCachedImageUrl(child)"
                                        alt="Child icon" class="w-full h-full object-cover object-center"
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

                    <!-- 页面2：简介，物品详情信息（列表布局） -->
                    <div class="popup-page popup-info-page flex-shrink-0 w-full h-full p-4 overflow-y-auto bg-white">
                        <div class="info-list space-y-4">


                            <!-- 图片列表 - 轮播图显示 -->
                            <div class="info-item p-3 border-b border-gray-200">
                                <h4 class="text-xs text-gray-500 mb-3">图片列表</h4>
                                <!-- http://localhost:3000/api/v1/items/e41586e0-71f1-4189-ac5e-2e76d59418dc/attachments/a78218d8-e1ef-4693-976b-e9b0d79a95ef -->
                                <!-- Swiper 轮播图 -->
                                <div v-if="currentPopupItem?.info?.attachments?.length" class="swiper-container">
                                    <div class="swiper-wrapper">
                                        <div class="swiper-slide"
                                            v-for="attachment in currentPopupItem.info.attachments"
                                            :key="attachment.id">
                                            <img :src="getAttachmentUrl(currentPopupItem, attachment)"
                                                :alt="attachment.name || '图片'"
                                                class="w-full h-48 object-cover rounded-lg"
                                                @click="openFullscreenImage(attachment)"
                                                @error="handleAttachmentError" />
                                        </div>
                                    </div>

                                    <!-- 分页器 -->
                                    <div class="swiper-pagination"></div>

                                    <!-- 导航按钮 -->
                                    <div class="swiper-button-prev"></div>
                                    <div class="swiper-button-next"></div>
                                </div>
                            </div>

                            <div class="info-item p-3 border-b border-gray-200">
                                <h4 class="text-xs text-gray-500 mb-1">物品名称</h4>
                                <p class="text-gray-800 font-medium">{{ currentPopupItem?.info?.name || '未知' }}</p>
                            </div>
                            <div class="info-item p-3 border-b border-gray-200">
                                <h4 class="text-xs text-gray-500 mb-1">物品ID</h4>
                                <p class="text-gray-800 font-mono text-sm">{{ currentPopupItem?.info?.id || '未知' }}
                                </p>
                            </div>
                            <div class="info-item p-3 border-b border-gray-200">
                                <h4 class="text-xs text-gray-500 mb-1">描述信息</h4>
                                <p class="text-gray-800">{{ currentPopupItem?.info?.description || '' }}</p>
                            </div>
                            <div class="info-item p-3 border-b border-gray-200">
                                <h4 class="text-xs text-gray-500 mb-1">内部物品数量</h4>
                                <p class="text-gray-800">{{ (currentPopupItem?.children || []).length }} 个</p>
                            </div>
                            <div class="info-item p-3 border-b border-gray-200">
                                <h4 class="text-xs text-gray-500 mb-1">更新时间时间</h4>
                                <p class="text-gray-800">{{ new
                                    Date(currentPopupItem?.info?.updatedAt).toLocaleString()
                                }}</p>
                            </div>
                            <div class="info-item p-3 border-b border-gray-200">
                                <h4 class="text-xs text-gray-500 mb-1">创建时间</h4>
                                <p class="text-gray-800">{{ new
                                    Date(currentPopupItem?.info?.createdAt).toLocaleString()
                                }}</p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- 全屏图片查看器 -->
        <div v-if="showFullscreenImage"
            class="fullscreen-image-viewer fixed inset-0 bg-black/90 z-50 flex items-center justify-center"
            @click="closeFullscreen">
            <!-- 关闭按钮 -->
            <button
                class="absolute top-4 right-4 p-2 rounded-full bg-black/50 text-white hover:bg-black/80 transition-all duration-300 z-10"
                @click.stop="closeFullscreen">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24"
                    stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                </svg>
            </button>
            <img :src="currentFullscreenImage" class="max-w-full max-h-full object-contain" alt="全屏图片" @click.stop />
        </div>
    </div>
</template>

<script setup>
import { ref, watch, onMounted, nextTick } from 'vue';
import { toast } from "@/components/ui/sonner";
import { useI18n } from "vue-i18n";
const { t } = useI18n();
// 引入 Swiper
import { Swiper } from 'swiper';
import { Pagination, Navigation } from 'swiper/modules';
import 'swiper/css';
import 'swiper/css/pagination';
import 'swiper/css/navigation';
const api = useUserApi();
const noPreviewImg = 'data:image/png;base64,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';
const addItem = {
    id: '-9',
    name: '添加',
    defImageUrl: 'data:image/webp;base64,UklGRqYDAABXRUJQVlA4IJoDAABQKACdASosASwBPp1OokylpCMiJbKICLATiWlu4W/RG/OZaGfCP3aiBHQbkRV8M28g30t0yXsY/UYOhHlBQR5QUEeUFBHlBQR5QUEeUFAtxVnj/TG9xwoHKRA/jT5W7Nq42qLj5x4MoL38J1vK3ZtXG33uTRIId19o6rmiPdVwUEeUFBF7gp5yVj7RvvUBqtcLdknFLhEYSTWSzBFhysUtsJ8NzA0MaHdVspFx7yjqx9o6fMdDAP1/XIvWUURCh1Y+0dVui066DNIn1JSwT+qfNQp1mzkPLgCby8eVlgLiemaPTTxRP8h+pOLBCeE4sYA3mLBvK3ZtV1AcLYL2jqx9o4WwVjkId19o6saQhgIm9IIqckB7Nq46zat/QX0tVAcQb5FUV1BQUEeUFBHcDSKtUeru0YwI8oKCPKCgjygoI8oKCPKCgjygoFgAAP7+DIAC/BSVfZ3g8QKevzJ61CEvxn+Od4GMYK3f0C1+P5oweCIPgpSOmz1QD+c2QR/Td4BAA6LvQDMpFTW772/JzFhMBW3MFkBENReH1Ew7MSnKutPppbbnficbO12hkYl7usCQ25hRYvayi5ytYhGRYDiR0DcJMulINcn2FqaFxOdJ8krO4vhLidU5GvNAAYhOn1XV/6z0VwRbj4lDFRZ+k6NIPxfVWeV+GN24qtMZd29H4TypNxcV6HxYOrn70rzrR7/xZY3k49OjDC/rAwYq7M/F2Rn9piTnCVG6R5AAjsWQJEgtC/IAArYgGk0vlk1aC6Yl94RPwWmwACsbOL11RrZ2yrtERdfAAQV0KNrjczaxlaQuCwAvVwEl/ttiLFJdY66AR75JlhXT6sV/gEt61raCX6+36cqLg53aP0pXcFS0S45PQ54NQbK7AAGbzIDLAJKhHiNEWJhoQ8SKlZQns9DGuBQk/YVx/mUX0cjVieU1bAwrrz9M2fHH+TH2xZtjBNoDYaynpDfyzs6r4LgvYDXEflknYVz+KX+c5xfvWXegepjiXP7t1FxHlhdWMEZwACYEueoLzPgxGFg5MLScBiaWj2Lh9OZ6OzAfSI6S1LC+keuq19GIlzsR+eq9v1feGoL4AGDr12gR8xcYNswfcczdpLrF+aBtJzK2PpCc/C65Z08VD2aYFMiVTSzPkDvwAMi2DcW872EvK7PoVkibz+WGC0hM4By89bm0aDXdHLhZWbBHQX9lcCfQvXF11IgpNpD7tnmxkAAAAAAA'
}

// 原有Props和Emits（新增popup-close事件，可选）
const props = defineProps({
    items: { type: Array, default: () => [] },
    tree: { type: Array, default: () => [] },
    currentLocation: { type: String, default: 'root' }
});
var cacheItems = {};
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
const defaultBackground = new URL(location.href).searchParams.get('bg') || 'https://picsum.photos/id/456/1080/1920';

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
    if (!item.imageUrl) item.imageUrl = noPreviewImg;
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
        const placeholder = noPreviewImg;
        imageCache.value[url] = placeholder;
        return placeholder;
    } finally {
        loadingUrls.delete(url);
    }
};
const getCachedImageUrl = (item) => {
    if (item.defImageUrl) return item.defImageUrl;
    const cacheItem = cacheItems[item.id];
    // 拿到缓存中的预览图url，如果没有就使用默认图url
    const imageId = item.imageId || cacheItem.imageId;
    const url = imageId ? `/api/v1/items/${item.id}/attachments/${imageId}` : getItemImage(item);
    item.imageUrl = url;
    if (!url.startsWith('data:image')) {
        if (!imageCache.value[url] && !loadingUrls.has(url)) loadAndCacheImage(url);
        return imageCache.value[url];
    }
    return url;
};
const handleImageError = (item) => {
    const url = getItemImage(item);
    imageCache.value[url] = noPreviewImg;
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
const editBackgroundUrl = () => {
    const newBg = prompt('请输入默认背景图片URL:', defaultBackground);
    if (newBg) {
        let lu = new URL(location.href);
        lu.searchParams.set('bg', newBg);
        location.href = lu.href;
    }
    // const currentLoc = locations.value[currentPage.value];
    // if (!currentLoc) return;
    // const newBg = prompt('请输入背景图片URL:', currentLoc.backgroundImage || defaultBackground);
    // if (newBg) {
    //     const updatedLocs = [...locations.value];
    //     updatedLocs[currentPage.value].backgroundImage = newBg;
    //     locations.value = updatedLocs;
    //     emit('update-tree', updatedLocs);
    // }
};
const handleAddItem = () => {
    const currentLoc = locations.value[currentPage.value];
    if (currentLoc) emit('add-item', currentLoc.id);
};

// 初始化拿到所有的数据
// var allItem = {};
const getAllItem = (pageSize) => {
    api.items.getAll({
        q: "",
        negateLabels: false,
        onlyWithoutPhoto: false,
        onlyWithPhoto: false,
        includeArchived: false,
        page: 1,
        pageSize: pageSize ?? 1,
        orderBy: 'name',
    }).then(resp => {
        if (resp.error) {
            toast.error(t("items.toast.failed_load_item"));
            return;
        }
        const data = resp.data;
        if (pageSize && data.total <= pageSize) {
            cacheItems = {};
            data.items.forEach(e => {
                // 因为与首次读取的数据不同，导致图片预览图很难重新加载，所以需要手动赋值，并且生成图片地址
                if (cacheItems[e.id]) {
                    let i = cacheItems[e.id];
                    for (const k in e) {
                        i[k] = e[k];
                    }
                    if (e.imageId) {
                        i.imageUrl = `/api/v1/items/${e.id}/attachments/${e.imageId}`;
                    }
                } else {
                    cacheItems[e.id] = e
                }
            });
        } else {
            getAllItem(data.total + 10)
        }
    }).catch(err => {
        return;
    })
}
// 获取所有数据
getAllItem();

// ########## 弹窗核心交互逻辑（修改为单一弹窗） ##########
// 1. 点击图标打开文件夹
const handleItemClick = (item) => {
    if (!item?.path) {
        item.path = [{
            "type": "item",
            "id": item.id,
            "name": item.name
        }]
        item.path.nextUpdateTime = Date.now() - 10;
    }
    if (item?.type != "location") {
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
    }
    // 添加缓存
    if (!cacheItems[item.id]) cacheItems[item.id] = item;
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
// 5. 上级按钮触发事件（外部实现逻辑）
const handleLestItemClick = (item) => {
    if (-1 < item.path.length - 2) {
        handleBreadcrumbClick(item.path[item.path.length - 2])
    }
};

// #regin 轮播图相关
const swiperInstance = ref(null);

// 获取附件 URL
const getAttachmentUrl = (popupItem, attachment) => {
    // 根据你的附件结构返回图片 URL
    return `/api/v1/items/${popupItem.id}/attachments/${attachment.id}`
};

// 处理图片加载错误
const handleAttachmentError = (event) => {
    event.target.src = noPreviewImg;
};

// 初始化轮播图
const initSwiper = () => {
    if (swiperInstance.value) {
        swiperInstance.value.destroy();
    }

    nextTick(() => {
        swiperInstance.value = new Swiper('.swiper-container', {
            modules: [Pagination, Navigation],
            slidesPerView: 1,
            spaceBetween: 10,
            pagination: {
                el: '.swiper-pagination',
                clickable: true,
            },
            navigation: {
                nextEl: '.swiper-button-next',
                prevEl: '.swiper-button-prev',
            },
            loop: true,
            autoplay: {
                delay: 3000,
                disableOnInteraction: false,
            },
            // 响应式配置
            breakpoints: {
                640: {
                    slidesPerView: 1,
                    spaceBetween: 10,
                },
                768: {
                    slidesPerView: 1,
                    spaceBetween: 20,
                }
            }
        });
    });
};

// 监听弹窗显示状态，初始化轮播图
watch(showItemPopup, (newVal) => {
    if (newVal) {
        nextTick(() => {
            setTimeout(initSwiper, 100); // 稍延迟确保 DOM 渲染完成
        });
    }
});
// #endregin

// 全屏图片查看相关状态
const showFullscreenImage = ref(false);
const currentFullscreenImage = ref('');

// 打开全屏图片查看
const openFullscreenImage = (attachment) => {
    currentFullscreenImage.value = getAttachmentUrl(currentPopupItem.value, attachment);
    showFullscreenImage.value = true;
};

// 关闭全屏查看
const closeFullscreen = () => {
    showFullscreenImage.value = false;
};


const moveItemsMode = ref(false);
const moveItems = () => {
    // 移动物品模式
    moveItemsMode.value = !moveItemsMode.value;
    if (!moveItemsMode.value) {
        clearStagedItems();
    }
}

const moveStagedItems = ref([]);
// 检查项目是否在暂存数组中
const isItemStaged = (item) => {
    return moveStagedItems.value.includes(item);
}
const removeItemStaged = (item) => {
    const index = moveStagedItems.value.indexOf(item);
    return index < 0 ? null : moveStagedItems.value.splice(index, 1);
}
const toggleStagedItem = (item) => {
    const index = moveStagedItems.value.indexOf(item);
    if (-1 < index) {
        // 如果已在暂存数组中，则移除
        moveStagedItems.value.splice(index, 1);
    } else {
        // 如果不在暂存数组中，则添加
        moveStagedItems.value.unshift(item);
    }
}
// 清空暂存数组（可选方法）
const clearStagedItems = () => {
    moveStagedItems.value.splice(0, moveStagedItems.value.length);
}
// 获取暂存项目的数量（可选方法）
const getStagedItemsCount = () => {
    return moveStagedItems.value.length;
}
// 获取暂存项目的数量（可选方法）
const moveStagedItemsApi = async (popupItem, locationItem) => {
    // console.log('popupItem, locationItem', popupItem, locationItem)
    // console.log('moveStagedItems...', moveStagedItems)
    let locationTarget = null;
    if (typeof locationItem == 'object' && locationItem.id && locationItem.name) {
        locationTarget = {
            id: locationItem.id,
            name: locationItem.name,
            treeString: locationItem.name
        }
    }
    let itemTarget = null;
    if (popupItem?.id) {
        const { data, error } = await api.items.get(popupItem.id);
        if (!error) {
            itemTarget = {};
            [
                "id",
                "assetId",
                "name",
                "description",
                "quantity",
                "insured",
                "archived",
                "createdAt",
                "updatedAt",
                "purchasePrice",
                "labels",
                "soldTime"
            ].forEach(k => itemTarget[k] = data[k]);
        }
    }

    const items = [...moveStagedItems.value];
    for (const item of items) {
        if (item.id == itemTarget?.id) {
            // 跳过重复的id
            continue;
        }

        // 通过api拿到当前项目的具体内容，然后修改后提交
        const { data, error } = await api.items.get(item.id);
        if (error) {
            toast.error(t("items.toast.failed_load_item"));
        } else {
            const payload = {
                ...data,
                locationId: data.location?.id,
                labelIds: data.labelIds,
                parentId: itemTarget ? itemTarget.id : null,
                assetId: data.assetId,
                purchasePrice: data.purchasePrice ?? 0,
                soldPrice: data.soldPrice ?? 0,
                purchaseTime: data.purchaseTime,
            };
            if (itemTarget) {
                payload.parentId = itemTarget.id;
            }
            if (locationTarget) {
                payload.location = locationTarget;
            }

            const { error } = await api.items.update(item.id, payload);
            if (error) {
                toast.error(t("items.toast.failed_save"));
                return;
            } else {
                // 清理
                removeItemStaged(item);
            }
        }
    }
    // TODO 重新获取当前容器的数据

    toast.success(t("items.toast.item_saved"));
}


</script>

<!-- 原有样式 -->
<style scoped>
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
    color: black;
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

<!-- Swiper 样式定制 -->
<style scoped>
.swiper-container {
    position: relative;
    /* 添加相对定位 */
    width: 100%;
    height: 200px;
    margin: 0 auto;
}

.swiper-slide {
    text-align: center;
    background: #f8f9fa;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 8px;
    overflow: hidden;
}

.swiper-slide img {
    display: block;
    width: 100%;
    height: 100%;
    object-fit: cover;
}

/* 分页器样式 */
:deep(.swiper-pagination) {
    position: absolute;
    /* 绝对定位在容器内 */
    bottom: 10px;
    /* 距离底部10px */
    left: 0;
    width: 100%;
    z-index: 10;
}

:deep(.swiper-pagination-bullet) {
    background: #cbd5e0;
    opacity: 0.7;
}

:deep(.swiper-pagination-bullet-active) {
    background: #4a5568;
}

/* 导航按钮样式 */
:deep(.swiper-button-prev),
:deep(.swiper-button-next) {
    position: absolute;
    /* 绝对定位在容器内 */
    top: 50%;
    /* 垂直居中 */
    transform: translateY(-50%);
    color: #4a5568;
    width: 30px;
    height: 30px;
    z-index: 10;
}

:deep(.swiper-button-prev) {
    left: 10px;
    /* 距离左侧10px */
}

:deep(.swiper-button-next) {
    right: 10px;
    /* 距离右侧10px */
}

:deep(.swiper-button-prev:after),
:deep(.swiper-button-next:after) {
    font-size: 20px;
}

/* 移动端适配 */
@media (max-width: 640px) {
    .swiper-container {
        height: 160px;
    }

    :deep(.swiper-button-prev),
    :deep(.swiper-button-next) {
        display: none;
        /* 移动端隐藏导航按钮 */
    }
}
</style>

<!-- 全屏图片查看器样式 -->
<style scoped>
.fullscreen-image-viewer {
    z-index: 50;
    cursor: zoom-out;
}

/* 添加过渡动画 */
.fullscreen-image-viewer {
    animation: fadeIn 0.3s ease-out;
}

@keyframes fadeIn {
    from {
        opacity: 0;
    }

    to {
        opacity: 1;
    }
}

/* 图片样式 */
.fullscreen-image-viewer img {
    animation: zoomIn 0.3s ease-out;
}

@keyframes zoomIn {
    from {
        transform: scale(0.9);
        opacity: 0;
    }

    to {
        transform: scale(1);
        opacity: 1;
    }
}

/* 关闭按钮样式 */
.fullscreen-image-viewer button {
    transition: all 0.3s ease;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
}

.fullscreen-image-viewer button:hover {
    transform: scale(1.1);
    background-color: rgba(0, 0, 0, 0.8);
}
</style>

<style>
.image-container {
    width: 80px;
    height: 80px;
    overflow: hidden;
    /* 隐藏超出容器的部分 */
    border: 0px solid #ddd;
    /* 可选：容器边框 */
    border-radius: 8px;
    /* 可选：圆角 */
}
</style>