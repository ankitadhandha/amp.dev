---
"$title": Предварительная загрузка PWA с AMP-страниц
"$order": '1'
description: Одна из рекомендуемых стратегий состоит в том, чтобы реализовать точку входа на ваш сайт в виде AMP-страницы, после чего выполнить фоновую предзагрузку PWA и переключиться на...
formats:
- websites
author: pbakaus
---

Одна из грамотных стратегий заключается в том, чтобы реализовать **точку входа на ваш сайт в виде AMP-страницы**, после чего **выполнить фоновую предзагрузку PWA** и переключиться на него для дальнейшего взаимодействия с пользователем:

- Все оконечные (leaf) страницы с контентом (страницы, которые содержат конкретный контент, а не сводную/обзорную информацию) публикуются в виде AMP-страниц для обеспечения почти моментальной загрузки.
- Такие AMP-страницы используют специальный элемент [`amp-install-serviceworker`](../../../documentation/components/reference/amp-install-serviceworker.md) для предзаполнения кеша и предзагрузки оболочки PWA в то время, пока пользователь просматривает уже загруженный контент.
- Когда пользователь нажимает следующую ссылку на вашем сайте (например, расположенный снизу призыв к действию, имитирующий процесс взаимодействия с приложением), Service Worker перехватывает запрос, берет на себя управление страницей и загружает оболочку PWA.

Читайте дальше, чтобы узнать, зачем и как использовать данный подход к разработке.

## Подключите PWA, чтобы улучшить опыт взаимодействия

### AMP для первоначального привлечения пользователей

