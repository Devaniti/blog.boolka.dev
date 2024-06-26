---
layout: post
title:  "Вступ до GPU програмування з D3D12: Вступ"
date:   2024-03-30 14:40:00 +0200
categories: Introduction_to_GPU_Programming_with_D3D12
lang: ua
---

Це перша частина серії постів про програмування GPU з використанням D3D12. У цій серії ми розглянемо як і навіщо запускати код на GPU, а також деякі суміжні теми.

Зверніть увагу, що деякі частини можуть оновлюватись, поки серія ще виходять.

## Цільова аудиторія

Ця серія призначена для людей, які хочуть познайомитися з програмуванням GPU або вже знайомі з високорівневим графічним API і хочуть ознайомитися з низькорівневим API.

Для розуміння цієї серії вам *не потрібно* знати D3D11, на відміну від багатьох інших ресурсів, доступних в Інтернеті.

## Вимоги

C++ – більшість графічних API надають нативний інтерфейс тільки для C і C++. У цій серії ми будемо використовувати C++, оскільки це спрощує певні речі. Перш ніж ви зможете ефективно використовувати вищезазначені API вам потрібно досягнути певного рівня знань C++.

Математика – ця серія торкнеться як графічних так і обчислень загального призначення на сучасних GPU. Щоб програмувати графіку, вам знадобиться гарне розуміння лінійної алгебри.

Сучасний GPU – ми використаємо певні функції, які доступні лише на досить сучасних GPU. Мінімальний необхідний GPU: NVIDIA GeForce RTX 20 серія, AMD Radeon RX 6000 серія, Intel ARC.

Тепер переходимо до вступу.

## Для чого програмувати на GPU?

GPU - це скорочення від Graphics Processing Unit. Це дуже поширений компонент, який є практично в будь-якому сучасному комп’ютері/смартфоні/ігровій консолі тощо. Швидше за все, саме за допомогою GPU ви зараз читаєте цей текст.

Перші GPU були розроблені для вирішення конкретної проблеми: рендер 3D-графіки. Але з роками, з розвитком реалістичності 3D-графіки, GPU ставали спроможними вирішувати все біль і більш загальні задачі, що призвело до того що сучасні GPU часто використовуются для обчислень, які більше не пов’язані з графікою: наукові симуляції, AI, майнінг криптовалют тощо.

Саме тому варто розібратись чим саме GPU відрізняються від CPU, чому вони використовуються лише в певних випадках і як написати та запускати власний код на GPU.

## Чому саме D3D12?

Щоб взаємодіяти з GPU, вам потрібно використовувати те що зазвичай називають “Графічний API”. Незважаючи на назву, такі API дозволяють запускати будь-яку роботу на GPU, яку GPU взагалі може виконувати, а не лише «Графіку».

На момент написання статті існує багато актуальних графічних API:

- D3D11 – високорівневий API, доступний тільки на Windows
- Vulkan – низькорівневий API, доступний на багатьох платформах
- D3D12 – низькорівневий API, доступний тільки на Windows
- Metal – може бути як високорівневим так і низькорівневим API, доступний тільки на платформах Apple
- WebGL – високорівневий API, призначений для використання в браузері
- WebGPU – низькорівневий API, призначений для використання в браузері

високорівневі API не були обрані, оскільки багато сучасних функцій GPU, такі як Hardware Accelerated Raytracing або Mesh Shaders, вимагають саме низькорівневих API.

API призначені для використання в браузері не були обрані, оскільки вони просто імлементуются за допомогою інших API, а тому є більш обмеженими ніж API на яких воні побудовані.

Серед API які залишились: Vulkan, D3D12 і Metal, усі 3 підходять для наших цілей. D3D12 серед них було обрано оскільки він є трохи простішим порівняно з Vulkan і більш доступним порівняно з Metal. Але багато концепцій та ідей є однаковими для цих API.

## Чим саме GPU відрізняються від CPU?

Початкове призначення GPU — рендер 3D-графіки. Якщо ми дуже спростимо цей процес, то задача рендеру 3D-графіки є наступною - потрібно обчислити, який колір отримує кожен піксель на екрані. Цей розрахунок має 2 важливі властивості:

1. Кожен піксель обчислюється незалежно, між роботою по розрахунку будь-яких 2 пікселів немає залежності.
2. Обчислення, виконані для кожного пікселя, однакові, між ними немає надмірної розбіжності.

І ось звідки надходить потужність GPU. У той час як CPU в основному розроблені для ефективного виконання будь-яких обчислень але з обмеженим паралелізмом, GPU спеціально створені для виконання високопаралельних програм.

Згідно зі [Steam Hardware Survey](https://store.steampowered.com/hwsurvey/Steam-Hardware-Software-Survey-Welcome-to-Steam?platform=pc), на момент написання статті, 98% користувачів мають CPU з 16 або менше ядер. Навіть якщо припустити, що кожен CPU має [Simultaneous Multithreading](https://en.wikipedia.org/wiki/Simultaneous_multithreading), це означає, що практично будь-який сучасний CPU не може виконувати більше 32 обчислень паралельно.

З іншого боку, найпоширенішим GPU на момент написання статті є RTX 3060. Він має 3584 shading units, що означає, що він може паралельно виконувати 3584 обчислень.

Ми бачимо, що середній GPU може обробляти в 100 разів більше завдань паралельно, ніж кращі CPU. Саме тому GPU отримують значно вищу продуктивність для дуже паралельних обчислень, таких як рендер 3D-графіки.

Однак є обчислення, які є непрактичними на GPU. Программи запущені на GPU не можуть вільно виділяти пам’ять самого GPU, за це відповідає CPU. Ви не маєте доступу до диска, мережі чи пристроїв вводу/виводу. GPU не можуть ефективно виконувати алгоритми, які є повністю послідовними та не можуть бути розпаралеленими. Через ці та інші подібні обмеження лише певні типи обчислення можуть ефективно працювати на GPU.
