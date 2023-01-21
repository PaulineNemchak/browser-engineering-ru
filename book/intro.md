---
title: Браузеры и веб
type: Вступление
next: history
prev: preface
...

Зачем изучать веб-браузеры? То, как я это вижу, браузеры имеют фундмаентальное значение для веба, современных вычислений и даже для экономики в целом---поэтому следует знать, как они работают. И на самом деле, классные алгоритмы, сложные структуры данных и фундаментальные концепты оживают внутри браузера. Эта книга проведёт вас через процесс построения браузера с нуля. Я надеюсь, вы влюбитесь в веб-браезуры по мере прочтения книги так же, как и я.

Браузер и я
==================


Я --- это говорит Крис --- знаком с интернетом[^theweb] всю свою возрослую жизнь. С тех пор, как я впервые столкнулся с вебом и его предшественниками в начале 90-х, я был восхищён браузерами и концепцией сетевых пользовательских интерфейсов. Когда я [сёрфил][websurfing] по интернету, даже на его ранних стадиях, я чувствовал, что вижу будущее вычислений. В каком-то смысле, веб и я выросли вместе --- например, в 1994 году, когда интернет стал коммерческим, я пошёл в колледж; там я провёл достаточно много времени в поисках информации в интернете, и к моему году выпуска в 1999 году, браузер привёл к известному буму дот-комов. Компания, в которой я сейчас работаю, Google, дитя интернета, была создана в это время. Для меня интернет --- что-то типа технологического товарища, и я никогда не уходил от веба далеко в моих исследованиях или работе.

