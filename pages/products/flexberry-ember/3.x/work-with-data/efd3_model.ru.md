---
title: Модели в сгенерированных приложениях
sidebar: flexberry-ember-3_sidebar
keywords: Flexberry Ember
toc: true
permalink: ru/efd3_model.html
lang: ru
summary: Обзор структуры моделей в сгенерированных приложениях.
---

## Особенности объявления сгенерированных моделей в коде
Модели в ember-приложениях наследуются от класса Ember Data [DS.Model](http://emberjs.com/api/data/classes/DS.Model.html). В "чистом" Ember-приложении объявление модели выглядит следующим образом (для генерации новой модели можно воспользоваться командой `ember generate model test-model-name`, тогда соответсвующая модель будет расположена в файле `test-model-name.js` в папке `models`):

```js
import DS from 'ember-data';

export default DS.Model.extend({
});
```

{% include note.html content="Генерируемые из [Flexberry Designer](fd_flexberry-designer.html) модели создаются в папку `models` и именуются следующим образом: если соответствующий C#-класс на [OData-бэкенде](fo_orm-odata-service.html) называется `NewPlatform.Someproject.Somemodel`, то файл с моделью в клиентском приложении должен называться `new-platform-someproject-somemodel`. Если на [OData-бэкенде](fo_orm-odata-service.html) используется атрибут [PublishName](fo_metadata-for-client.html) для упрощения именования моделей, то наименование пространства имен в этом случае в клиентской модели может отсутствовать (имя клиентской модели будет формироваться соответственно имени в EDM-модели на OData-бакенде)" %}

## Базовые классы и миксины для моделей Flexberry Ember
Генерируемые из [Flexberry Designer](fd_flexberry-designer.html) модели чаще всего имеют следующую структуру

```js
// Импорт для валидации.
import { buildValidations } from 'ember-cp-validations';

// Импорт базового класса для моделей во Flexberry Ember.
import EmberFlexberryDataModel from 'ember-flexberry-data/models/model';

// Импорт модели для работы в офлайн-режиме.
import OfflineModelMixin from 'ember-flexberry-data/mixins/offline-model';

// Проекции, правила валидации и сама модель определяются из соответсвующего миксина.
import {
  defineProjections,
  ValidationRules,
  Model as AgregatorClassMixin
} from '../mixins/regenerated/models/i-i-s-gen-test-agregator-class';

// Подготовка для задания на модель правил валидации данных.
const Validations = buildValidations(ValidationRules, {
  dependentKeys: ['model.i18n.locale'],
});

// Непосредственно определение модели.
let Model = EmberFlexberryDataModel.extend(OfflineModelMixin, AgregatorClassMixin, Validations, {
});

// Определение проекций (сами проекции заданы в миксине)
defineProjections(Model);

// Экспорт модели.
export default Model;
```

Соответствующий модели миксин с таким же именем расположен в папке `mixins/regenerated/models`. Его структура следующая:

```js
// Необходимые импорты.
import Mixin from '@ember/object/mixin';
import $  from 'jquery';
import DS from 'ember-data';
import { validator } from 'ember-cp-validations';

// Импорт для объявления проекций.
import { attr, belongsTo, hasMany } from 'ember-flexberry-data/utils/attributes';

// Импорт для перечислимого типа.
import Enum1TypeEnum from '../../../enums/i-i-s-gen-test-enum1-type';

// Объявление атрибутов модели.
export let Model = Mixin.create({
  ...
});

// Объявление типичных правил валидации.
export let ValidationRules = {
  ...
};

// Объявление проекций.
export let defineProjections = function (modelClass) {
  ...
};
```

Модели определяются ["стандартным" для Ember способом](https://guides.emberjs.com/v3.4.0/models/defining-models/). Импорты и экспорт соответствуют требованиям синтаксиса [ember-cli](http://ember-cli.com).
Создаваемая модель наследуется от [базового технологического класса](https://github.com/Flexberry/ember-flexberry-data/blob/develop/addon/models/model.js), определенного в аддоне [`ember-flexberry-data`](https://github.com/Flexberry/ember-flexberry-data). Дополнительно в моделях, унаследованных от базовой технологической модели, используются [правила валидации модели](efd3_model-validation.html).

{% include warning.html content="Для **каждой** модели должен быть описан [сериализатор](efd3_serializer.html) для корректного взаимодействия с сервером." %}

Если у модели есть класс-предок, то для класса-потомка объявление модели будет чуть отличаться:

```js
import $ from 'jquery';
import { buildValidations } from 'ember-cp-validations';

import {
  defineBaseModel, // Импорт метода для работы с моделью предка.
  defineProjections,
  ValidationRules,
  Model as Child1Mixin
} from '../mixins/regenerated/models/i-i-s-gen-test-child1';

// Импорт модели предка.
import ParentClassModel from './i-i-s-gen-test-parent-class';

// Импорт правил валидации для модели предка.
import { ValidationRules as ParentValidationRules } from '../mixins/regenerated/models/i-i-s-gen-test-parent-class';

// Определение правил валидации с учётом правил валидации модели предка.
const Validations = buildValidations($.extend({}, ParentValidationRules, ValidationRules), {
  dependentKeys: ['model.i18n.locale'],
});

// Модель является расширением ParentClassModel, а не EmberFlexberryDataModel.
let Model = ParentClassModel.extend(Child1Mixin, Validations, {
});

// Определение элементов от базовой модели.
defineBaseModel(Model);
defineProjections(Model);

export default Model;
```

Соответствующий миксин отличается тем, что добавлено определение `defineBaseModel`:
```js
...
export let defineBaseModel = function (modelClass) {
  modelClass.reopenClass({
    _parentModelName: 'i-i-s-gen-test-parent-class'
  });
};
...
```

## Правила генерации атрибутов и связей в модели
Атрибуты в моделях определяются ["стандартным" для Ember способом](https://guides.emberjs.com/v3.4.0/models/defining-models/#toc_defining-attributes), [также как и связи](https://guides.emberjs.com/v3.4.0/models/relationships/).

```js
export let Model = Mixin.create({
  // Определение собственных атрибутов.
  doubleField: DS.attr('decimal'),
  stringField: DS.attr('string'),

  // Определение мастера.
  myMaster: DS.belongsTo('i-i-s-gen-test-master-for-child1', { inverse: null, async: false }),

  // Определение детейлов.
  detail1ForChild1: DS.hasMany('i-i-s-gen-test-detail1-for-child1', { inverse: 'child1', async: false }),
  detail2ForChild1: DS.hasMany('i-i-s-gen-test-detail2-for-child1', { inverse: 'child1', async: false })
});
```

{% include important.html content="Имена свойств должны начинаться с маленькой буквы." %}

### Генерируемые типы и трансформации
Для атрибутов любых моделей `Ember` доступны встроенные типы данных `string` (строка), `number` (число), `boolean` (логический тип) и `date` (дата). Для определения других типов данных в моделях Ember используются [трансформации](https://guides.emberjs.com/v3.4.0/models/defining-models/#toc_transforms).

Дополнительно к стандартным типам данных в аддоне `ember-flexberry-data` были добавлены трансформации `decimal` (вещественный тип), `file` (тип "Файл"), `flexberry-enum` (тип для перечислений), `guid` (тип "GUID", который используется по умолчанию для первичных ключей).

Подробнее об использовании трансформаций для перечислений можно посмотреть [тут](efd3_enum.html).

### inverse-связи в модели

Задание [inverse-связи](https://guides.emberjs.com/v3.4.0/models/relationships/#toc_reflexive-relations) используется, например, при работе с детейлами.

Задание связи от агрегатора к детейлу.

```javascript
export let Model = Mixin.create({
  ...
  detail1ForChild1: DS.hasMany('i-i-s-gen-test-detail1-for-child1', { inverse: 'child1', async: false }),
});
```

Задание связи от детейла к агрегатору.

```javascript
export let Model = Mixin.create({
  ...
  child1: DS.belongsTo('i-i-s-gen-test-child1', { inverse: 'detail1ForChild1', async: false })
});
```

## Первичный ключ в модели


## Представления в моделях. Особенности включения связей в представлениях
## Генерируемые сериализаторы для моделей
## Вспомогательный класс information?
## Вспомогательные модели Flexberry Ember
