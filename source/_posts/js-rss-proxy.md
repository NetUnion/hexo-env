---
title: 使用 Cloudflare Workers 反代 RSS 订阅
date: 2023-04-29 01:44:27
tags: [JavaScript]
description: 一个使用 JavaScript 编写的 RSS 反代脚本
---

最近因为种种原因，原本可以访问的部分动漫种子站也无法访问了，对于网页端的访问倒还好说，但 BT 软件的 RSS 自动下载功能就失效了，笔者参考了[这篇文章](https://bgm.tv/group/topic/380685)，写了这个 JavaScript 脚本，可以直接部署在 Cloudflare Workers 上，使用时直接在需要反代的网址前加上对应的域名就好了，对除了磁力链接以外的种子下载连接也做了反代。具体的部署就请看前面的链接吧，直接上代码了：

```JavaScript
/*
 * https://github.com/netnr/workers
 *
 * 2019-10-12 - 2022-05-05
 * netnr
 *
 * https://github.com/Rongronggg9/rsstt-img-relay
 *
 * 2021-09-13 - 2022-05-29
 * modified by Rongronggg9
 * 
 * 2023-4-21 
 * modified by papersman
 */

export default {
    async fetch(request, _env) {
        return await handleRequest(request);
    }
}

/**
 * Configurations
 */
const config = {
    // 是否丢弃请求中的 Referer，在目标网站应用防盗链时有用
    dropReferer: true,
    // 黑名单，URL 中含有任何一个关键字都会被阻断
    // blockList: [".m3u8", ".ts", ".acc", ".m4s", "photocall.tv", "googlevideo.com", "liveradio.ie", ".gov"],
    blockList: [],
    typeList: ["image", "video", "audio", "application", "font", "model", "html", "text", "zip"]
};

/**
 * Respond to the request
 * @param {Request} request
 */
async function handleRequest(request) {
    //请求头部、返回对象
    let reqHeaders = new Headers(request.headers),
        outBody, outStatus = 200, outStatusText = 'OK', outCt = null, outHeaders = new Headers({
            "Access-Control-Allow-Origin": "*",
            "Access-Control-Allow-Methods": "GET, POST, PUT, PATCH, DELETE, OPTIONS",
            "Access-Control-Allow-Headers": reqHeaders.get('Access-Control-Allow-Headers') || "Accept, Authorization, Cache-Control, Content-Type, DNT, If-Modified-Since, Keep-Alive, Origin, User-Agent, X-Requested-With, Token, x-access-token"
        });

    try {
        //取域名第一个斜杠后的所有信息为代理链接
        let url = request.url.substr(8);
        url = decodeURIComponent(url.substr(url.indexOf('/') + 1));

        //需要忽略的代理
        if (request.method == "OPTIONS" || url.length < 3 || url.indexOf('.') == -1 || url == "favicon.ico" || url == "robots.txt") {
            //输出提示
            const invalid = !(request.method == "OPTIONS" || url.length === 0)
            outBody = JSON.stringify({
                code: invalid ? 400 : 0,
                usage: 'Add https://rss.sakisakura.moe/ before the RSS URL you want to proxy.',
            });
            outCt = "application/json";
            outStatus = invalid ? 400 : 200;
        }
        //阻断
        else if (blockUrl(url)) {
            outBody = JSON.stringify({
                code: 403,
                msg: 'The keyword "' + config.blockList.join(' , ') + '" was block-listed by the operator of this proxy.'
            });
            outCt = "application/json";
            outStatus = 403;
        }
        else {
            url = fixUrl(url);

            //构建 fetch 参数
            let fp = {
                method: request.method,
                headers: {}
            }
            // 发起 fetch
            let fr = (await fetch(url, fp));
            outCt = fr.headers.get('content-type');

            //保留头部其它信息
            const dropHeaders = ['content-length', 'content-type', 'host'];
            if (config.dropReferer) dropHeaders.push('referer');
            let he = reqHeaders.entries();
            for (let h of he) {
                const key = h[0], value = h[1];
                if (!dropHeaders.includes(key)) {
                    fp.headers[key] = value;
                }
            }
            if (config.dropReferer && url.includes('.sinaimg.cn/')) fp.headers['referer'] = 'https://weibo.com/';

            // 当访问mikanani.me/RSS的时候，将返回的xml中的mikanani.me替换
            if (url.includes('mikanani.me/RSS')) {
                const response = await fetch(url, fp);
                const text = await response.text();
                outBody = text.replace(/mikanani.me\/Download\//g, 'rss.sakisakura.moe/https://mikanani.me/Download/');
                outCt = response.headers.get('content-type');
                outStatus = response.status;
                outStatusText = response.statusText;
            } else if (url.includes('acg.rip/.xml')) {  //当访问acg.rip/.xml的时候，将返回的xml中的acg.rip/t/替换
                const response = await fetch(url, fp);
                const text = await response.text();
                outBody = text.replace(/acg.rip\/t\//g, 'rss.sakisakura.moe/https://acg.rip/t/');
                outCt = response.headers.get('content-type');
                outStatus = response.status;
                outStatusText = response.statusText;
            } else if (url.includes('bangumi.moe/rss')) {  //当访问bangumi.moe/rss的时候，将返回的xml中的bangumi.moe/download替换
                const response = await fetch(url, fp);
                const text = await response.text();
                outBody = text.replace(/bangumi.moe\/download\//g, 'rss.sakisakura.moe/https://bangumi.moe/download/');
                outCt = response.headers.get('content-type');
                outStatus = response.status;
                outStatusText = response.statusText;
            } else if (url.includes('share.dmhy.org/topics/rss/rss.xml') ||
                        url.includes('blog.davidwang.org/atom.xml') || 
                        url.includes('acgnx.se/rss.xml')) { // 不需要修改下载链接的站点
                outBody = fr.body;
                outStatus = fr.status;
                outStatusText = fr.statusText;
            } else if (url.includes('https://mikanani.me/Download/') || 
                        url.includes('https://acg.rip/t/') || 
                        url.includes('https://bangumi.moe/download/')) { // 保证能下载torrent文件
                outBody = fr.body;
                outStatus = fr.status;
                outStatusText = fr.statusText;
            } else { // 火焰啊，吞噬一切！
                outBody = JSON.stringify({
                    code: 403,
                    msg: 'Forbidden'
                });
                outCt = "application/json";
                outStatus = 403;
            };

            if (["POST", "PUT", "PATCH", "DELETE"].indexOf(request.method) >= 0) {
                const ct = (reqHeaders.get('content-type') || "").toLowerCase();
                fp.headers['content-type'] = ct
                if (ct.includes('application/json')) {
                    fp.body = JSON.stringify(await request.json());
                } else if (ct.includes('application/text') || ct.includes('text/html')) {
                    fp.body = await request.text();
                } else if (ct.includes('form')) {
                    fp.body = await request.formData();
                } else {
                    fp.body = await request.blob();
                }
            };

            // 阻断
            if (blockType(outCt)) {
                outBody = JSON.stringify({
                    code: 415,
                    msg: 'The keyword "' + config.typeList.join(' , ') + '" was whitelisted by the operator of this proxy, but got "' + outCt + '".'
                });
                outCt = "application/json";
                outStatus = 415;
            }
        }
    } catch (err) {
        outBody = err.stack;
        outCt = "text/plain;charset=UTF-8";
        outStatus = 500;
        outStatusText = "Internal Server Error";
    }

    //设置类型
    if (outCt && outCt != "") {
        outHeaders.set("content-type", outCt);
    }
    
    let response = new Response(outBody, {
        status: outStatus,
        statusText: outStatusText,
        headers: outHeaders
    })

    return response;
}

/**
 * Fix URL
 * @param {string} url 
 */
function fixUrl(url) {
    if (url.startsWith('http://') || url.startsWith('https://')) {
        return url;
    } else {
        return 'http://' + url;
    }
}

// 阻断 url
function blockUrl(url) {
    url = url.toLowerCase();
    return config.blockList.some(x => url.includes(x));
}
// 阻断 type
function blockType(type) {
    type = type.toLowerCase();
    return !config.typeList.some(x => type.includes(x));
}


```

这边笔者已经搭了一个服务了：<https://rss.sakisakura.moe>，用的免费的 Workers 计划，每天能承载 10w 次请求。而且笔者的这个脚本做了比较严格的阻断，只有蜜柑、动漫花园、ACG.RIP、bangumi.moe 和 AcgnX 的 RSS 页面和对应的下载链接才能代理。

NYAA 因为反爬所以没有，有懂得的 dalao 欢迎修改代码。
