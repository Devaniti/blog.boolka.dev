﻿---
layout: post
title:  "Вступ до GPU програмування з D3D12: Частина 4 - Основи API"
date:   2025-04-02 3:00:00 +0900
categories: Introduction_to_GPU_Programming_with_D3D12
lang: ua
---

[Попередній пост](Introduction-to-GPU-Programming-with-D3D12-Part-3-History.html)

Перш ніж почати використовувати D3D12 API, вам потрібно зрозуміти як вони структуровані. Є декілька спільних рис між релевантними API про які вам варто знати.

## Компоненти API

Є три окремі API які ми покриємо: DXGI, D3D12 і DirectStorage. Є також інші суміжні API яких ми не будемо торкатись у цій серії: DirectSR, DirectML.

### DXGI

DXGI (DirectX Graphics Infrastructure) це API спільна для усіх версій DirectX починаючи з [10ї](https://www.youtube.com/watch?v=G_tgT9ytmPc). Включає наступну функціональність:

- Перелічення та запит інформації про GPU
- Перелічення та запит інформації про екрани
- Налаштування виводу на екран
- Налаштування розширення екрану
- тощо

Використання цього API технічно необов'язкове, але тільки у випадку коли вам не треба перелічувати GPU або виводити щось на екран.

### D3D12

D3D12 це скорочення від Direct3D 12. Незважаючи на назву, його можна використовувати для двовимірної графіки та обчислень загального призначення. Це основний API з яким ви будете працювати. Включає наступну функціональність:

- Створення GPU ресурсів (буфери, текстури, тощо)
- Компіляція шейдерів (Vertex, Pixel, Compute, тощо)
- Запис команд для роботи GPU
- Відправка роботи на ГПУ
- Синхронізація CPU і GPU
- тощо

### DirectStorage

DirectStorage — це API, який дозволяє завантажувати дані з диска. Незважаючи на поширену хибну думку, для DirectStorage не потрібні NVMe SSD або будь-які особливі типи накопичувачів. Він підтримує *все*, з чого можна прочитати файл: SSD, HDD, мережеве сховище, оптичні диски, *дискети*, тощо. У цій серії ми називатимемо його "DStorage". Включає наступну функціональність:

- Читання з диска в CPU пам'ять
- Читання з диска в GPU пам'ять
- Декомпресія на льоту
- Синхронізація диска з CPU або GPU
- тощо

Його використання абсолютно необов'язкове, оскільки ви можете читати з диску використовуючи будь-який інший API. Однак є кілька переваг:

- Використовувати DStorage простіше аніж виконувати ці операції вручну
- Він надає опцію встановлення пріоритету запиту на читання
- Він має спеціальну оптимізацію для NVMe SSD, що дозволяє зменшити накладні витрати на введення-виведення
- Він включає високопродуктивну декомпресію на GPU
- Ви зможете скористатися майбутніми оновленнями DStorage

## nano-COM

COM (Component Object Model) - це стандарт програмування інтерфейсів Microsoft. Усі вищезазначені API використовують спрощену версію COM, яка називається "nano-COM". Вам не потрібен попередній досвід роботи з COM, щоб почати використовувати nano-COM.

Але, якщо у вас вже є попередній досвід роботи з COM, або ви наткнетесь на матеріали про COM у інтернеті, то ось список важливих відмінностей між ними:

- nano-COM не потребує ініціалізації.
  - Звичайний COM вимагає ініціалізацію за допомогою `CoInitialize`\[`Ex`\]/`CoUninitialize`\[`Ex`\].
- nano-COM завжди багатопотоковий.
  - Звичайний COM можна ініціалізувати як у однопотоковому так і у багатопотоковому режимі.
  - Однак це не означає, що всі об'єкти API є багатопотоковими.
- Ви не можете створити nano-COM об'єкти самостійно.
  - Звичайний COM дозволяє створювати об'єкти за допомогою `CoCreateInstance`.
  - У nano-COM об'єкти створюються або за допомогою вільних функцій, доступних в API, або за допомогою методів інших об'єктів.

### HRESULT

HRESULT це статус код який відображає результат методу. Більшість методів у nano-COM повертають цей код.

Є 2 корисні макроси для перевірки коду на успіх чи помилку: `SUCCEEDED(x)` і `FAILED(x)`. Ось приклад перевірки на помилки за допомогою макросу `FAILED`:

```cpp
HRESULT hr = obj->Method();
if (FAILED(hr))
{
    // Тут обробка помилки
}
```

Також поширеною практикою є створення макросу який буде мати стандартну обробку помилок, і використовувати його в усіх методах у яких ви не очікуєте помилок:

```
#define ASSERT_D3D12_SUCCEEDED(expression) \
    { \
        HRESULT hr = expression; \
        assert(SUCCEEDED(hr)); \
    }
ASSERT_D3D12_SUCCEEDED(obj->Method());
```

### Інтерфейси

Основним методом розкриття функціональності програмі через nano-COM є інтерфейси. Коли ви створюєте об'єкт, ви отримуєте вказівник на інтерфейс. Ви ніколи не зможете розіменувати ці вказівники, оскільки всі ці інтерфейси є абстрактними класами.

У кожного nano-COM інтерфейсу є наступні властивості:

- Його ім'я починається з `I`, Що означає Interface (інтерфейс)
- Він наслідується від `IUnknown` інтерфейсу
- У нього є унікальний Interface ID (IID)
- Він використовує підрахунок посилань для контролю тривалості життя (використовуючи інтерфейс `IUnknown`)
- Він має функцію динамічного приведення типів (використовуючи інтерфейс `IUnknown`)

#### IID

Interface ID (IID) це унікальний ідентифікатор інтерфейсу. Його основне призначення це динамічна передача типу інтерфейсу. Щоб створити об'єкт, треба викликати метод створення з IID бажаного типу. Таким чином один метод може повертати вказівники на різні інтерфейси. Щоб виконати динамічне приведення типу, ви також використовуєте IID для вказання бажаного типу.

Ви можете отримати IID декількома способами:

- Використавши константу. Для інтерфейсу `ISampleInterface`, такий константний IID буде мати ім’я `IID_ISampleInterface`.
  - Цей спосіб може вимагати лінкування додаткових бібліотек.
- Використавши MSVC-специфічний оператор [`__uuidof`](https://learn.microsoft.com/en-us/cpp/cpp/uuidof-operator?view=msvc-170).
  - Він підтримує наступні параметри
    - Тип інтерфейсу - наприклад `__uuidof(ISampleInterface)`
    - Вказівник на об'єкт - наприклад `__uuidof(sampleObjectPointer)`
    - Посилання на об'єкт - наприклад `__uuidof(*sampleObjectPointer)`
  - Подібно до `sizeof`, оператор `__uuidof` не обчислює переданий вираз, а тільки використовує його тип. Таким чином, незважаючи на те, що `__uuidof(*sampleObjectPointer)` виглядає як розіменування вказівника інтерфейсу, його не буде обчислено, тому в цьому випадку, його розіменування безпечне.

Ось приклад того, як ми можемо використовувати IID:

```cpp
IDXGIFactory7* factory = nullptr; // змінна що отримає об'єкт
HRESULT hr = CreateDXGIFactory2(0, IID_IDXGIFactory7, (void**)&factory);
```

Цей код створює об'єкт `IDXGIFactory7` за допомогою функції `CreateDXGIFactory2`. Для того щоб вказати що ми хочемо саме `IDXGIFactory7`, ми передали `IID_IDXGIFactory7`. Ми також могли використати усі інші способи отримати IID:

```cpp
__uuidof(IDXGIFactory7)
__uuidof(factory)
__uuidof(*factory)
```

Аргумент після IID це адреса змінної на вказівник інтерфейсу. Пара аргументів - IID і адреса змінної що отримує вказівник це доволі поширена схема у методах nano-COM, так що було б зручно якщо б ми могли спростити передачу вказівника і його типу. На щастя є макрос який саме це і робить - `IID_PPV_ARGS`. Він заміняється на IID визначений з типу вказівника разом з його адресою. Використовуючи `IID_PPV_ARGS` ми можемо переписати приклад наступним чином:

```cpp
IDXGIFactory7* factory = nullptr; // змінна що отримає об'єкт
HRESULT hr = CreateDXGIFactory2(0, IID_PPV_ARGS(&factory));
```

#### IUnknown

[`IUnknown`](https://learn.microsoft.com/en-us/windows/win32/api/unknwn/nn-unknwn-iunknown) це базовий клас для усіх nano-COM інтерфейсів.

Він має 3 методи:

- [`ULONG AddRef()`](https://learn.microsoft.com/en-us/windows/win32/api/unknwn/nf-unknwn-iunknown-addref)
- [`ULONG Release()`](https://learn.microsoft.com/en-us/windows/win32/api/unknwn/nf-unknwn-iunknown-release)
- [`HRESULT QueryInterface(REFIID riid, void **ppvObject)`](https://learn.microsoft.com/en-us/windows/win32/api/unknwn/nf-unknwn-iunknown-queryinterface(refiid_void))

Перші 2 відповідають за підрахунок посилань, останній дозволяє виконувати динамічне приведення типів.

#### Підрахунок посилань

Коли об'єкт створюється, він має кількість посилань рівну 1. Ви можете збільшити її за допомогою методу `AddRef`, і зменшити — за допомогою методу `Release`. Коли кількість посилань досягає 0, об'єкт знищується, і ви більше не можете його використовувати.

Ось як це може виглядати у коді. Ви хочете створити об'єкт `IDXGIFactory7` за допомогою функції `CreateDXGIFactory2`, використати його а потім знищити:

```cpp
IDXGIFactory7* factory = nullptr; // змінна що отримає об'єкт
HRESULT hr = CreateDXGIFactory2(0, IID_PPV_ARGS(&factory));
if (SUCCEEDED(hr))
{
    // Якщо створення було успішне, factory тепер ма є кількість посилань рівну 1
    // Ми можемо використовувати factory тут
    ...
    // Нам більше не потрібен об'єкт, і ми хочемо його знищити
    factory->Release(); // Цей виклик зменшить кількість посилань до 0, і знищить об'єкт
    factory = nullptr; // Для безпеки переписуємо вказівник, адже ми вже не можемо використовувати старий об'єкт
}
```

Ось ще один приклад: у вашій програмі є 2 окремі модулі: А та Б. A створив об'єкт і передав його Б. Б зберіг вказівник на об'єкт для майбутнього використання та викликав `AddRef`. Коли А більше не потребує об'єкт, він викликає `Release`. Те ж саме зробить Б. Не має значення у якому порядку А і Б припиняють використовувати об'єкт, він буде знищений лише після того, коли обидва викличуть `Release`. Ось як це можна реалізувати:

```cpp
// Перевірка помилок пропущена щоб приклад був коротшим
// Модуль А
IDXGIFactory7* g_FactoryA = nullptr;

void Create()
{
    HRESULT hr = CreateDXGIFactory2(0, IID_PPV_ARGS(&g_FactoryA));
    B.ImportFactory(g_FactoryA);
}

void Use()
{
    // Тут використовуємо g_FactoryA
}

void Release()
{
    g_FactoryA->Release();
    g_FactoryA = nullptr;
}

// Модуль Б
IDXGIFactory7* g_FactoryB = nullptr;

void ImportFactory(IDXGIFactory7* factory)
{
    g_FactoryB = factory;
    g_FactoryB->AddRef();
}

void Use()
{
    // Тут використовуємо g_FactoryB
}

void Release()
{
    g_FactoryB->Release();
    g_FactoryB = nullptr;
}
```

Методи `AddRef` і `Release` повертають нову кількість посилань. Однак це значення має використовуватися лише для тестування. Можна використовувати це значення, наприклад у `assert`, щоб впевнитись що виклик `Release` зменшив кількість посилань до 0 і знищив об'єкт. Але цю кількість посилань не варто використовувати для логіки програми.

У D3D12 також існує внутрішній підрахунок посилань, окремий від зовнішнього підрахунку посилань, яким ви керуєте за допомогою `AddRef` і `Release`. У деяких випадках один D3D12 об'єкт може неявно зберігти посилання на інший, і в таких випадках він використовує цей окремий внутрішній підрахунок посилань. Об'єкт буде знищено лише тоді, коли кількість і внутрішніх і зовнішніх посилянь досягне 0. Зазвичай вам не потрібно про це турбуватися, оскільки ви відповідаєте лише за зовнішні посилання. Однак за допомогою деяких інструментів налагодження ви зможете побачити об'єкти які мають 0 зовнішніх посилань, але ще не знищені.

#### Динамічне приведення типів

Якщо у вас є об'єкт для певного інтерфейсу, метод `QueryInterface` дозволяє вам отримати інший інтерфейс з нього. По суті, це динамічне приведення типыв. Цей метод має 2 параметри: IID бажаного інтерфейсу та адресу для збереження результату, тому це ще один кандидат для використання `IID_PPV_ARGS`. Якщо цей метод успішно виконається, кількість посилань також збільшиться. Логіка полягає в тому, що у вас є 2 вказівники на один об'єкт, і ви викличите `Relase` у кожному з них.

Ось приклад. У вас є наступні інтерфейси:

<img src="{{site.baseurl}}/diagrams/DynamicCast.mmd.svg" height="300" alt="Приклад Діаграми Класів">

Припустимо, у вас є об'єкт `ID3D12Device`, і ви хочете отримати з нього інтерфейс `ID3D12DebugDevice`. Ось як ми можемо використати `QueryInterface`:

```cpp
ID3D12Device* device = /* тут дійсний вказівник */;
ID3D12DebugDevice* debugDevice = nullptr; // змінна що отримає вказівник після приведення типу
if (SUCCEEDED(device->QueryInterface(IID_PPV_ARGS(&debugDevice))))
{
    // Приведення типу успішне
    // Тут можна використовувати об'єкт debugDevice
    ...
}
```

Однак у цьому фрагменті коду є помилка. **У разі успіху `QueryInterface` збільшує кількість посилань**. Це означає, що тепер ми повинні викликате `Release` після успішного `QueryInterface`. Як ви щойно бачили, про це досить легко забути. На щастя, є підхід, який усуває цю проблему.

### ComPtr

[`Microsoft::WRL::ComPtr`](https://learn.microsoft.com/en-us/cpp/cppcx/wrl/comptr-class?view=msvc-170) це клас розумного вказівника, який за вас викликає `AddRef` і `Release`. Це та сама ідея, що й `std::shared_ptr`, з різницею, що він використовує вбудований підрахунок посилань об'єкта nano-COM замість того щоб створювати окремий.

#### Додавання ComPtr до вашого проекту

`ComPtr` це клас у просторі імен `Microsoft::WRL`. Ви можете використати `using Microsoft::WRL::ComPtr;`, щоб не доводилось щоразу вказувати простір імен. Оскільки таке використання `using` робить лише `ComPtr` видимим за межами свого простору імен, вона не має такого недоліку як у `using namespace`, тому її безпечно використовувати глобально.

Оскільки `ComPtr` не є частиною вищезазначених API, вам потрібно включити його окремо. Він доступний у файлі `wrl/client.h`.

Отже, щоб додати його у свій проект, ви можете використати додати наступний фрагмент коду:

```cpp
#include <wrl/client.h>
using Microsoft::WRL::ComPtr;
```

#### Методи

`ComPtr` має багато методів, ми розглянемо найважливіші з них:

- `ComPtr::ComPtr` (конструктор)
 - Конструктор за замовчуванням - Ініціалізує вказівник значенням `nullptr`.
 - Конструктор копіювання - копіює вказівник і, якщо він не `nullptr`, збільшує кількість посилань.
 - Конструктор переміщення - копіює вказівник, потім встановлює вказівник в іншому об'єкті на nullptr. Не змінює кількість посилань.
 - З сирого вказівника - зберігає вказівник і, якщо він не `nullptr`, *збільшує кількість посилань*.
- `ComPtr::~ComPtr` (деструктор) - якщо збережений вказівник не є `nullptr`, зменшує кількість посилань.
- `Reset` - якщо збережений вказівник не є `nullptr`, зменшує кількість посилань і присвоює йому `nullptr`
- `operator&`/`ReleaseAndGetAddressOf` - виконує `Reset`, потім повертає адресу змінної, яка зберігає вказівник. *Це означає, що після виклику цього методу вказівник усередині завжди буде `nullptr`.*
- `Attach` - виконує `Reset` і зберігає переданий вказівник, *без збільшення кількості посилань*.
- `operator->`/`Get` - Повертає сирий вказівник. `operator->` дозволяє напряму викликати методи інтерфейсу через `ComPtr`.
- `As` - виконує `QueryInterface` зі збереження результату в іншому `ComPtr`. Повертає значення (`HRESULT`), яке повертає `QueryInterface`.

Ви можете переглянути повний список методів у [документації ComPtr](https://learn.microsoft.com/en-us/cpp/cppcx/wrl/comptr-class?view=msvc-170#public-constructors).

#### Приклад

Давайте перепишемо попередній приклад використовуючи `ComPtr`:

```cpp
ComPtr<ID3D12Device> device = /* тут дійсний вказівник */;
ComPtr<ID3D12DebugDevice> debugDevice; // Ініціалізується значенням nullptr
// Метод "As" за типом вказівника debugDevice знаходить правильний IID, і повертає результат QueryInterface
if (SUCCEEDED(device.As(&debugDevice))) 
{
    // Приведення типу успішне
    // Тут можна використовувати об'єкт debugDevice
    ...
    // Не потрібно вручну зменшувати кількість посилань
}
```

З `ComPtr` ми не можемо забути викликати `Release`, і простіше використовувати `QueryInterface`. Зауважте, що ви все ще можете використовувати макрос `IID_PPV_ARGS` з `ComPtr` так, ніби це був сирий вказівник.

Однак є ще кілька речей, на які слід звернути увагу:

- Якщо вам потрібно отримати сирий вказівник, у випадку `ComPtr` вам потрібно буде викликати метод `Get`. Він знадобиться для певних методів nano-COM, які очікують сирі вказівники на інші об'єкти nano-COM.
- В ідеалі варто створювати об'єкти nano-COM безпосередньо в `ComPtr`. Але якщо у вас уже є сирий вказівник, який ви хочете перетворити на `ComPtr`, вам, імовірно, потрібно буде використати метод `Attach`, щоб запобігти збільшенню кількості посилань.
- `operator&` звільняє збережений об'єкт перед поверненням адреси змінної вказівник. Це робить його зручним для створення нового об'єкта та збереження вказівник в `ComPtr`. Але ви повинні бути обережними з цим оператором, щоб переконатися, що ви викликаєте його лише тоді, коли ви не збираєтеся зберігати вказівник усередині.

## Версіонування

З моменту виходу D3D12 майже 10 років тому було додано багато нового функціоналу. Це означає, що його потрібно було якимось чином додати. У всіх API, які ми будемо розглядати, це робиться однаково.

Візьмемо як приклад `ID3D12GraphicsCommandList`. Цей інтерфейс дає доступ до різних видів GPU роботи. Деякі функції, такі як трасування променів, були додані після початкового випуску D3D12 шляхом створення нового інтерфейсу, який успадковує попередній:

<img src="{{site.baseurl}}/diagrams/ID3D12GraphicsCommandList.mmd.svg" height="800" alt="Діаграма класів для ID3D12GraphicsCommandList7">

Базова функціональність містяться в `ID3D12GraphicsCommandList`, трасування променів додано в `ID3D12GraphicsCommandList4`, Mesh Shaders додано в `ID3D12GraphicsCommandList6` тощо. Інші інтерфейси оновлюються таким же чином, якщо у вас є `ISampleInterface`, оновлення додадуть `ISampleInterface2`, `ISampleInterface3` тощо. Якщо у вас є вказівник на інтерфейс, ви також можете використовувати всі методи з попередніх версій, оскільки нові версії успадковують старіші.

Підтримка цих нових інтерфейсів визначається або версією ОС, або версією `Agility SDK` (докладніше про Agility SDK у майбутньому пості). Отже, якщо ви використовуєте стару версію ОС або Agility SDK, можливо, ви не зможете використовувати останній інтерфейс. Це не залежить від комплектуючих комп'ютера. Якщо GPU не підтримує трасування променів, ви все одно можете використовувати `ID3D12GraphicsCommandList4` як інтерфейс, при умові достатньо нової версії ОС або Agility SDK, вам просто не буде дозволено викликати методи, пов’язані з трасуванням променів.

Є 2 способи отримання нового інтерфейсу:

- Створити об'єкт відразу для нового інтерфейсу. Оскільки більшість методів створення приймають IID, ви можете просто використати тип нової версії інтерфейсу.
- Створити об'єкт для старої версії інтерфейсу, а потім використати `QueryInterface`, щоб отримати нову версію.

Навіть якщо ви вкажете конкретну версію інтерфейсу при створенні, об'єкт який ви отримаєте буде імпелентувати найновіший з підтримуваних інтерфейсів. Саме тому підхід з `QueryInterface` працює, якщо ви створете об'єкт для інтерфейсу `ID3D12GraphicsCommandList`, ви отримаєте об'єкт який імплементує усі інтерфейси з цієї діаграми класів, якщо ОС або Agility SDK їх підтримує.

Однак, якщо вам потрібно обробити випадок, коли остання версія інтерфейсу може не підтримуватися, запит нового інтерфейсу під час створення об'єкта може призвести до збою створення без створення жодного об'єкта. З іншого боку, якщо ви спочатку створюєте об'єкт для старішої версії інтерфейсу, він завжди працюватиме, а потім ви можете виконати `QueryInterface` і обробити помилку, щоб отримати новий інтерфейс.

Якщо певна версія інтерфейсу не підтримується, спроба створити об'єкт для цієї версії поверне помилку, і об'єкт не буде створено. Однак якщо ви спочатку створите об'єкт за допомогою старішої версії інтерфейсу, яка точно підтримується, створення вдасться. Потім ви можете викликати `QueryInterface`, щоб отримати новіший інтерфейс і обробити помилку, якщо вона недоступна.

Отже, рекомеднація з приводу використання інтерфейсів різних версій наступна:

- Створюйте об'єкти для останньої версії інтерфейсу яка буде підтримуватися або яку ви вимагаєте для роботи вашої програми.
- Використовуйте `QueryInterface`, щоб отримати новішу версію інтерфейсу та безпечно обробити ситуацію, коли вона не підтримується.

## Корисні матеріали

### Документація

- [Microsoft Learn](https://learn.microsoft.com) - Містить документацію для всіх 3 цих API. Цей сайт найбіль корисний коли вам потрібно знайти деталі конкретного методу, класу, структури або переліку. Це також єдиний ресурс, який містить документацію DXGI.
- [DirectX-Specs](https://microsoft.github.io/DirectX-Specs/) - Містить всю документацію для D3D12, включно з найновішим функціоналом.
- [Direct3D 11.3 Functional Specification](https://microsoft.github.io/DirectX-Specs/d3d/archive/D3D11_3_FunctionalSpec.htm) - Це величезний документ з усіма деталями DirectX 10-11.3. На жаль, D3D12 не має еквівалентного документа, тому у випадках, коли щось не описано в документації D3D12, передбачається, що воно працює так само, як і в D3D11.
- [DirectStorage Developer Guidance](https://github.com/microsoft/DirectStorage/blob/main/Docs/DeveloperGuidance.md) - Посібник із використання DStorage у поєднанні з кращими практиками та частковою документацією.

### Приклади

- [DirectX-Graphics-Samples](https://github.com/Microsoft/DirectX-Graphics-Samples) - Колекція прикладів D3D12, що охоплює багато різних функцій.
- [DirectStorage Samples](https://github.com/microsoft/DirectStorage/tree/main/Samples) - Колекція прикладів DStorage.

### Інші посилання

- [DirectX landing page](https://devblogs.microsoft.com/directx/landing-page/) - Список посилань на матеріали, пов'язані з DirectX.
- [DirectX (Developers) Discord Server](https://discord.gg/directx) - Discord Сервер де ви можете задавати питання щодо DirectX. Дуже рекомендую приєднатися.
- [NVIDIA's Advanced API Performance](https://developer.nvidia.com/blog/tag/advanced-api-performance) - Кращі практики D3D12, які допомагають максимізувати продуктивність. Зауважте, що вони можуть включати поради специфічні для GPU від Nvidia, які не обов'язково будуть покращувати продуктивність на інших GPU.
