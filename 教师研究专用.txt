// ==UserScript==
// @name         【教师研究专用】在线考试前端检测原理演示（终极完美版）
// @namespace    http://tampermonkey.net/
// @version      3.0
// @description  仅用于教师研究畅课防作弊原理，严禁用于考试作弊
// @author       教学研究专用
// 【关键1：全域名通配，覆盖目标站点的所有子域名】
// @match        *://*.example.com/*
// @include      *://*.example.com/*
// 【关键2：绝对不要加 @noframes，确保iframe正常注入】
// @grant        none
// 【关键3：document-start 确保在页面JS加载前完成劫持】
// @run-at       document-start
// 【关键4：补全MIT许可证，符合油猴平台规则】
// @license      MIT
// ==/UserScript==

(function() {
    'use strict';

    // ==============================================
    // 全局变量定义：所有拦截事件、API列表统一管理
    // ==============================================
    const blockedEvents = [
        'blur', 'focusout', 'visibilitychange',
        'fullscreenchange', 'webkitfullscreenchange', 'mozfullscreenchange', 'MSFullscreenChange',
        'copy', 'paste', 'contextmenu', 'cut', 'selectstart', 'dragstart',
        'beforeunload', 'pagehide', 'unload'
    ];

    const fullscreenRequestMethods = [
        'requestFullscreen', 'webkitRequestFullscreen', 'mozRequestFullScreen', 'msRequestFullscreen'
    ];

    const fullscreenExitMethods = [
        'exitFullscreen', 'webkitExitFullscreen', 'mozCancelFullScreen', 'msExitFullscreen'
    ];

    const fullscreenStateProps = [
        'fullscreenElement', 'webkitFullscreenElement', 'mozFullScreenElement', 'msFullscreenElement',
        'fullscreenEnabled', 'webkitFullscreenEnabled', 'mozFullScreenEnabled', 'msFullscreenEnabled'
    ];

    // ==============================================
    // 核心1：全量劫持事件监听，覆盖所有防作弊事件
    // ==============================================
    const originalAddEventListener = EventTarget.prototype.addEventListener;
    const originalRemoveEventListener = EventTarget.prototype.removeEventListener;

    EventTarget.prototype.addEventListener = function(type, listener, options) {
        if (blockedEvents.includes(type.toLowerCase())) {
            console.log(`[研究日志] 已拦截 ${type} 事件绑定`);
            return;
        }
        return originalAddEventListener.call(this, type, listener, options);
    };

    EventTarget.prototype.removeEventListener = function(type, listener, options) {
        if (blockedEvents.includes(type.toLowerCase())) {
            console.log(`[研究日志] 已拦截 ${type} 事件解绑`);
            return;
        }
        return originalRemoveEventListener.call(this, type, listener, options);
    };

    // 原生函数伪装，规避篡改检测
    EventTarget.prototype.addEventListener.toString = () => 'function addEventListener() { [native code] }';
    EventTarget.prototype.removeEventListener.toString = () => 'function removeEventListener() { [native code] }';

    // ==============================================
    // 核心2：全链路劫持全屏API，伪造全程全屏状态
    // ==============================================
    // 拦截全屏请求
    fullscreenRequestMethods.forEach(method => {
        if (Element.prototype[method]) {
            const originalMethod = Element.prototype[method];
            Element.prototype[method] = function() {
                console.log(`[研究日志] 已拦截 ${method} 全屏请求`);
                return Promise.resolve();
            };
            Element.prototype[method].toString = () => 'function requestFullscreen() { [native code] }';
        }
    });

    // 拦截退出全屏
    fullscreenExitMethods.forEach(method => {
        if (document[method]) {
            const originalMethod = document[method];
            document[method] = function() {
                console.log(`[研究日志] 已拦截 ${method} 退出全屏请求`);
                return Promise.resolve();
            };
            document[method].toString = () => 'function exitFullscreen() { [native code] }';
        }
    });

    // 永久重写全屏状态
    fullscreenStateProps.forEach(prop => {
        Object.defineProperty(document, prop, {
            get: () => prop.includes('Enabled') ? true : document.documentElement,
            configurable: false,
            enumerable: true
        });
    });

    // ==============================================
    // 核心3：永久重写页面可见性/焦点，永远返回激活可见
    // ==============================================
    Object.defineProperty(document, 'hidden', {
        value: false,
        writable: false,
        configurable: false
    });
    Object.defineProperty(document, 'visibilityState', {
        value: 'visible',
        writable: false,
        configurable: false
    });
    Object.defineProperty(window, 'onblur', { value: null, writable: false, configurable: false });
    Object.defineProperty(window, 'onfocus', { value: null, writable: false, configurable: false });
    Object.defineProperty(document, 'hasFocus', {
        value: () => true,
        writable: false,
        configurable: false
    });
    document.hasFocus.toString = () => 'function hasFocus() { [native code] }';

    // 高频焦点保活
    setInterval(() => {
        window.focus();
        document.body.focus();
    }, 500);

    // ==============================================
    // 核心4：SPA单页考试跳转加固，永久锁定劫持状态
    // ==============================================
    const observer = new MutationObserver(() => {
        console.log('[研究日志] 检测到页面跳转，重新加固劫持状态');
        EventTarget.prototype.addEventListener = EventTarget.prototype.addEventListener;
        EventTarget.prototype.removeEventListener = EventTarget.prototype.removeEventListener;
        Element.prototype.requestFullscreen = Element.prototype.requestFullscreen;
        document.exitFullscreen = document.exitFullscreen;
    });
    observer.observe(document, { childList: true, subtree: true });

    // ==============================================
    // 核心5：解除复制粘贴/右键/文本选择限制
    // ==============================================
    const bypassEvents = ['contextmenu', 'copy', 'paste', 'cut', 'selectstart', 'dragstart'];
    bypassEvents.forEach(event => {
        document.addEventListener(event, e => {
            e.stopImmediatePropagation();
            return true;
        }, true);
    });

    // ==============================================
    // 核心6：终极iframe穿透，动态/多层iframe全覆盖
    // ==============================================
    const injectIframeHijack = (iframe) => {
        try {
            if (!iframe.contentWindow || !iframe.contentDocument) return;

            const iframeWindow = iframe.contentWindow;
            const iframeDocument = iframe.contentDocument;

            // 劫持iframe事件监听
            const iframeAddEventListener = iframeWindow.EventTarget.prototype.addEventListener;
            const iframeRemoveEventListener = iframeWindow.EventTarget.prototype.removeEventListener;

            iframeWindow.EventTarget.prototype.addEventListener = function(type, listener, options) {
                if (blockedEvents.includes(type.toLowerCase())) {
                    console.log(`[研究日志] 已拦截iframe内 ${type} 事件绑定`);
                    return;
                }
                return iframeAddEventListener.call(this, type, listener, options);
            };

            iframeWindow.EventTarget.prototype.removeEventListener = function(type, listener, options) {
                if (blockedEvents.includes(type.toLowerCase())) {
                    console.log(`[研究日志] 已拦截iframe内 ${type} 事件解绑`);
                    return;
                }
                return iframeRemoveEventListener.call(this, type, listener, options);
            };

            // 劫持iframe全屏API
            fullscreenRequestMethods.forEach(method => {
                if (iframeWindow.Element.prototype[method]) {
                    const original = iframeWindow.Element.prototype[method];
                    iframeWindow.Element.prototype[method] = function() {
                        console.log(`[研究日志] 已拦截iframe内 ${method} 全屏请求`);
                        return Promise.resolve();
                    };
                }
            });

            // 重写iframe可见性/焦点
            Object.defineProperty(iframeDocument, 'hidden', {
                value: false,
                writable: false,
                configurable: false
            });
            Object.defineProperty(iframeDocument, 'visibilityState', {
                value: 'visible',
                writable: false,
                configurable: false
            });
            Object.defineProperty(iframeWindow, 'hasFocus', {
                value: () => true,
                writable: false,
                configurable: false
            });

            console.log('[研究日志] iframe劫持逻辑已成功注入');
        } catch (e) {
            console.log('[研究日志] 跨域iframe无法注入，已跳过', e);
        }
    };

    // 监听动态创建的iframe
    const iframeObserver = new MutationObserver((mutations) => {
        mutations.forEach(mutation => {
            mutation.addedNodes.forEach(node => {
                if (node.tagName && node.tagName.toLowerCase() === 'iframe') {
                    console.log('[研究日志] 检测到新创建的iframe，开始注入');
                    node.onload = () => injectIframeHijack(node);
                    setTimeout(() => injectIframeHijack(node), 100);
                }
            });
        });
    });
    iframeObserver.observe(document, { childList: true, subtree: true });

    // 页面加载完成后给已存在的iframe注入
    window.addEventListener('DOMContentLoaded', () => {
        document.querySelectorAll('iframe').forEach(iframe => {
            iframe.onload = () => injectIframeHijack(iframe);
            setTimeout(() => injectIframeHijack(iframe), 100);
        });
    });

    console.log('[研究日志] 在线考试前端检测原理演示脚本（终极完美版）已加载，仅用于教学研究');
})();