[^theweb]: В широком смысле, веб это тесно взаимосвязанная сеть (“веб”) [веб-страниц](https://en.wikipedia.org/wiki/Web_page) в интернете. Если вы никогда не создавали веб-страницу, я рекомендую серию статей [Изучение веб-разработки][learn-web] на MDN, особенно гайд [Начало работы с вебом][learn-basics]. Эту книгу будет проще читать,  если вы знакомы с основными технологиями.

[learn-web]: https://developer.mozilla.org/ru/docs/Learn
[learn-basics]: https://developer.mozilla.org/ru/docs/Learn/Getting_started_with_the_web

[websurfing]: https://www.pcmag.com/encyclopedia/term/web-surfing

[^bbs]: Для меня это [BBS](https://ru.wikipedia.org/wiki/BBS) системы через коммутируемые модемные соединения. BBS не так сильно отличаются от браузера, если подумать о них как  об окне в динамический контент, созданный где-то в интернете.

Однаджы в первый год учёбы в колледже я посетил презентацию торгового представителя RedHat. Презентация, конечно же, была направлена на продажу RedHat Linux, называя его «операционной системой будущего» и спекулируя на тему «года компьютеров на Линуксе». Но, когда был задан вопрос о проблемах, с которыми сталкивается RedHat, представитель упомянул не Линукс, а _веб_: он сказал, что кто-то «должен сделать хороший браузер для Линукса».[^netscape-linux] Даже тогда, в самые первые годы веба, браузер уже был необходимым компонентом каждого компьютера. Он даже бросил вызов: «насколько сложно построить лучший браузер»? И правда, насколько сложным это может быть? Почему это так сложно? Этот вопрос надолго засел у меня в голове.[^meantime-linux]

[^netscape-linux]: Netscape Navigator был уже доступен на Линуксе, но он был не настолько быстр и функционален как имплементации на других операционных системах.

[^meantime-linux]: "Лучший браузер на Linux, чем Netscape" появился очень нескоро...

И впрямь, насколько сложно! После семи лет работы над Chrome я знаю ответ на его вопрос: создание браузера одновременно и просто, и невероятно трудно, как намеренно, так и случайно. И куда бы вы ни посмотрели, вы везде видите эволюцию и историю веба, свёрнутые в одну кодовую базу. Но больше всего это весело и бесконечно интересно.

Вот как я влюбился в браузеры. Теперь, позвольте мне рассказать, почему вы тоже влюбитесь.


Веб в истории
==================

Веб — это великий и безумный эксперимент. Сегодня совершенно нормально смотреть видео, читать новости и общаться с друзьями в интернете. Из-за этого может показаться, что веб прост и очевиден, закончен, уже построен. Но веб ни прост, ни очевиден. Это результат экспериментов и исследований, которые берут своё начало почти с самого начала вычислительной техники[^precursors], посвящённый тому, чтобы помочь людям общаться и учиться друг у друга.


[^precursors]: Вебу _также_ нужны были дисплеи с насыщенными цветами, мощные UI-библиотеки, быстрые сети и достаточная мощь ЦП и объём носителей информации. Как это часто бывает с технологиями, веб имел много предшественников, но приобрёл он свою современную форму только когда все части собрались вместе.

Изначально интернет был мировой сетью компьютеров, обычно находящихся в университетах, лабораториях и крупных корпорациях, связанных физическими кабелями и обменивающихся через специфичные для задачи протоколы. На этом был построен ранний веб. Веб-страницы были файлами в определенном формате на определенных компьютерах, и веб-браузеры использовали специальный протокол для запроса этих файлов. URL-адреса веб-страниц называли компьютер и файл, а ранние серверы не делали ничего, кроме как читать файлы с диска. Логическая структура веб-страниц отражала ее физическую структуру.

Многое изменилось. HTML теперь, как правило, динамически собирается на лету[^server-side-rendering] и отправляется по запросу в ваш браузер. Собираемые части сами наполняются динамическим контентом — новостями, содержимым входящих сообщений и рекламными объявлениями, адаптированными к вашим конкретным вкусам. Даже URL-адреса больше не идентифицируют конкретный компьютер — сети доставки содержимого (CDN) направляют URL-адрес на любой из тысяч компьютеров по всему миру. На более высоком уровне большинство веб-страниц обслуживаются не с чьего-либо домашнего компьютера[^self-hosted], а с платформы социальных сетей или платформы облачных вычислений.

[^server-side-rendering]: «Рендеринг на стороне сервера» — это процесс сборки HTML на сервере при загрузке веб-страницы. Рендеринг на стороне сервера часто использует такие веб-технологии, как JavaScript, и даже [headless браузер](https://en.wikipedia.org/wiki/Headless_browser). Еще одно место, которое браузеры берут под контроль!

[^self-hosted]: Люди на самом деле делали это! И когда их сайт становился популярным, ему не хватало пропускной способности или компьютерной мощности, и он становился недоступным.


Многое изменилось, но некоторые вещи остались прежними — основные строительные блоки, представляющие суть интернета:

* Веб — это _информационная сеть_, связанная _гиперссылками_.
* Информация запрашивается с помощью _сетевого протокола HTTP_ и структурируется в формате _HTML документа_.
* Документы идентифицируются по URL-адресам, а _не_ по их содержимому, и могут быть динамическими.
* Веб-страницы могут ссылаться на вспомогательные ресурсы в различных форматах, включая изображения, видео, CSS и JavaScript.
* Пользователь использует _User Agent_, называемый _браузером_, для навигации по сети.
* Все эти строительные блоки являются открытыми, стандартизированными и бесплатными для использования или реиспользования.

С философской точки зрения, тот или иной из этих принципов может показаться вторичным. Можно попытаться провести различие между сетевым взаимодействием и рендерингом. Можно было бы абстрагировать ссылки и сети от конкретного выбора протокола и формата данных. Кто-то может задаться вопросом, необходим ли браузер в теории, или утверждать, что HTTP, URL-адреса и гиперссылки являются единственными действительно важными частями Интернета.

С философской точки зрения, возможно, один или другой из этих принципов является второстепенным. Можно попытаться выделить сетевые и аспекты ренедера в вебе. Можно абстрагировать ссылки и сетевое взаимодействие от конкретного протокола и формата данных. Можно спросить, необходим ли в теории браузер, или поспорить, что HTTP, URL и гиперссылки являются единственно необходимыми частями веб-сайта.

Возможно.[^perhaps] Веб-сайт всё же является экспериментом; основные технологии развиваются и растут. Но интернет не является случайностью; его исходный дизайн отражает истины не только о вычислениях, но и о том, как люди могут связываться и взаимодействовать. Веб не только выжил, но и процветал во время виртуализации хостинга и контента, в частности благодаря элегантности и эффективности этого исходного дизайна.

[^perhaps]: Действительно, какие-то выборы технологий в реализациях могут быть заменены, и, возможно, это произойдет в будущем. Например, JavaScript в конечном итоге может быть заменен другим языком или технологией, HTTP другим протоколом, а HTML его преемником. Конечно, все эти технологии прошли через много версий, но веб остался вебом.

Ключевая вещь, которую нужно понять, это то, что этот великий эксперимент [не закончен](change.md). Суть интернета останется, но изучая веб-браузеры, вы имеете шанс внести свой вклад и определить его будущее.

Настоящие кодовые базы браузеров
======================

Давайте я расскажу вам, каково это — контрибьютить в браузеры. Во время моих первых нескольких месяцев работы над Хромом я наткнулся на код, реализующий тег [`<br>`][br-tag] --- да, старый-добрый тег `<br>`, который я много раз использовал для вставки новых строк в веб-страницы! И оказалось, что реализация почти не занимает кода, ни в Хроме, ни в простом браузере из этой книги.

[br-tag]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/br

Но Chrome в целом --- его функции, скорость, безопасность, надежность --- _вау_. _Тысячи_ человеко-лет были потрачены на это. Есть постоянное девление, что надо делать больше --- добавлять больше функций, улучшать производительность, идти в ногу с «веб-экосистемой» --- для тысяч бизнесов, миллионов разработчиков[^developers] и миллиардов пользователей в интернете.

[^developers]: Обычно я предпочитаю использовать слово «инженер» --- отсюда и название этой книги --- но «разработчик» или «веб-разработчик» гораздо более распространены в интернете. Одна из важных причин в том, что любой может создать веб-страницу --- не только обученные программисты и специалисты в области информатики. «Веб-разработчик» также более инклюзивный термин для дополнительных, критических роли, таких как дизайнеры, авторы, редакторы и фотографы. Веб-разработчик --- это любой, кто делает веб-страницы, независимо от того, как они это делают.

Working on such a codebase can feel daunting. I often find lines of code last
touched 15 years ago by someone I've never met; or even now discover files and
code that I never knew existed; or see lines of code that don’t look necessary,
yet seem to be important. How do I understand that 15-year-old code? Or learn
the purpose of these new files? Can I delete those lines of code, or are they
there for a reason?

What's amazing is that, despite the scale and the pace and the complexity, there
is still room to contribute. Every browser has thousands of unfixed bugs, from
the smallest of mistakes to myriad mix ups and mismatches. Every browser must be
endlessly tuned and optimized to squeeze out that last bit of performance. Every
browser requires painstaking work to continuously refactor the code to reduce
its complexity, often through the careful[^browsers-abstraction-hard]
introduction of modularization and abstraction.

[^browsers-abstraction-hard]: Browsers are so performance-sensitive that, in
many places, merely the introduction of an abstraction---the function call or
branching overhead---has an unacceptable performance cost!

What makes a browser different from most massive code bases is their _urgency_.
Browsers are nearly as old as any “legacy” codebase, but are _not_ legacy, not
abandoned or half-deprecated, not slated for replacement. On the contrary, they
are vital to the world’s economy. Browser engineers must therefore fix and
improve rather than abandon and replace. And since the character of the web
itself is highly decentralized, the use cases met by browsers are to a
significant extent _not determined_ by the companies “owning” or “controlling” a
particular browser. Other people---you---can contribute ideas and proposals and
implementations.

What's amazing is that, despite the scale and the pace and the complexity, there
is still plenty of room to contribute. Every browser today is open-source, which
opens up its implementation to the whole community of web developers. Browsers
evolve like giant R&D projects, where new ideas are constantly being proposed
and tested out. As you would expect, some features fail and some succeed. The
ones that succeed end up in specifications and are implemented by other
browsers. That means that every web browser is open to contributions---whether
fixing bugs or proposing new features or implementing promising optimizations.

And it's worth contributing, because working on web browsers is a lot of fun.

Browser code concepts
=====================

HTML, CSS, HTTP, hyperlinks, and JavaScript---the core of the web---are
approachable enough, and if you've made a web page before you've seen that
programming ability is not required. That's because HTML & CSS are meant to be
black boxes---declarative APIs---where one specifies _what_ outcome to achieve,
and the _browser itself_ is responsible for figuring out the _how_ to achieve
it. Web developers don't, and mostly can't, draw their web page's pixels on
their own.

As a black box, the browser is either magical or frustrating (depending on
whether it is working correctly or not!). But that also makes a browser a pretty
unusual piece of software, with unique challenges, interesting algorithms, and
clever optimizations. Browsers are worth studying for the pure pleasure of it.

[^loss-of-control]: Loss of control is not necessarily specific to the web---much
of computing these days relies on mountains of other peoples’ code.

There are practical reasons for the unusual design of a browser. Yes, developers
lose some control and agency---when pixels are wrong, developers cannot fix them
directly.[^loss-of-control] But they gain the ability to deploy content on the
web without worrying about the details, to make that content instantly available
on almost every computing device in existence, and to keep it accessible in the
future, mostly avoiding the inevitable obsolescence of most software.

What makes that all work is the web browser's implementations of [inversion of
control][inversion], [constraint programming][constraints], and [declarative
programming][declarative]. The web _inverts control_, with an intermediary---the
browser---handling most of the rendering, and the web developer specifying
parameters and content to this intermediary.[^forms] Further, these parameters
usually take the form of _constraints_ over relative sizes and positions instead
of specifying their values directly;[^constraints] the browser solves the
constraints to find those values. The same idea applies for actions: web pages
mostly require _that_ actions take place without specifying _when_ they do. This
_declarative_ style means that from the point of view of a developer, changes
"apply immediately," but under the hood, the browser can be [lazy][lazy] and
delay applying the changes until they become externally visible, either due to
subsequent API calls or because the page has to be displayed to the
user.[^style-calculation]

[inversion]: https://en.wikipedia.org/wiki/Inversion_of_control
[constraints]: https://en.wikipedia.org/wiki/Constraint_programming
[declarative]: https://en.wikipedia.org/wiki/Declarative_programming
[lazy]: https://en.wikipedia.org/wiki/Lazy_evaluation

[^forms]: For example, in HTML there are many built-in [form control
elements][forms] that take care of the various ways the user of a web page can
provide input. The developer need only specify parameters such as button names,
sizing, and look-and-feel, or JavaScript extension points to handle form
submission to the server. The rest of the implementation is taken care of by the
browser.

[forms]: https://developer.mozilla.org/en-US/docs/Learn/Forms/Basic_native_form_controls

[^constraints]: Constraint programming is clearest during web page layout, where
font and window sizes, desired positions and sizes, and the relative arrangement
of widgets is rarely specified directly. A fun question to consider: what does
the browser "optimize for" when computing a layout?

[^style-calculation]: For example, when exactly does the browser compute which
CSS styles apply to which HTML elements, after a web page changes
those styles? The change is visible to all subsequent API calls, so in that
sense it applies "immediately." But it is better for the browser to delay style
re-calculation, avoiding redundant work if styles change twice in quick
succession. Maximally exploiting the opportunities afforded by declarative
programming makes real-world browsers very complex.

To me, browsers are where algorithms _come to life_. A browser contains a
rendering engine more complex and powerful than any computer game; a full
networking stack; clever data structures and parallel programming techniques; a
virtual machine, an interpreted language, and a JIT; a world-class security
sandbox; and a uniquely dynamic system for storing data.

And the truth is---you use the browser all the time, maybe for reading this
book! That makes the algorithms more approachable in a browser than almost
anywhere else: the web is already familiar. After all, it's at the center of
modern computing.

The role of the browser
=======================

Every year the web expands its reach to more and more of what we do with
computers. It now goes far beyond its original use for document-based
information sharing: many people now spend their entire day in a browser, not
using a single other application! Moreover, desktop applications are now often
built and delivered as _web apps_: web pages loaded by a browser but used like
installed applications.[^pwa] Even on mobile devices, apps often embed a browser
to render parts of the application UI.[^hybrid] Perhaps in the future both
desktop and mobile devices will largely be a container for web apps. Already,
browsers are a critical and indispensable part of computing.

[^pwa]: Related to the notion of a web app is a Progressive Web App, which is a
web app that becomes indistinguishable from a native app through [progressive
enhancement][prog-enhance-def].

[^hybrid]: The fraction of such "hybrid" apps that are shown via a "web view" is
    likely increasing over time. In some markets like China, "super-apps" act
    like a mobile web browser for web-view-based games and widgets.
    
So given this centrality, it's worth knowing how the web works. And in fact, the
web is built on simple concepts: open, decentralized, and safe computing; a
declarative document model for describing UIs; hyperlinks; and the User Agent
model.[^useragent] It's the browser that makes these concepts real. The browser
is the User Agent, but also the _mediator_ of the web's interactions and the
_enforcer_ of its rules. The browser is the _implementer_ of the web: Its
sandbox keeps web browsing safe; its algorithms implement the declarative
document model; its UI navigates links. Web pages load fast and react smoothly
only when the browser is hyper-efficient.

[^useragent]: The User Agent concept views a computer, or software within the
    computer, as a trusted assistant and advocate of the human user.

Such lofty goals! How does the browser deliver on them? It's worth knowing. And
the best way to understand that question is to build a web browser.

Browsers and you
================

This book explains how to build a simple browser, one that can---despite its
simplicity---display interesting-looking web pages and support many interesting
behaviors.[^prog-enhance] As you’ll see, it’s surprisingly easy, and it
demonstrates all the core concepts you need to understand a real-world browser.
You'll see what is easy and what is hard; which algorithms are simple, and which
are tricky; what makes a browser fast, and what makes it slow.

[^prog-enhance]: You might relate this to the history of the web and the idea of
[progressive enhancement][prog-enhance-def].

[prog-enhance-def]:
https://en.wikipedia.org/wiki/Progressive_enhancement

The intention is for you to build your own browser as you work through the early
chapters. Once it is up and running, there are endless opportunities to improve
performance or add features. Many of these exercises are features implemented in
real browsers, and I encourage you to try them---adding features is one of the
best parts of browser development!

The book then moves on to details and advanced features that flesh out the
architecture of a real browser’s rendering engine, based on my experiences with
Chrome. After finishing the book, you should be able to dig into the source code
of Chromium, Gecko, or WebKit, and understand it without too much trouble.

I hope the book lets you appreciate a browser's depth, complexity, and power. I
hope the book passes along its beauty---its clever algorithms and data
structures, its co-evolution with the culture and history of computing, its
centrality in our world. But most of all, I hope the book lets you see in
yourself someone building the browser of the future.