AMP — идеальное решение для так называемых **оконечных (leaf) страниц**. Это контентные страницы, которые ваши пользователи органически находят с помощью поисковых систем, полученных от друзей ссылок, а также ссылок с других сайтов. Благодаря [специализированному механизму предварительного рендеринга](../../../about/how-amp-works.html) AMP-страницы загружаются невероятно быстро, что выражается в значительном снижении доли отключающихся пользователей (новейшее [исследование DoubleClick](https://www.doubleclickbygoogle.com/articles/mobile-speed-matters/) показывает, что **более чем 53% всех пользователей отключаются через 3 секунды ожидания**).

### PWA для насыщенной интерактивности и вовлечения

Прогрессивные веб-приложения, с другой стороны, обеспечивают гораздо большую интерактивность и вовлечение, но не имеют *свойств мгновенной первой загрузки*, присущих AMP-страницам. В их основе лежит технология Service Worker, представляющая собой клиентский прокси, который позволяет кешировать все виды ресурсов для ваших страниц, однако Service Worker активируется только *после* первой загрузки.

{{ image('/static/img/docs/pwamp_comparison.png', 977, 549, align='', caption='The pros and cons of AMP vs. PWA.') }}

## Предзагрузка вашего PWA с помощью `amp-install-serviceworker`

AMP позволяет устанавливать Service Worker вашего прогрессивного веб-приложения в AMP-страницу — даже если эта AMP-страница выдается из AMP-кеша! Если все сделано правильно, ссылка перехода на PWA (с одной из ваших AMP-страниц) будет загружаться практически мгновенно, подобно первому переходу на AMP-страницу.

[tip type="tip"] **СОВЕТ.** Если вы еще не знакомы с Service Worker, рекомендуем [курс Джейка Арчибальда на Udacity](https://www.udacity.com/course/offline-web-applications--ud899). [/tip]

Сначала установите Service Worker на все свои AMP-страницы с помощью [`amp-install-serviceworker`](../../../documentation/components/reference/amp-install-serviceworker.md), прежде всего включив компонент посредством его скрипта в тег `<head>` вашей страницы:

[sourcecode:html]
<script async custom-element="amp-install-serviceworker"
  src="https://cdn.ampproject.org/v0/amp-install-serviceworker-0.1.js"></script>
[/sourcecode]

Затем добавьте следующую строку где угодно внутри тега `<body>` (измените, как необходимо, чтобы указать на ваш Service Worker):

[sourcecode:html]
<amp-install-serviceworker
      src="https://www.your-domain.com/serviceworker.js"
      layout="nodisplay">
</amp-install-serviceworker>
[/sourcecode]

И наконец, на этапе установки Service Worker кешируйте все ресурсы, которые понадобятся PWA:

[sourcecode:javascript]
var CACHE_NAME = 'my-site-cache-v1';
var urlsToCache = [
  '/',
  '/styles/main.css',
  '/script/main.js'
];

self.addEventListener('install', function(event) {
  // Perform install steps
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(function(cache) {
        console.log('Opened cache');
        return cache.addAll(urlsToCache);
      })
  );
});
[/sourcecode]

[tip type="tip"] **СОВЕТ.** Существуют более простые способы работы с Service Worker. Попробуйте [вспомогательные библиотеки Service Worker](https://github.com/GoogleChrome/sw-helpers). [/tip]

## Сделайте так, чтобы все ссылки на AMP-странице вели в PWA

Скорее всего, большинство ссылок на ваших AMP-страницах ведут на другие страницы с контентом. Есть две стратегии, позволяющие гарантировать, что последующие щелчки по ссылкам приведут к «модернизации» среды до прогрессивного веб-приложения, — в [зависимости от того, как вы используете AMP](../../../documentation/guides-and-tutorials/optimize-measure/discovery.md):

### 1. Если вы совместно используете свои канонические страницы с AMP-страницами

В этом сценарии у вас есть канонический (не-AMP) сайт и вы создаете AMP-страницы, которые связаны ссылками с каноническими страницами. В настоящее время это наиболее распространенный способ использования AMP, и это означает, что ссылки на ваших AMP-страницах, скорее всего, будут вести на каноническую версию вашего сайта. **Хорошие новости: если ваш канонический сайт — это ваше PWA, ничего делать больше не нужно**.

### 2. Если ваш канонический сайт — AMP-ресурс

В этом случае ваши канонические страницы — *это и есть* ваши AMP-страницы: вы создаете весь свой сайт с помощью AMP и просто используете AMP в качестве библиотеки (любопытный факт: сайт, на котором вы читаете этот материал, создан именно таким способом). **В этом случае большинство ссылок на ваших AMP-страницах будут вести на другие AMP-страницы.**

Теперь вы можете развернуть PWA по отдельному пути (например, `your-domain.com/pwa`) и использовать уже запущенный Service Worker для **перехвата навигации браузера при каждом нажатии на ссылку на AMP-странице**:

[sourcecode:javascript]
self.addEventListener('fetch', event => {
    if (event.request.mode === 'navigate') {
      event.respondWith(fetch('/pwa'));

      // Immediately start downloading the actual resource.
      fetch(event.request.url);
    }

});
[/sourcecode]

Особенно интересно в этой технике то, что теперь вы используете т. н. «стратегию прогрессивного улучшения» для перехода от AMP к PWA. Однако это также означает, что по умолчанию браузеры, которые еще не поддерживают Service Worker, будут переходить с AMP на AMP, никогда не переходя к PWA.

AMP решает эту проблему с помощью [подстановки URL оболочки](../../../documentation/components/reference/amp-install-serviceworker.md#shell-url-rewrite). Добавляя в тег [`amp-install-serviceworker`](../../../documentation/components/reference/amp-install-serviceworker.md) шаблон URL для резервного варианта действий, вы даете AMP следующую инструкцию: если поддержка Service Worker не обнаружена, заменять все совпадающие ссылки на данной странице ссылками, ведущими на унаследованные URL оболочки:

[sourcecode:html]
<amp-install-serviceworker
      src="https://www.your-domain.com/serviceworker.js"
      layout="nodisplay"
      data-no-service-worker-fallback-url-match=".*"
      data-no-service-worker-fallback-shell-url="https://www.your-domain.com/pwa">
</amp-install-serviceworker>
[/sourcecode]

При наличии этих атрибутов все последующие клики на AMP-страницах будут выполнять переадресацию в ваше PWA, вне зависимости от присутствия Service Worker.

[tip type="read-on"] **ДОПОЛНИТЕЛЬНАЯ ИНФОРМАЦИЯ.** Вы сделали уже так много — почему бы не использовать существующие AMP-страницы для создания своего PWA? [Подробнее здесь](amp-in-pwa.md). [/tip]